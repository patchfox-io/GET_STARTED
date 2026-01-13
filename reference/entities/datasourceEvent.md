# datasourceEvent Entity Reference

## field description 

### datasource
[datasource](./datasource.md) object from which this datasourceEvent came. Nested edits and datasets collections are always empty to prevent nested reference issues.

### packages
Collection of [package](./package.md) objects. Nested packages and findings collections within the top level findings collection are always empty to prevent nested reference issues. 

### id
Database record id. 

### purl
The identifier for this datasourceEvent. See [PatchFox Data Nomenclature](./reference/pf_core_concepts/pf_data_nomenclature.md) for how PatchFox uses the pURL spec to identify datasources and datasourceEvents. 

### txid
Transaction id representing the ingest by PatchFox of this datasourceEvent. 

### jobId
The id of the current or most recent job involving this datasourceEvent.

### commitHash
The git commit hash represented by this datasourceEvent.

### commmitBranch
The git branch from which this datasourceEvent eminated. 

### commitDateTime
DateTime stamp in POSIX format indicating when the git commit represented by this datasourceEvent occured. 

### eventDateTime
DataTime stamp in POSIX format indicating when PatchFox received this datasourceEvent. 

### payload
Binary field that's will always be a single random byte when returned in this manner. In the database it represents the gzipped, actual, payload sent to PatchFox for processing. 

### status
Current PatchFox processing job status for this datasourceEvent.

### processingError
Currently unused field.

### ossEnriched
Boolean indicating whether or not this datasourceEvent has undergone the OSS enrichment stage of PatchFox pipeline processing. 

### packageIndexEnriched
Boolean indicating whether or not this datasourceEvent has undergone the package-index enrichment stage of PatchFox pipeline processing. 

### analyzed
Boolean indicating whether or not this datasourceEvent has undergone the analyze stage of PatchFox pipeline processing. 

### forecasted 
Boolean indicating whether or not this datasourceEvent has undergone the forecast stage of PatchFox pipeline processing. 

### recommend 
Boolean indicating whether or not this datasourceEvent has undergone the recommend stage of PatchFox pipeline processing. 

### payloadNull
Curently unusued field.


## example API response 

