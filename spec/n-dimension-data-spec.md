# Supporting N-dimensional data & Multiple Measures

This is a proposed data representation to generalise the `data`
described in the core table spec.

Essentially the idea is to provide one uniform representation for N
dimensional data instead of the 4 different approaches for a single
observation, an array of observations and a table of observations with
no representation for higher dimensions previously proposed.

This approach is intended to work for tables with an arbitrary number
of dimensions, by providing multiple headers along one axis.  We
acknowledge that we could provide multiple headers along both axis,
allowing Roll/Ups and aggregations for example, however we have chosen
not to support this at this time.  We may support this in a future
extension.

## Multiple Measures

We believe the best way to support measures is to abstract over the
two different kinds of cubes, by treating measures uniformly like
they're another kind of dimension, even in the multiple-measure on a
single observation case.

From an API users perspective both styles of multi-measure cube should
be made to look the same by listing the measures as headers like with
other dimensions.  We should therefore adopt a "cell as value"
approach rather a "cell as observation" approach, meaning in the case
of multiple-measures on a single observation we should expand them out
to be listed as a new cell.

Value objects which occur within spreadsheet cells, will in both cases
still link to the underlying observations URI, so the main difference
an API would notice between both styles of dataset is merely that an
observations `@id` would appear duplicated with different measure
values in different cells.

## Multiple Headers on one axis

The JSON snippet below illustrates how we describe a multi-column
dataset.  Where free dimensions are mapped into multiple columns, in a
hierarchy specified by the `column_hierarchy` key.  This defines that
the outermost column header must be `year`, followed by `gender` and
`measure` e.g.

```
+------------+-------------------------------------------+-------------------------------------------+-------------------------------------------+-------------------------------------------+
|            |                      1999                 |                   2000                    |                   2001                    |                    2002                   |
|            |---------------------+---------------------+---------------------+---------------------+---------------------+---------------------+---------------------+---------------------+
|            |        Male         |        Female       |        Male         |        Female       |        Male         |        Female       |        Male         |        Female       |
+------------+---------------------+---------------------+---------------------+---------------------+---------------------+---------------------+---------------------+---------------------+
|  Ref Area  |  Count   |   Ratio  |   Count  |  Ratio   |   Count  |  Ratio   |  Count   |   Ratio  |   Count  |  Ratio   |   Count  |  Ratio   |  Count   |   Ratio  |   Count  |  Ratio   |
+------------+----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+
| S12000005  | 101      | 20.3     | 104      | 21.2     | 102      | 20.6     | 203      | 31.3     | 90       | 19.4     | 98       |19.6      | 223      | 30.3     | 10       | 1.3      |
+------------+----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+
```

The values for the headers are then defined inside the `headers` map
under the keys `columns` and `rows`.

Each header value is an ordered array of identifiers referencing each
`dimension_value`.


```json
{
        "headers" : {"columns": {"year" : ["1999", "2000", "2001", "2002"],
                                 "gender" : ["Male", "Female"],
                                 "measure" : ["Count", "Ratio"]},

                     "rows" : {"refarea" : ["S12000005", "S12000042", "S12000034", "S12000035", "S12000041", "S12000013", "S12000006", "S12000036", "S12000008", "S12000045",
                                            "S12000033"]},

                     "column_hierarchy" : ["year", "gender", "measure"]},


        "data": [[[101, 20.3], [104, 21.2]],
                 [[102, 20.6], [203, 31.3]],
                 [[90, 19.4], [98, 19.6]],
                 [[223, 30.3], [10, 1.3]],
                 "..."
                ]
}
```

We may in the future add support for multiple row headers, but assume
that "rows" will be paged and that "columns" will be materialised.  In
the case where a client is asking the server for "too much", it may
respond with no key/value pair for `data`.

There are several approaches for representing `data`, we could adopt a
flat, row major order approach like with
[json-stat](http://json-stat.org/) though have here proposed using
nested arrays corresponding to the nesting of column headers.  This
approach has some clarity benefits.  Above we represent each cell as
the associated measure literal for illustration; but plan to store an
object in each cell position that links back to the `@id` of the
appropriate observation and store its measured value under the key
`value`.

## Sorting

We provide two mutually exclusive methods for sorting data,
`by_column_value` and `by_row_order`, both options also support a
`direction` property which lets you specify either `asc` or `desc` for
an ascending or descending order respectively.

### Sorting by_column_value

The key `by_column_value` is used to indicate which column dimension
the data was sorted by.  Setting this means that all of the rows
(including the row headers) will be sorted by either the `asc`ending
or `desc`ending order of values in the specified column.  The value
for the `by_column_value` key identifies the column to sort on by
specifying a path to the column dimension.  For example the JSON
snippet below shows that the data is sorted by the column 2000 / Male
/ Ratio column.  It is only valid to sort on leaf columns, not parent
ones.

```json
        "sorted" : {"by_column_value" : ["2000" "Male" "Ratio"],
                    "direction": "asc"}
```

### Sorting by_row_order

The other way to sort is by the order of the values in the row
dimension.  This is an orthogonal, way to sort as you are sorting not
by values in the data, but by the order of the free dimension that is
mapped to the row axis.

The actual algorithm used to sort `by_row_order` is implementation
specific, but should where supplied use the appropriate properties
defined in the code-list.  If no such properties are supplied
implementations should choose to sort on another parameter, such as an
associated label or identifier.

As with `by_column_value` the `direction` can be set as either `asc`
or `desc`.

```json
        "sorted" : {"by_row_order" : "refarea",
                    "direction" : "desc"}
```