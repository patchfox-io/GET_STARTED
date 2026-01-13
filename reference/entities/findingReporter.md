# findingReporter Entity Reference

## field description

### findings 
Collection of [finding](./finding.md) objects. Nested collections packages, reporters are always empty as is data object to to prevent nested reference issues. 

### id
Database record id.

### name
The name of the reporter of the finding. 

## example API response

```
{
  "code": 200,
  "type": "SUCCESSFUL",
  "description": "OK",
  "txid": "85955d75-df80-4b66-b92c-b7b823cf0e65",
  "requestReceivedAt": "2025-07-14T21:14:36.248178233Z",
  "data": {
    "titlePage": {
      "content": [
        {
          "findings": [
            {
              "packages": [],
              "reporters": [],
              "data": null,
              "id": 1125,
              "identifier": "CVE-2025-48068"
            }
          ],
          "id": 1,
          "name": "GRYPE"
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
