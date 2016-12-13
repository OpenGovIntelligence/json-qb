# Core Table

This document proposes a format for a JSON document structure that can
describe 2D and 1D slices of data from a data cube along with a means
of expressing accompanying meta-data.  Additionally the format
supports the description of meta-data associated with N-dimensional
slices.

This core format is purposely intended to be entirely separated from
service & behaviourial concerns, such as providing HATEOS links to API
calls, pagination etc.

The format is intended to define a minimal core format and an
extension mechanism for augmenting it with additional namespaced
features.  Whilst this format is intended to be used with the JSON-qb
REST API, the features that link the API to this document will be
defined in an extension, elsewhere.

We believe this extensibity is important, as it will ensure
interoperability and allow people the flexibility to define extensions
that suit their own usecases.  It will also allow us to avoid baking
in requirements for specific partners into the core data
specification.

Note that this is not yet intended to be a formal specification, but
an illustrative description of our ideas, and hopefully a basis for
constructing a formal specification.

This is a work in progress.

<!-- Document Info -->
<table class="table">
  <tr>
    <th>Authors</th>
    <td>
      Rick Moynihan, Ric Roberts, Bill Roberts (<a href="http://swirr.com/">Swirrl</a>)
    </td>
  </tr>
  <tr>
    <th>Version</th>
    <td>0.1</td>
  </tr>
  <tr>
    <th>Copyright</th>
    <td>
      Copyright &copy; 2016 by Swirrl IT Limited . This work is licensed under a
      <a href="http://creativecommons.org/licenses/by/4.0/">
        Creative Commons Attribution 4.0 International License</a>.
    </td>
  </tr>
</table>

# Typographic Conventions

- `value` corresponds to a specific programmatic construct, such as a
  key in a hashmap, or a datatype.

- `[foo bar]` corresponds to a path in a JSON tree.

- `<foo>` indicates a named variable/placeholder or class of things,
  e.g. the path `[foo bar <baz>]` would correspond to either all
  `<baz>`s under the path `[foo bar]` or it might describe a variable
  for a specific but unstated `<baz>`.

# Background & Purpose

This format is designed to support and satisfy the following use-cases
and constraints:

1. Enabling developers (in particular Javascript developers) to easily
   build charts and visualisations on top of QB data, without
   requiring knowledge of Linked Data.

2. Optimise for developers familiar with 2 dimension tabular
   representations of data.  We believe highly-dimensional data is
   usually best presented as 2 dimensional tables, with means for
   navigating to adjacent/near-by slices.

3. Support thin (or thick) clients that wish to drill-down from an
   N-dimensional cube to a 2 dimensional table, to a single column or
   even a single cell (observation).

4. Optimise towards presenting paginated views of the data, rather
   than large datasets which might require more resources than are
   available in a client/web-browser.

5. Provide a bridge from the Javascript world of JSON-tree's to the
   graph data world of RDF, with representations of URI's, language
   tags, and RDF data-types e.g. `xsd:dateTime`'s.  In particular we
   want to allow javascript developers to leverage additional metadata
   and identifiers from the RDF world, should they wish.

6. Whilst offering hooks into the RDF world, we shouldn't require
   Javascript developers to have indepth knowledge of Linked Data.

7. Avoids terminology of rows or columns, as that would imply an
   intended orientation for display.  The format in the document is
   agnostic to how it is rendered, so it is left to the application to
   decide in which orientation to render it.  This means that as a
   design principle we identify dimensions by their keys, not by their
   display properties.

8. We follow the convention that key names should always mean the same
   thing.

# Format Description

