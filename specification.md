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

Note that whenever a URI appears as a parameter in a GET request, it should be URL-encoded.

Use of https optional

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
        "order":"1"
       },
       {"id":"http://purl.org/linked-data/sdmx/2009/dimension#refPeriod",
        "label":"Reference period",
        "order":"2"
       },
       {"id":"http://purl.org/linked-data/cube#measureType",
        "label":"Measure type",
        "order":"3"
       },
       {"id":"http://statistics.gov.scot/def/dimension/gender",
        "label":"Gender",
        "order":"4"
       },       
       {"id":"http://statistics.gov.scot/def/dimension/age",
        "label":"Age",
        "order":"5"
       },
       {"id":"http://statistics.gov.scot/def/dimension/outcome",
        "label":"Outcome",
        "order":"6"
       }
   ]
}
```

## 5. Measures

```
GET measures
```

## 6. Attributes

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


#### dimension (mandatory)

### 7.2 Example request


### 7.3 Example response



## 8. Slice

```
GET slice
```


## 9. Observation