```
{
  "code": 200,
  "type": "SUCCESSFUL",
  "description": "OK",
  "txid": "9c3882a3-c69e-4041-a88b-3ed24da60f94",
  "requestReceivedAt": "2025-07-15T14:43:41.322257033Z",
  "data": {
    "titlePage": {
      "content": [
        {
          "datasource": {
            "edits": [],
            "datasets": [],
            "id": 1,
            "latestTxid": "aa9e7f58-67e3-4494-a942-c826e1baa0a0",
            "latestJobId": "fefe2ea7-1e6a-4289-ac2f-668e2cd5f721",
            "purl": "pkg:generic/github/emissary%3A%3Amaster@golang",
            "domain": "github",
            "name": "emissary::master",
            "commitBranch": "master",
            "type": "golang",
            "numberEventsReceived": 2.0,
            "numberEventProcessingErrors": 0.0,
            "firstEventReceivedAt": 1745635789.907769000,
            "lastEventReceivedAt": 1745635789.907769000,
            "lastEventReceivedStatus": "ACCEPTED",
            "status": "PROCESSING",
            "packageIndexes": [
              3,
              4,
              1,
              12,
              9,
              8,
              7,
              10,
              2,
              11,
              5,
              6
            ]
          },
          "packages": [
            {
              "findings": [
                {
                  "packages": [],
                  "reporters": [
                    {
                      "findings": [],
                      "id": 1,
                      "name": "GRYPE"
                    }
                  ],
                  "data": {
                    "finding": null,
                    "id": 135,
                    "identifier": "GHSA-m425-mq94-257g",
                    "severity": "HIGH",
                    "description": "gRPC-Go HTTP/2 Rapid Reset vulnerability",
                    "cpes": [
                      "cpe:2.3:a:google:grpc:v1.27.1:*:*:*:*:*:*:*"
                    ],
                    "reportedAt": 1748829449.200639000,
                    "publishedAt": null,
                    "patchedIn": [
                      "1.56.3"
                    ]
                  },
                  "id": 135,
                  "identifier": "GHSA-m425-mq94-257g"
                },
                {
                  "packages": [],
                  "reporters": [
                    {
                      "findings": [],
                      "id": 1,
                      "name": "GRYPE"
                    }
                  ],
                  "data": {
                    "finding": null,
                    "id": 2,
                    "identifier": "CVE-2023-44487",
                    "severity": "MEDIUM",
                    "description": "HTTP/2 Stream Cancellation Attack",
                    "cpes": [
                      "cpe:2.3:a:golang:networking:v0.0.0-20190404232315-eb5bcb51f2a3:*:*:*:*:go:*:*"
                    ],
                    "reportedAt": 1748829390.289238000,
                    "publishedAt": null,
                    "patchedIn": [
                      "0.17.0"
                    ]
                  },
                  "id": 2,
                  "identifier": "CVE-2023-44487"
                }
              ],
              "criticalFindings": [],
              "highFindings": [
                {
                  "packages": [],
                  "reporters": [
                    {
                      "findings": [],
                      "id": 1,
                      "name": "GRYPE"
                    }
                  ],
                  "data": {
                    "finding": null,
                    "id": 135,
                    "identifier": "GHSA-m425-mq94-257g",
                    "severity": "HIGH",
                    "description": "gRPC-Go HTTP/2 Rapid Reset vulnerability",
                    "cpes": [
                      "cpe:2.3:a:google:grpc:v1.27.1:*:*:*:*:*:*:*"
                    ],
                    "reportedAt": 1748829449.200639000,
                    "publishedAt": null,
                    "patchedIn": [
                      "1.56.3"
                    ]
                  },
                  "id": 135,
                  "identifier": "GHSA-m425-mq94-257g"
                }
              ],
              "mediumFindings": [
                {
                  "packages": [],
                  "reporters": [
                    {
                      "findings": [],
                      "id": 1,
                      "name": "GRYPE"
                    }
                  ],
                  "data": {
                    "finding": null,
                    "id": 2,
                    "identifier": "CVE-2023-44487",
                    "severity": "MEDIUM",
                    "description": "HTTP/2 Stream Cancellation Attack",
                    "cpes": [
                      "cpe:2.3:a:golang:networking:v0.0.0-20190404232315-eb5bcb51f2a3:*:*:*:*:go:*:*"
                    ],
                    "reportedAt": 1748829390.289238000,
                    "publishedAt": null,
                    "patchedIn": [
                      "0.17.0"
                    ]
                  },
                  "id": 2,
                  "identifier": "CVE-2023-44487"
                }
              ],
              "lowFindings": [],
              "datasourceEvents": [],
              "id": 11,
              "purl": "pkg:golang/google.golang.org/grpc@v1.27.1",
              "type": "golang",
              "namespace": "google.golang.org",
              "name": "grpc",
              "version": "v1.27.1",
              "numberVersionsBehindHead": -1,
              "numberMajorVersionsBehindHead": -1,
              "numberMinorVersionsBehindHead": -1,
              "numberPatchVersionsBehindHead": -1,
              "mostRecentVersion": null,
              "mostRecentVersionPublishedAt": null,
              "thisVersionPublishedAt": null,
              "updatedAt": 1745635789.907769000
            }
          ],
          "id": 1,
          "purl": "pkg:generic/github/emissary%3A%3Amaster@golang?commitdatetime=2020-07-23T01%3A18%3A45%2B00%3A00&commithash=2e953969de21612061901e922d103391512a1ff2",
          "txid": "aa9e7f58-67e3-4494-a942-c826e1baa0a0",
          "jobId": "d2c6c2f4-af25-4fdd-886f-79762847ffff",
          "commitHash": "2e953969de21612061901e922d103391512a1ff2",
          "commitBranch": "master",
          "commitDateTime": 1595467125.000000000,
          "eventDateTime": 1745635789.907769000,
          "payload": "AA==",
          "status": "READY_FOR_NEXT_PROCESSING",
          "processingError": null,
          "ossEnriched": true,
          "packageIndexEnriched": true,
          "analyzed": true,
          "forecasted": false,
          "recommended": false,
          "payloadNull": false
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
      "totalPages": 1,
      "totalElements": 1,
      "last": true,
      "size": 20,
      "number": 0,
      "sort": {
        "sorted": false,
        "empty": true,
        "unsorted": true
      },
      "numberOfElements": 1,
      "first": true,
      "empty": false
    }
  }
}
```