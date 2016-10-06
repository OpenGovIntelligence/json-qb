# JSON-qb Specification

This is work in progress.

<!-- Document Info -->
<table class="table">
  <tr>
    <th>Authors</th>
    <td>
      Bill Roberts, Ric Roberts, Rick Moynihan (<a href="http://www.swirrl.com">Swirrl</a>)
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

## 1. Background and purpose

The RDF Data Cube Vocabulary is a commonly used standard for representing statistical data as RDF.  Data provided in this form can be accessed using the existing machinery of Linked Data, but skills and tooling for use of Linked Data are less widespread than for some other web technologies.

This API is designed to support web developers to use statistical data stored in the form of an RDF Data Cube, while assuming minimal knowledge of Linked Data.

The API is designed to be deliberately 'lossy': i.e. it favours simplicity and ease of use for developers over completeness.  Some aspects of the RDF Data Cube representation of the data will not be available through the API.

### 1.1 Introduction


### 1.2 Relevant standards

[RDF Data Cube Vocabulary](https://www.w3.org/TR/vocab-data-cube/)

[JSON](https://tools.ietf.org/html/rfc7159)

## 2. Structure of a data cube

TODO: explain about data cubes - dimensions, measures, observations etc

Note that cubes can be 'sparse' - not all combinations of dimensions necessarily have a corresponding observation.

Refer to OpenGovIntelligence work on an 'application profile' for use of the RDF Data Cube Vocabulary.  The API assumes that the underlying data follows this approach.

## 3. API overview

(TO DO: explain overall structure of API)

Note that whenever a URI appears as a parameter in a GET request, it should be URL-encoded.  Explain that URIs are used as identifiers.  Can treat them as just string identifiers, but for extra info, there is the option of using the Linked Data machinery, to look up that identifier and retrieve more info about it in RDF (eg JSON-LD).

The API does not currently include methods to support discovery of available datasets.  It assumes that you have found the identifier (URI) of a dataset supporting this API through other means.

Use of https optional

Dealing with multilingual labels? eg allowing more than one label for an item with @en or @gr etc after it?  Or provide some way to specify a general language preference?  eg allow a parameter of language=en etc on any method call.

## 4. Dimensions

```
GET dimensions
```

### 4.1 Parameters

#### dataset (mandatory)

The URI of the dataset. 

### 4.2 Example request

For dataset [http://statistics.gov.scot/data/road-safety](http://statistics.gov.scot/data/road-safety)
```
http://example.com/api/v1/dimensions?dataset=http%3A%2F%2Fstatistics.gov.scot%2Fdata%2Froad-safety
```

### 4.3 Example response

For each dimension, the _id_ and _label_ parameters are mandatory. The _order_ parameter is optional and can be used to guide the order in which dimensions are presented.  See [qb:order](https://www.w3.org/TR/vocab-data-cube/#ref_qb_order).

```json
{
   "version":"1",
   "dimensions":[
       {"id":"http://purl.org/linked-data/sdmx/2009/dimension#refArea",
        "label":"Reference area",
        "order":1
       },
       {"id":"http://purl.org/linked-data/sdmx/2009/dimension#refPeriod",
        "label":"Reference period",
        "order":2
       },
       {"id":"http://purl.org/linked-data/cube#measureType",
        "label":"Measure type",
        "order":3
       },
       {"id":"http://statistics.gov.scot/def/dimension/gender",
        "label":"Gender",
        "order":4
       },       
       {"id":"http://statistics.gov.scot/def/dimension/age",
        "label":"Age",
        "order":5
       },
       {"id":"http://statistics.gov.scot/def/dimension/outcome",
        "label":"Outcome",
        "order":6
       }
   ]
}
```

## 5. Measures

The measure provides a definition of what the values of the observations in the dataset represent.  In many datasets there is only a single measure, but it is possible to have more than one measure in a dataset....discuss measure dimension approach...rule out multiple measures per observation?

```
GET measures
```

## 6. Attributes

...typically used for units...attaching unit to observations vs attaching unit to measure property?

```
GET attributes
```

## 7. Dimension values

```
GET dimension-values
```

Returns all the values of the selected dimension that appear in the selected dataset.  Note that this is not necessarily the full list of values which could in other circumstances be permitted as values of that dimension, but the ones which have at least one observation associated with them in this dataset.

### 7.1 Parameters

#### dataset (mandatory)

The URI of the dataset

#### dimension (mandatory)

The URI of the dimension - the value of the 'id' parameter of one of the dimensions returned by the ```dimensions``` method.

### 7.2 Example request

```
http://example.com/api/v1/dimensions?dataset=http%3A%2F%2Fstatistics.gov.scot%2Fdata%2Froad-safety&dimension=http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fdimension%2Fgender
```
Get all the values of the 'gender' dimension in the Road Safety dataset.

### 7.3 Example response

```json
{
   "version":"1",
   "values":[
      {"id":"http://statistics.gov.scot/def/concept/gender/female",
       "label":"Female",
       "order":1},
      {"id":"http://statistics.gov.scot/def/concept/gender/female",
       "label":"Male",
       "order":2},
      {"id":"http://statistics.gov.scot/def/concept/gender/female",
       "label":"All",
       "order":3}
   ]
}
```

_order_ is optional, but provides a way for the data owner to suggest a suitable order for displaying the different possible values.

Note that for some dimensions there may be large number of possible values.  It is common for datasets to have thousands of values of the refArea dimension for example.  

(Need to support paging for the responses to this method?)

## 8. Slice

This bit might change a lot!

Not sure of value of slices with more than 2 dimensions, and a zero dimensional slice is covered by the _observation_ method, so not needed.  Maybe we can limit to 1D or 2D.  And Rick has a suggestion that we could always return a 2D structure, even if only 1D in reality. (to discuss).

Other options: 
* return something following the JSON-stat format.  https://json-stat.org  (example: https://json-stat.org/samples/galicia.json )
* the Google charts table format (or similar) https://developers.google.com/chart/interactive/docs/reference#dataparam

## 8.1 Get slice contents
```
GET slice
```
Retrieve a set of observations from the data cube corresponding to particular values of selected dimensions.  For a N-dimensional data cube, the _slice_ method can return a 0,1,...N dimensional slice, depending on how many dimensions are fixed. a 0-dimensional slice is equivalent to the _observation_ method.  If no dimensions are fixed, then _slice_ will return all observations in the dataset.

#### 8.1.1 Parameters

##### dataset (mandatory)

##### 0 or more dimension identifiers (optional)

The parameter should be the id (URI) of a dimension (as returned by _dimensions_) and the value of the parameter should be the id of an allowed value of the dimension (as returned by _dimension-values_)

#### 8.1.2 Example request - 1D slice

This example fixes all but one dimension of the dataset, so returning a 1-dimensional set of observations.  The example leaves the time dimension unspecified, so the response represents a time series.

```
http://example.com/api/v1/slice?dataset=http%3A%2F%2Fstatistics.gov.scot%2Fdata%2Froad-safety
&http%3A%2F%2Fpurl.org%2Flinked-data%2Fsdmx%2F2009%2Fdimension%23refArea=ttp%3A%2F%2Fstatistics.gov.scot%2Fid%2Fstatistical-geography%2FS12000034
&http%3A%2F%2Fpurl.org%2Flinked-data%2Fcube%23measureType=http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fmeasure-properties%2Fcount
&http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fdimension%2Fgender=http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fconcept%2Fgender%2Fmale
&http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fdimension%2Fage=http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fconcept%2Fage%2Fall
&http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fdimension%2Foutcome=http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fconcept%2Foutcome%2Fkilled-or-seriously-injured

```

Question to consider: if there is only one possible value of a dimension in a dataset, should we require the user to specify it?

#### 8.1.3 Example response - 1D slice

```json
{
  "version":"1",
  "observations":[
     {
       "id":"http://statistics.gov.scot/data/road-safety/year/2006/S12000034/gender/male/age/all/outcome/killed-or-seriously-injured/people/count",
       "http://purl.org/linked-data/sdmx/2009/dimension#refPeriod":"http://reference.data.gov.uk/id/year/2006",
       "value":123       
     },
     {
       "id":"http://statistics.gov.scot/data/road-safety/year/2007/S12000034/gender/male/age/all/outcome/killed-or-seriously-injured/people/count",
       "http://purl.org/linked-data/sdmx/2009/dimension#refPeriod":"http://reference.data.gov.uk/id/year/2007",
       "value":134       
     },
     {
       "id":"http://statistics.gov.scot/data/road-safety/year/2008/S12000034/gender/male/age/all/outcome/killed-or-seriously-injured/people/count",
       "http://purl.org/linked-data/sdmx/2009/dimension#refPeriod":"http://reference.data.gov.uk/id/year/2008",
       "value":168       
     },
     {
       "id":"http://statistics.gov.scot/data/road-safety/year/2009/S12000034/gender/male/age/all/outcome/killed-or-seriously-injured/people/count",
       "http://purl.org/linked-data/sdmx/2009/dimension#refPeriod":"http://reference.data.gov.uk/id/year/2009",
       "value":164       
     },
     {
       "id":"http://statistics.gov.scot/data/road-safety/year/2010/S12000034/gender/male/age/all/outcome/killed-or-seriously-injured/people/count",
       "http://purl.org/linked-data/sdmx/2009/dimension#refPeriod":"http://reference.data.gov.uk/id/year/2010",
       "value":139       
     },
     {
       "id":"http://statistics.gov.scot/data/road-safety/year/2011/S12000034/gender/male/age/all/outcome/killed-or-seriously-injured/people/count",
       "http://purl.org/linked-data/sdmx/2009/dimension#refPeriod":"http://reference.data.gov.uk/id/year/2011",
       "value":132       
     },
     {
       "id":"http://statistics.gov.scot/data/road-safety/year/2012/S12000034/gender/male/age/all/outcome/killed-or-seriously-injured/people/count",
       "http://purl.org/linked-data/sdmx/2009/dimension#refPeriod":"http://reference.data.gov.uk/id/year/2012",
       "value":147       
     },
     {
       "id":"http://statistics.gov.scot/data/road-safety/year/2013/S12000034/gender/male/age/all/outcome/killed-or-seriously-injured/people/count",
       "http://purl.org/linked-data/sdmx/2009/dimension#refPeriod":"http://reference.data.gov.uk/id/year/2013",
       "value":129       
     },
     {
       "id":"http://statistics.gov.scot/data/road-safety/year/2014/S12000034/gender/male/age/all/outcome/killed-or-seriously-injured/people/count",
       "http://purl.org/linked-data/sdmx/2009/dimension#refPeriod":"http://reference.data.gov.uk/id/year/2014",
       "value":130       
     }
  ]
}
```

how should we deal with ordering? (if at all - client could use the dimension-values method to find out about preferred order)

should we provide labels for the free dimension values in the response, or require the client to look them up separately with the _dimension-values_ method (eg including '2014' as well as http://reference.data.gov.uk/id/year/2014 ?)

include the unit in the response? (in this case 'http://statistics.gov.scot/def/concept/measure-units/people' or get that from the measure?)




#### 8.1.4 Example request - 2D slice

This example fixes all but two dimension of the dataset, so returning a 2-dimensional set of observations.  The example leaves the time dimension and the area dimenmsion unspecified, so the response represents a table of areas and times.

```
http://example.com/api/v1/slice?dataset=http%3A%2F%2Fstatistics.gov.scot%2Fdata%2Froad-safety
&http%3A%2F%2Fpurl.org%2Flinked-data%2Fsdmx%2F2009%2Fdimension%23refArea=ttp%3A%2F%2Fstatistics.gov.scot%2Fid%2Fstatistical-geography%2FS12000034
&http%3A%2F%2Fpurl.org%2Flinked-data%2Fcube%23measureType=http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fmeasure-properties%2Fcount
&http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fdimension%2Fgender=http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fconcept%2Fgender%2Fmale
&http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fdimension%2Fage=http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fconcept%2Fage%2Fall
&http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fdimension%2Foutcome=http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fconcept%2Foutcome%2Fkilled-or-seriously-injured

```

#### 8.1.5 Example response - 2D slice

```json
{
  "version":"1",
  "observations":[
     {
       "id":"http://statistics.gov.scot/data/road-safety/year/2006/S92000003/gender/male/age/all/outcome/killed-or-seriously-injured/people/count",
       "http://purl.org/linked-data/sdmx/2009/dimension#refArea":"http://statistics.gov.scot/id/statistical-geography/S92000003",
       "http://purl.org/linked-data/sdmx/2009/dimension#refPeriod":"http://reference.data.gov.uk/id/year/2006",
       "value":1916       
     },
     {
       "id":"http://statistics.gov.scot/data/road-safety/year/2006/S12000033/gender/male/age/all/outcome/killed-or-seriously-injured/people/count",
       "http://purl.org/linked-data/sdmx/2009/dimension#refArea":"http://statistics.gov.scot/id/statistical-geography/S12000033",
       "http://purl.org/linked-data/sdmx/2009/dimension#refPeriod":"http://reference.data.gov.uk/id/year/2006",
       "value":1916       
     },
     {
       "id":"http://statistics.gov.scot/data/road-safety/year/2006/S12000034/gender/male/age/all/outcome/killed-or-seriously-injured/people/count",
       "http://purl.org/linked-data/sdmx/2009/dimension#refArea":"http://statistics.gov.scot/id/statistical-geography/S12000034",
       "http://purl.org/linked-data/sdmx/2009/dimension#refPeriod":"http://reference.data.gov.uk/id/year/2006",
       "value":134       
     },
     ... (many more areas for this value of time, followed by cycling through areas for next value of time)
     {
       "id":"http://statistics.gov.scot/data/road-safety/year/2007/S92000003/gender/male/age/all/outcome/killed-or-seriously-injured/people/count",
       "http://purl.org/linked-data/sdmx/2009/dimension#refArea":"http://statistics.gov.scot/id/statistical-geography/S92000003",
       "http://purl.org/linked-data/sdmx/2009/dimension#refPeriod":"http://reference.data.gov.uk/id/year/2007",
       "value":168       
     },
     ...
 
  ]
}
```

#### 8.1.6 Paging of large responses


### 8.2 Slice - number of observations in the slice

```
slice/num-observations
```

This method returns the number of observations that will be returned in the slice.  Note that it is not obligatory for a data cube to have an observation for all combinations of dimension values, so the result of this method may be less than the product of the 'sizes' of each dimension.  It can be zero if there are no observations in the dataset corresponding to the specified dimensions.

This method can also be useful to check the size of a response before requesting the full details, so allowing choices of paging options etc.

#### 8.2.2 Parameters

Same as for _slice_

#### 8.2.3 Example request

```
http://example.com/api/v1/slice/num-observations?dataset=http%3A%2F%2Fstatistics.gov.scot%2Fdata%2Froad-safety
&http%3A%2F%2Fpurl.org%2Flinked-data%2Fsdmx%2F2009%2Fdimension%23refArea=ttp%3A%2F%2Fstatistics.gov.scot%2Fid%2Fstatistical-geography%2FS12000034
&http%3A%2F%2Fpurl.org%2Flinked-data%2Fcube%23measureType=http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fmeasure-properties%2Fcount
&http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fdimension%2Fgender=http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fconcept%2Fgender%2Fmale
&http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fdimension%2Fage=http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fconcept%2Fage%2Fall
&http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fdimension%2Foutcome=http%3A%2F%2Fstatistics.gov.scot%2Fdef%2Fconcept%2Foutcome%2Fkilled-or-seriously-injured
```

#### 8.2.4 Example response

## 9. Observation


## 10. Dataset metadata

```
GET dataset-metadata
```
Returns dataset metadata, such as title, description, licence

### 10.1 Parameters


#### Dataset (mandatory)

The URI of the dataset

### 10.2 Example request

For dataset [http://statistics.gov.scot/data/road-safety](http://statistics.gov.scot/data/road-safety)
```
http://example.com/api/v1/dataset-metadata?dataset=http%3A%2F%2Fstatistics.gov.scot%2Fdata%2Froad-safety
```

### 10.3 Example response


```json
{
   "version":"1",
   "title":"Road Casualties",
   "comment":"Number of people killed and seriously injured on Scotland's roads by age and gender",
   "description":"Number of people killed and serious injured on Scotland's roads. These figures are from the STATS-19 statistical returns collected from the police forces across Scotland. These statistics only focus on those accidents involving killed and seriously injured casualties. The statistics were compiled from returns made by police forces, which cover all accidents in which a vehicle is involved that occur on roads (including footways) and result in personal injury, if they become known to the police. The vehicle need not be moving, and need not be in collision - for example, the returns include accidents involving people alighting from buses. Very few, if any, fatal accidents do not become known to the police. However, there will be non-fatal injury accidents which are not reported by the public to the police, and so are not counted in these statistics. The publication [Reported Road Casualties Scotland](http://www.transportscotland.gov.uk/statistics/reported-road-casualties-scotland-all-editions) provides more information on this matter. Damage only accidents are not included in the above definition, and so the road accident statistical returns do not cover damage only accidents. It is thought that the number of damage only accidents is about fourteen times the number of reported injury road accidents. For more detailed statistics of injury road accidents and a full description of the terms used see the publication Reported Road Casualties Scotland and the Key Reported Road Casualties Scotland Statistical Bulletin. The figures they contain may differ slightly from those published here due to late returns and amendments made to the database in the periods between the finalisation of the statistics for the purpose of the publications.",
   "license":"http://www.nationalarchives.gov.uk/doc/open-government-licence/version/3/"
 }
```





