# PatchFox Data Service Database API Documentation

## Overview

The data-service provides an HTTP API for quering the database and receiving results as paginated collections of JSON encoded database records. The service itself is a spring-boot microservice that implements Spring Data REST paging and sorting capabilities WITH THE FOLLOWING EXCEPTIONS:  

* the `sort` parameter uses of a period - NOT A COMMA! The correct way to use the `sort` parameter is `sort=commitDateTime.desc`

---

## Table Entity Overview

The system manages software package data with security vulnerability information. See [Entity Description](./entities/entities.md) for a list and short description of each. 

---


### Database Query API 

**Endpoint Pattern:** `{HOST}/api/v1/db/{tableName}/query`
- **Table names are case-insensitive** (e.g., `PACKAGE`, `package`, `Package` all work)


#### Capabilities

**Exact Match for Boolean, Numeric, and Dates:**
```http
GET /api/v1/db/datasetMetrics/query?isCurrent=true
GET /api/v1/db/datasetMetrics/query?totalFindings=100
GET /api/v1/db/datasetMetrics/query?commitDateTime=2024-01-01T00:00:00Z
```

**Numeric Comparisons:**
```http
GET /api/v1/db/datasetMetrics/query?totalFindings=gt.100        # Greater than 100
GET /api/v1/db/package/query?numberVersionsBehindHead=lt.5      # Less than 5
GET /api/v1/db/datasetMetrics/query?criticalFindings=gte.10     # Greater than or equal
GET /api/v1/db/datasetMetrics/query?lowFindings=lte.50          # Less than or equal
GET /api/v1/db/datasetMetrics/query?totalFindings=gte.10&gt.100 # Between 10 and 100
```

**Date Range Filtering:**
```http
GET /api/v1/db/datasetMetrics/query?commitDateTime=gt.2024-01-01T00:00:00Z
GET /api/v1/db/datasourceEvent/query?eventDateTime=lt.2024-12-31T23:59:59Z
GET /api/v1/db/datasetMetrics/query?commitDateTime=gte.2024-01-01T00:00:00Z&gt.2024-12-31T23:59:59Z
```

**Pattern Matching For String Values:**
```http
GET /api/v1/db/findingData/query?description=SQL # Contains "SQL"
```

**Multiple Values (OR logic for same field):**
```http
GET /api/v1/db/package/query?type=npm,maven,pypi   # type is npm OR maven OR pypi
```

**Boolean Fields:**
```http
GET /api/v1/db/datasetMetrics/query?isCurrent=true&isForecastRecommendationsTaken=false
GET /api/v1/db/edit/query?isPfRecommendedEdit=true&isUserEdit=false
```

#### Complex Query Examples

**Find vulnerable npm packages:**
```http
GET /api/v1/db/package/query?type=npm&totalFindings=gt.0
```

**Find current dataset metrics with high critical findings:**
```http
GET /api/v1/db/datasetMetrics/query?isCurrent=true&criticalFindings=gt.10
```

**Find packages with versions:**
```http
GET /api/v1/db/package/query?version=1.0.0,1.1.0,2.0.0
```


## special queryDSL endpoints 

There are many times when the question being asked is tied in with a given Dataset at a given time. For questions involving Packages, Findings, or Edits associated with a given Dataset at a given time, there are the following four endpoints to help. 

### packages 

**find packages associated wtih a given dataset.**
```http
/api/v1/db/datasetMetrics/package/query
```

**find package types (dedup of find packages) associated wtih a given dataset**
```http
/api/v1/db/datasetMetrics/packageType/query
```

**find packages associated with a set of datasources that are associated with a given dataset.** 
```http
/api/v1/db/datasetMetrics/datasource/package/query?datasetName=reddit&isCurrent=true&datasources.purl=go__src&commitDateTime=gt.1999-12-29T00:24:26Z
```

### findings

**find finding types associated wtih a given dataset**
```http
/api/v1/db/datasetMetrics/package/findingType/query
```

**find findings associated wtih a set of datasources that are associated with a given dataset**
```http
/api/v1/db/datasetMetrics/datasource/package/finding/query
```

### edits

**find edits associated wtih a given dataset**
```http
/api/v1/db/datasetMetrics/edit/query
```

**find edits associated wtih a given set of datasources that are associated with a given dataset**
```http
/api/v1/db/datasetMetrics/datasource/edit/query
```


Every one of these takes a small set of query string arguments intended for a DatasetMetrics query. Any remaining arguments will be passed along to the subsequent query to the Package, Finding, or Edit datastores respectively. These arguments are (and they must be camelCased):

* __datasetName__ (required)
  * the name of the Dataset or Datasets you want to include in the query
  * single name ex `datasetName=foo` 
  * multiple names ex `datasetName=foo,bar`
* __datasources.purl__ (required for datasetMetrics/datasource queries)
  * comma delimited list of datasource names (or full purls) used to scope the query to only datasources in the dataset that match the provided datasource names.
* __commitDateTime__ (optional)
  * the specific commitDateTime(s) you want records for. If not supplied you will get the latest record 
  * single date exact ex `commitDateTime=2025-04-09T01:08:40.648Z`
  * on or after date ex `commitDateTime=gte.2025-04-09T01:08:40.648Z`
  * range ex `commitDateTime.gte=2025-04-09T01:08:40.648Z&commitDateTime=lt.2025-05-09T01:08:40.648Z`
* __{isCurrent, isForecastSameCourse, isForecastRecommendationsTaken}__ (one of required)
  * indicates what kind of Dataset information you want: the actual data, the forecast based on the actual data, or the recommendations made based on the actual and forecasted data. 
  * just the actual data ex `isCurrent=true`
  * the actual and forecast data `isCurrent=true&isForecastSameCourse=true`

For example, if you make the following call 

```
/api/v1/db/datasetMetrics/package/query?datasetName=foo&isCurrent=true&purl=bar
```

PatchFox will retrieve the most recent metrics record for dataset "foo" marked "is_current". It will then look at all the Packages associated with that record for anything with a field "purl" that contains the text "bar" and return those to the caller. 


## special argument available only to genAI callers

Because the PatchFox genAI agent is making queries by way of tool middleware to the data-service it has access to a special argument called `select` that serves the same purpose as a SELECT statment in SQL. The effect is to reduce the fields returned by the API to only those specified by the `select` argument. 

### ex. select only the status field from the dataset table where the "id" field equals "1"
```
https://{DATA_SERVICE_HOST}/api/v1/db/dataset/query?id=1&select=status
```

### ex. select the status and name fields from the dataset table where the "id" field equals "1"
```
https://{DATA_SERVICE_HOST}/api/v1/db/dataset/query?id=1&select=status,name
```

### ex select the nested field dataset.datasources.type field from the dataset collection "datasources"
```
https://{DATA_SERVICE_HOST}/api/v1/db/dataset/query?select=dataset.datasources.type
```


