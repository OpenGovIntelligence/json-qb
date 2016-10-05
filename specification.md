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

Support web developers to use statistical data, without assuming in-depth knowledge of Linked Data

### 1.1 Introduction


### 1.2 Relevant standards

[RDF Data Cube Vocabulary](https://www.w3.org/TR/vocab-data-cube/)

[JSON](https://tools.ietf.org/html/rfc7159)

## 2. Structure of a data cube


## 3. API overview

(TO DO: explain overall structure of API)

Note that whenever a URI appears as a parameter in a GET request, it should be URL-encoded.

Use of https optional

## 4. Dimensions

GET dimensions

### 4.1 Parameters

#### dataset (mandatory)

The URI of the dataset. 

### 4.2 Example request

For dataset [http://statistics.gov.scot/data/road-safety](http://statistics.gov.scot/data/road-safety)
```
http://example.com/api/v1/dimensions?dataset=http%3A%2F%2Fstatistics.gov.scot%2Fdata%2Froad-safety

### 4.3 Example response

```json
[{...}]
```

## 5. Measures


## 6. Attributes



## 7. Dimension values


## 8. Slice


## 9. Observation







