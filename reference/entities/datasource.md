# datasource Entity Reference 

## field description 

### edits
Collection of [edit](./edit.md) objects associated with this datasource. 

### datasets
Collection of [dataset](./dataset.md) objects associated with this datasource. Nested datasources collection is always empty to prevent nested reference issues. 

### id
Database record id.

### latestTxid
The transaction id of the last datasourceEvent object processed by PatchFox for this dataset. 

### latestJobId
The id of the of the last pipeline job that occurred against this dataset. 

### purl
The identifier for this datasource. See [PatchFox Data Nomenclature](./reference/pf_core_concepts/pf_data_nomenclature.md) for how PatchFox uses the pURL spec to identify datasources and datasourceEvents. 

### domain
The name of the deployment this datasource is associated with. 

### commitBranch
Datasources are source-controlled build files. This field represents the source control branch this datasource record represents.

### type
The type of build file represented by this datasource in pURL terms.

### numberEventsReceived
Number of [datasourceEvent](./datasourceEvent.md) objects sent by this datasource to PatchFox.  

### numberEventProcessingErrors
Number of [datasourceEvent](./datasourceEvent.md) objects sent by this datasource to PatchFox that resulted in an error and thus was not accepted by PatchFox.

### firstEventReceivedAt
DateTime in POSIX format of when the first datasourceEvent sent by this datasource was received by PatchFox.

### lastEventReceivedAt
DateTime in POSIX format of when the latest datasourceEvent sent by this datasource was received by PatchFox.

### lastEventReceivedStatus
HTTP status reported by PatchFox of whether or not the latest datasourceEvent was accepted by PatchFox for processing. 

### status
Current PatchFox processing job status for this datasource.

### packageIndexes
REMOVED BY API TOOL TO PREVENT CONTEXT BUFFER OVERFLOW -- Collection of [package](./package.md) record objects presently associated with this datasource. 


## example API response 

```
{
  "code": 200,
  "type": "SUCCESSFUL",
  "description": "OK",
  "txid": "c0c9c8f8-1c4b-4048-bf40-088a0c591fdd",
  "requestReceivedAt": "2025-07-14T22:15:26.921736379Z",
  "data": {
    "titlePage": {
      "content": [
        {
          "edits": [
            {
              "datasetMetrics": null,
              "datasource": null,
              "id": 1570,
              "commitDateTime": 1595467125.001000000,
              "eventDateTime": 1745635794.464974000,
              "editType": "CREATE",
              "before": "",
              "after": "pkg:golang/github.com/stretchr/testify@v1.2.2",
              "sameEditCount": 0,
              "criticalFindings": 0,
              "highFindings": 0,
              "mediumFindings": 0,
              "lowFindings": 0,
              "reduceCvesIndex": null,
              "reduceCveGrowthIndex": null,
              "reduceCveBacklogIndex": null,
              "reduceCveBacklogGrowthIndex": null,
              "reduceStalePackagesIndex": null,
              "reduceStalePackagesGrowthIndex": null,
              "reduceDownlevelPackagesIndex": null,
              "reduceDownlevelPackagesGrowthIndex": null,
              "growPatchEfficacyIndex": null,
              "removeRedundantPackagesIndex": null,
              "decreaseBacklogRank": null,
              "decreaseVulnerabilityCountRank": null,
              "avoidsVulnerabilitiesRank": null,
              "increaseImpactRank": null,
              "sameEdit": false,
              "userEdit": true,
              "pfRecommendedEdit": false
            }
          ],
          "datasets": [
            {
              "datasources": [],
              "id": 1,
              "latestTxid": "5a9c8d32-b132-45c0-9d6f-6c2b945799a9",
              "latestJobId": "d2c6c2f4-af25-4fdd-886f-79762847ffff",
              "name": "github",
              "updatedAt": 1748829388.675668000,
              "status": "PROCESSING"
            }
          ],
          "id": 2,
          "latestTxid": "ea590c25-abda-46fe-82aa-a141642292dc",
          "latestJobId": "fefe2ea7-1e6a-4289-ac2f-668e2cd5f721",
          "purl": "pkg:generic/github/emissary__vendor__github.com__sirupsen__logrus%3A%3Amaster@golang",
          "domain": "github",
          "name": "emissary__vendor__github.com__sirupsen__logrus::master",
          "commitBranch": "master",
          "type": "golang",
          "numberEventsReceived": 2.0,
          "numberEventProcessingErrors": 0.0,
          "firstEventReceivedAt": 1745635794.464974000,
          "lastEventReceivedAt": 1745635794.464974000,
          "lastEventReceivedStatus": "ACCEPTED",
          "status": "PROCESSING",
          "packageIndexes": [
            15,
            14,
            17,
            13,
            18,
            16
          ]
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