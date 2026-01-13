# dataset Entity Reference 

## field description 

### datasources
Collection of [datasource](./datasource.md) objects. Nested edits and datasets collections are always empty to prevent nested reference issues.

### id
Database record id. 

### latestTxid
The transaction id of the last datasourceEvent object processed by PatchFox for this dataset. 

### latestJobId
The id of the of the last pipeline job that occurred against this dataset. 

### name
The name of the dataset. 

### updatedAt
The timestamp when last this record was updated in POSIX format. 

### status
The job status of this dataset. 

## example API response

```
{
  "code": 200,
  "type": "SUCCESSFUL",
  "description": "OK",
  "txid": "737096c9-71ee-4cc3-a836-1f89aa0789f8",
  "requestReceivedAt": "2025-07-14T22:02:08.594334545Z",
  "data": {
    "titlePage": {
      "content": [
        {
          "datasources": [
            {
              "edits": [],
              "datasets": [],
              "id": 698,
              "latestTxid": "e27784c5-846a-4c5c-9873-381da88fa04a",
              "latestJobId": "fefe2ea7-1e6a-4289-ac2f-668e2cd5f721",
              "purl": "pkg:generic/github/contributors%3A%3Amain@pypi",
              "domain": "github",
              "name": "contributors::main",
              "commitBranch": "main",
              "type": "pypi",
              "numberEventsReceived": 67.0,
              "numberEventProcessingErrors": 0.0,
              "firstEventReceivedAt": 1745806515.117474000,
              "lastEventReceivedAt": 1745806611.907398000,
              "lastEventReceivedStatus": "ACCEPTED",
              "status": "PROCESSING",
              "packageIndexes": [
                2264,
                2266,
                2272,
                2268,
                2265,
                2269,
                2270,
                2267
              ]
            }
          ],
          "id": 1,
          "latestTxid": "5a9c8d32-b132-45c0-9d6f-6c2b945799a9",
          "latestJobId": "d2c6c2f4-af25-4fdd-886f-79762847ffff",
          "name": "github",
          "updatedAt": 1748829388.675668000,
          "status": "PROCESSING"
        }
      ],
      "pageable": {
        "pageNumber": 0,
        "pageSize": 20,
        "sort": {
          "sorted": false,
          "empty": true,
          "unsorted": true
        },
        "offset": 0,
        "unpaged": false,
        "paged": true
      },
      "last": true,
      "totalPages": 1,
      "totalElements": 1,
      "size": 20,
      "number": 0,
      "sort": {
        "sorted": false,
        "empty": true,
        "unsorted": true
      },
      "first": true,
      "numberOfElements": 1,
      "empty": false
    }
  }
}
```