This document uses the
[Births in scotland by area and year](http://statistics.gov.scot/slice?dataset=http%3A%2F%2Fstatistics.gov.scot%2Fdata%2Fbirths&http%3A%2F%2Fpurl.org%2Flinked-data%2Fcube%23measureType=http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fmeasure-properties%2Fcount&http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fdimension%2Fgender=http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fconcept%2Fgender%2Fall&http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fdimension%2FtimePeriod=http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fconcept%2Ftime-period%2Fcalendar-year)
table as a motivating example.  Browsing and exploring this dataset
at the previous link through PMD should assist in understanding the
ideas presented in this document.

In particular the slice we are describing here corresponds to the
following table:

```
| (Council Areas)           | 1999 | 2000 | 2001 | 2002 | 2003 | 2004 |
|---------------------------+------+------+------+------+------+------|
| Aberdeen City             | 2311 | 2088 | 2097 | 2098 | 2003 | 2075 |
| Aberdeenshire             | 2501 | 2320 | 2247 | 2326 | 2368 | 2388 |
| Angus                     | 1085 | 1049 | 1103 | 1014 | 1089 | 1071 |
| Argyll and Bute           |  834 |  792 |  780 |  755 |  732 |  807 |
| City of Edinburgh         | 4764 | 4643 | 4489 | 4477 | 4577 | 4568 |
| Clackmannanshire          |  544 |  490 |  529 |  480 |  468 |  501 |
| Comhairle nan Eilean Siar |  265 |  228 |  226 |  242 |  255 |  223 |
| Dumfries and Galloway     | 1402 | 1369 | 1283 | 1343 | 1307 | 1431 |
| Dundee City               | 1522 | 1462 | 1468 | 1436 | 1548 | 1539 |
| East Ayrshire             | 1290 | 1228 | 1198 | 1157 | 1236 | 1271 |
```

Below we describe piece by piece the relevant parts of a Core Table
document, the complete example of the above table is described in [/spec/table-format/table-format-1.json](/spec/table-format/table-format-1.json).

## Dimension Metadata & Stucture (Any number of Free Dimensions)

The `structure` object contains sub-objects `all_dimensions` and
`all_dimension_values` that represent all available dimensions and all
their possible values in the cube.  This meta data is intended to be
included in every response to provide applications with knowledge of
all the available drop downs and their available values.

This structure will always be present regardless of the dimensionality
and number of free dimensions.

Additionally the objects `free_dimensions`, and `locked_dimensions`
provide information on the "drill-down" state, i.e. the dimensions
that the `locked_value` of the `locked_dimensions`, and the free
dimensions - which will ultimately be related to the `headings` when
there are either 1 or 2 free dimensions.

For example the following is the structure object associated with the
table above:

```json
"structure":
 {"free_dimensions": {"refarea":
                      {"@id":"http://purl.org/linked-data/sdmx/2009/dimension#refarea",
                       "label":"Reference Area"},
                      "refperiod":
                      {"@id":"http://purl.org/linked-data/sdmx/2009/dimension#refperiod",
                       "label":"Reference Period"}},
  "locked_dimensions": {"timeperiod":
                        {"@id":"http://statistics.gov.scot/def/dimension/timePeriod",
                         "label":"time period",
                         "locked_value":
                         {"@id":
                          "http://statistics.gov.scot/def/concept/time-period/calendar-year",
                          "label":"Calendar Year"}},
                        "gender":
                        {"@id":"http://statistics.gov.scot/def/dimension/gender",
                         "label":"Gender",
                         "locked_value":
                         {"@id":"http://statistics.gov.scot/def/concept/gender/all",
                          "label":"All"}}},
  "all_dimensions": {"refarea":
                     {"@id":"http://purl.org/linked-data/sdmx/2009/dimension#refarea",
                      "label":"Reference Area"},
                     "refperiod":
                     {"@id":"http://purl.org/linked-data/sdmx/2009/dimension#refperiod",
                      "label":"Reference Period"},
                     "gender":
                     {"@id":"http://statistics.gov.scot/def/dimension/gender",
                      "label":"Gender"},
                     "timeperiod":
                     {"@id":"http://statistics.gov.scot/def/dimension/timePeriod",
                      "label":"Time Period"}},
  "all_dimension_values":
  {"refArea": {"S12000005":
               {"@id":
                "http://statistics.gov.scot/id/statistical-geography/S12000005",
                "label":"Clackmannanshire"},
               "S12000042":
               {"@id": "http://statistics.gov.scot/id/statistical-geography/S12000042",
                "label":"Dundee City"},
               "S12000034":
               {"@id": "http://statistics.gov.scot/id/statistical-geography/S12000034",
                "label":"Aberdeenshire"},
                "...": "more..."
                },
   "refPeriod": {"1999":
                 {"@id":"http://reference.data.gov.uk/id/year/1999",
                  "label":"1999"},
                 "2000":
                 {"@id":"http://reference.data.gov.uk/id/year/2000",
                  "label":"2000"},
                 "2001":
                 {"@id":"http://reference.data.gov.uk/id/year/2001",
                  "label":"2001"},
                 "2002":
                 {"@id":"http://reference.data.gov.uk/id/year/2002",
                  "label":"2002"},
                 "2003":
                 {"@id":"http://reference.data.gov.uk/id/year/2003",
                  "label":"2003"},
                 "2004":
                 {"@id":"http://reference.data.gov.uk/id/year/2004",
                  "label":"2004"}},
   "timePeriod": {"between-mid-points-of-years":
                  {"@id": "http://statistics.gov.scot/def/concept/time-period/between-mid-points-of-years",
                   "label":"Between Mid Points Of Years"},
                  "2004":
                  {"@id":"http://reference.data.gov.uk/id/year/2004",
                   "label":"2004"}},
   "gender": {"all":
              {"@id":"http://statistics.gov.scot/def/concept/gender/all",
               "label":"All"},
              "female":
              {"@id":"http://statistics.gov.scot/def/concept/gender/female",
               "label":"Female"},
              "male":
              {"@id":"http://statistics.gov.scot/def/concept/gender/male",
               "label":"Male"}}}
````

## Multiple Column Table (2 Free Dimensions)

We define the keys `table`, `table_by`, `headings` and `total_cells`
to describe a multi column table or 2D slice.  The `headings` and
`total_cells` keys are present for all dimensionalities of data,
whilst the `table` and `table_by` keys will only be present if the
data is two dimensional.

When there are two dimensions the `headings` object will contain
`<dimension-key>`'s for each of the dimensions in the table (here
`refArea` and `refPeriod`).

The values under these keys will always be an ordered array of keys
corresponding to `dimension-values` specified in
`[structure dimension-values <dimension-key>]`.  The order of the keys
corresponds directly to the order of the data under `table`.

A summarised example of the table above looks like this:

````json
 "headings": {"refArea": ["S12000005", "S12000042", "S12000034", "S12000035", "S12000041", "S12000013", "S12000006", "S12000036", "S12000033"],
              "refPeriod": ["1999", "2000", "2001", "2002", "2003", "2004"]},
 "table_by":"refPeriod",
 "total_cells": 186,
 "table": {"1999":[2311, 2501, 1085, 834, 4764, 544, 265, 1402, 1522],
           "2000":[2088, 2320, 1049, 792, 4643, 490, 228, 1369, 1462],
           "2001":[2097, 2247, 1103, 780, 4489, 529, 226, 1283, 1468],
           "2002":[2098, 2326, 1014, 755, 4477, 480, 242, 1343, 1436],
           "2003":[2003, 2368, 1089, 732, 4577, 468, 255, 1307, 1548],
           "2004":[2075, 2388, 1071, 807, 4568, 501, 223, 1431, 1539]}
````

`NOTE` in practice the table arrays will contain objects with `@id`
links to the underlying observations whilst the `value` key will
reference the observations `measure-property`.  Additional metadata on
the observation may also be present.  An example "full" cell is [shown below](#single-cell-values).

The `table` key contains an object with keys corresponding to
`dimension-values` belonging to the dimension specified by `table_by`.

Therefore if we were to print the table shown above from this data we
would need to:

1. Inspect `table_by` to discover which dimension the keys in the
   `table` map belong to.  Here we note that `table_by` shows that
   they are `refPeriod`s.

2. As HashMaps (in many languages) aren't explicitly ordered we next
   need to check the order in which to process this dimension.  We do
   this by looking up the column sort-order for `refPeriod` in
   `headings refPeriod` this shows that the order of the columns
   should be `1999, 2000, 2001, 2002, 2003, 2004`.

3. The arrays under `[table <dimension-value>]` therefore represent
   columns of values for each cell.  Whilst the other dimension in
   `headings`, `refArea` correspond to row header's in our above table
   rendering.  Note that this means that the count of the
   `[headings refPeriod]` array should be equal to the number of keys
   in the `table` object, and that the count of the headings under
   `[headings refArea]` should be equal to the length of each of the
   arrays beneath `table`.  Note that these arrays are all implicitly
   positionally indexed.  So to render the table we could trivially do
   so with two for loops.

The key `total_cells` provides a count of the number of cells within the
currently selected slice.  This value may be greater than the amount
of cells shown in the `table`, due to pagination.

The above is basically the minimum a developer would need to do to
draw a basic of the data.  However it's worth noting that the
`dimension-value` keys aren't guaranteed to be human readable, though
ideally they normally would be.  To make sure we have the right labels
for each value we need to look the values up inside `[structure
dimension-values <dimension-value-key> label]`

## Single Column Table (1 Free Dimension)

When the dimensions are sufficiently locked to form a single column
the data will be represented like this:

````
 "headings": {"refArea": ["S12000005", "S12000042", "S12000034", "S12000035", "S12000041", "S12000013", "S12000006", "S12000036", "S12000033"]},
 "total_cells": 31,
 "array": [2311, 2501, 1085, 834, 4764, 544, 265, 1402, 1522]
````

Note that because we now only have 1 free dimension, the column no
longer has a column label (technically the label of the column would
be the union of all its locked dimension/values).  For this reason we
change the format of the data structure to be just a single array.
Because the format of the structure has changed though, we change the
key name from `table` to `array` to indicate a change in the datatype,
and provide an easy way to test for the case where there is a single
free dimension.

A Core Table document will be considered invalid if it has both a
`table` and `array` key simultaneously.

`total_cells` is 31, because there are 31 councils in Scotland,
however the page size extension is cutting the results short.

## Single Cell Values

It is possible to lock all the dimensions so you end up with a single
cell instead of a table or column/array.  When this happens the data
looks like this:

````
 "total_cells": 31,
 "cell": 2311
````

Note that there are no `headings` as there are no longer any free
dimensions.

TBD.  Cells need to be objects not literals.  We need to update this
example to use objects not literal values with `@id` links to the RDF
observations and other keys containing `value` and metadata.

## > 2 dimensions

When there are more than 2 dimensions, we explicitly choose not to
provide a representation of the data itself, and only provide data
pertaining to the `structure` and dataset meta-data etc.

The reason for this is that when there are many dimensions there is
often a large amount of data for which there is no simple or intuitive
way to paginate or display.  Likewise paginated sections of higher
dimensional slices are largely meaningless to visualisations and end
users.  If users want the whole cube then they can use an alternate
format more suited to it, such as `application/n-triples`.

## Service Layer Extension

The Table Core format so far says nothing about integration with the
service layer, and has instead focused on being a data serialisation
format for cube slices.  We believe that this boundary is important
and also requires representation in the data format, for example some
applications may wish to communicate both the URI of a resource and a
URL link to another service route.  One example of this integration is
with `_pagination` links which could be used to propogate application
state in the manner of [HATEOS](https://en.wikipedia.org/wiki/HATEOAS)
query parameters.

In order to achieve a clear separation of concerns and retain the
ability for applications to unambiguously separate the data from the
service we reserve the `_` prefix in JSON object keys for applications
and services to overlay the core data format with their own
definitions anywhere outside of a `@context` block.  This allows
developers the flexibility to add service specific annotations to the
format anywhere in the JSON tree.

If we are to build tools that can work across different services then
we may also need to standardise these links, though I think we should
do so in another specification.  Also we may wish to even incorporate
a more elaborate 'extension mechanism' that allows services to
implement different parts of the standard, whilst extending it in
unforseen ways.


## JSON-LD

This format is implemented as a JSON-LD document.  JSON-LD already
contains formally specified tools for bridging between RDF and JSON,
in particular it provides methods of associating a JSON object with a
URI `@id`, whilst providing mechanisms for prefixing URI's and mapping
between URI's and localised symbolic aliases and names via the
`@context` map.  Additionally it includes support for representing
`@language` strings.  JSON-LD documents always have two
interpretations, which can though need not align; the JSON
interpretation and an RDF interpretation.

The primary motivation for using JSON-LD for the table format is not
however to provide an RDF interpretation, as if you want an RDF
interpretation you can just access the cube as RDF.  The primary
motiviation for adopting JSON-LD is to use its features to provide a
mapping between these two worlds.

However, we believe it's important to follow the principle of least
surprise and correctness and ensure that if one of these documents is
interpreted in RDF mode, that the quads yielded by the process map
idempotently onto the original RDF.  This does not impose the
requirement that the RDF retrieved should be complete, just that it is
not contradictory to the source RDF.

We believe it important that if we are to leverage the JSON-LD
standard for its features, then we are compelled to also ensure it is
correct and semantically valid RDF.  However in the current JSON-LD
implementation it is not really possible to communicate the graph
perfectly without [adversely impacting the JSON implementation](https://github.com/json-ld/json-ld.org/issues/443).

The reason for this is due to limitations in JSON-LD's expressive
power and the fact that DSD's are quite deep structures and
consequently require you to add extra depth to the JSON document, at
the expense of the JSON user.

My goal therefore was to represent an isomorphic subset of the graph
that corresponds precisely with the real RDF.  This subset is really
only intended to be correct, not complete, nor useful.

The JSON table discussed
[here](/spec/table-format/table-format-1.json) translates into the
following RDF triples:

````nquads
<http://statistics.gov.scot/data/births> <http://purl.org/dc/terms/issued> "2014-07-29T02:00:00+02:00"^^<http://www.w3.org/2001/XMLSchema#dateTime> .
<http://statistics.gov.scot/data/births> <http://purl.org/dc/terms/modified> "2016-02-09T12:17:48Z"^^<http://www.w3.org/2001/XMLSchema#dateTime> .
<http://statistics.gov.scot/data/births> <http://purl.org/dc/terms/title> "Births" .
````


TBD: align JSON with RDF.

# Alternative Formats

TBD: list reasons for inventing a new format and not adopting one of
the alternatives below:

- http://json-stat.org/
- https://www.w3.org/TR/csv2json/

# Future Work / Considerations

- Consider whether we need a more elaborate extension mechanism, with
  e.g. namespaces etc.

- Add support for filtering rows

- Consider support for tree hierarchies / aggregations / roll-ups etc
  in table rows.

- Applications will need to define a reliable heuristic for generating
  prefixes and keys for use within the JSON.  We should discuss
  approaches to this, though this spec may not need to care how it is
  done.

- Consider support for nested headers

- Indicating how data is sorted, e.g. by which column and which
  direction.

- 2 dimensional paging??  We've optimised for the common case, but we
  might want to consider what supporting paging might look like in 2+
  dimensions.  I suspect it will be too complicated and rarely useful
  though.
