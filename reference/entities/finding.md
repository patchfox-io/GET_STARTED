# finding Entity Reference 

## field description

### packages
Collection of [package](./package.md) objects to which this finding is associated. Nested finding collections are always empty to prevent nested reference issues. 

### reporters
Collection of [findingReporter](./findingReporter.md) objects representing every tool that reported this finding. Nested finding collections are always empty to prevent nested reference issues. 

### data
[findingData](./findingData.md) object representing a summary of the finding. 

## example API response 

```
{
  "code": 200,
  "type": "SUCCESSFUL",
  "description": "OK",
  "txid": "c71b76a8-f6ea-477b-9666-9cf36410f17e",
  "requestReceivedAt": "2025-07-14T19:57:36.090334118Z",
  "data": {
    "titlePage": {
      "content": [
        {
          "packages": [
            {
              "findings": [],
              "criticalFindings": [],
              "highFindings": [],
              "mediumFindings": [],
              "lowFindings": [],
              "datasourceEvents": [],
              "id": 39264,
              "purl": "pkg:maven/junit/junit@4.11",
              "type": "maven",
              "namespace": "junit",
              "name": "junit",
              "version": "4.11",
              "numberVersionsBehindHead": 12,
              "numberMajorVersionsBehindHead": 0,
              "numberMinorVersionsBehindHead": 2,
              "numberPatchVersionsBehindHead": -1,
              "mostRecentVersion": "4.13.2",
              "mostRecentVersionPublishedAt": 1613233914.000000000,
              "thisVersionPublishedAt": 1352920907.000000000,
              "updatedAt": 1748829391.520550000
            },
            {
              "findings": [],
              "criticalFindings": [],
              "highFindings": [],
              "mediumFindings": [],
              "lowFindings": [],
              "datasourceEvents": [],
              "id": 36659,
              "purl": "pkg:maven/junit/junit@4.12",
              "type": "maven",
              "namespace": "junit",
              "name": "junit",
              "version": "4.12",
              "numberVersionsBehindHead": 8,
              "numberMajorVersionsBehindHead": 0,
              "numberMinorVersionsBehindHead": 1,
              "numberPatchVersionsBehindHead": -1,
              "mostRecentVersion": "4.13.2",
              "mostRecentVersionPublishedAt": 1613233914.000000000,
              "thisVersionPublishedAt": 1417709863.000000000,
              "updatedAt": 1748829391.489523000
            }
          ],
          "reporters": [
            {
              "findings": [],
              "id": 1,
              "name": "GRYPE"
            }
          ],
          "data": {
            "finding": null,
            "id": 17,
            "identifier": "CVE-2020-15250",
            "severity": "MEDIUM",
            "description": "TemporaryFolder on unix-like systems does not limit access to created files",
            "cpes": [
              "cpe:2.3:a:junit:junit:4.11:*:*:*:*:*:*:*"
            ],
            "reportedAt": 1748829389.979435000,
            "publishedAt": null,
            "patchedIn": [
              "4.13.1"
            ]
          },
          "id": 17,
          "identifier": "CVE-2020-15250"
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