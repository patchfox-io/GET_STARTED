# edit Entity Reference

## field description

### datasetMetrics
[datasetMetrics](./datasetMetrics.md) object to which this edit object is attached. Nested datasources, edits, packageFamilies collections are always empty to prevent circular reference issues. 

### id
Database record id. 

### commitDateTime
DateTime stamp in POSIX format indicating when the git commit represented by this datasourceEvent occured. 

### eventDateTime
DataTime stamp in POSIX format indicating when PatchFox received the datasourceEvent tied to this edit record.

### editType
The type of edit represented by this record. One of CREATE, UPDATE, DELETE.

### before
The package pURL before the edit took place. Is empty in case of CREATE edit. 

### after
The package pURL after the edit took place.

### sameEditCount
Count of how many times this edit was made in the previous 90 days. Used as part of PES calculation. See [PatchFox Custom Metrics](./reference/custom_metrics.md) for details as to what PES is. 

### criticalFindings
Count of how many critical findings are associated with the package represented by the "after" field. 

### highFindings
Count of how many high findings are associated with the package represented by the "after" field. 

### mediumFindings
Count of how many medium findings are associated with the package represented by the "after" field. 

### lowFindings
Count of how many low findings are associated with the package represented by the "after" field. 

### reduceCvesIndex
If this edit is associated with a [datasetMetrics](./datasetMetrics.md) record that has the "isForecastRecommendationTaken" flag set this edit record indicates a PatchFox recommended patch and this field will be populated. Indicates degree to which, with respect to all other packages currently in the dataset, this edit will reduce the number of findings in the dataset. 

### reduceCveGrowthIndex
If this edit is associated with a [datasetMetrics](./datasetMetrics.md) record that has the "isForecastRecommendationTaken" flag set this edit record indicates a PatchFox recommended patch and this field will be populated. Indicates degree to which, with respect to all other packages currently in the dataset, this edit will reduce the rate of growth of the number of findings in the dataset. 

### reduceCveBacklogIndex
If this edit is associated with a [datasetMetrics](./datasetMetrics.md) record that has the "isForecastRecommendationTaken" flag set this edit record indicates a PatchFox recommended patch and this field will be populated. Indicates degree to which, with respect to all other packages currently in the dataset, this edit will reduce the number of findings in the backlog the dataset. 

### reduceCveBacklogGrowthIndex
If this edit is associated with a [datasetMetrics](./datasetMetrics.md) record that has the "isForecastRecommendationTaken" flag set this edit record indicates a PatchFox recommended patch and this field will be populated. Indicates degree to which, with respect to all other packages currently in the dataset, this edit will reduce the growth of the number of findings in the backlog in the dataset. 

### reduceStalePackagesIndex
If this edit is associated with a [datasetMetrics](./datasetMetrics.md) record that has the "isForecastRecommendationTaken" flag set this edit record indicates a PatchFox recommended patch and this field will be populated. Indicates degree to which, with respect to all other packages currently in the dataset, this edit will reduce the number of stale packages in the dataset. 

### reduceStalePackagesGrowthIndex
If this edit is associated with a [datasetMetrics](./datasetMetrics.md) record that has the "isForecastRecommendationTaken" flag set this edit record indicates a PatchFox recommended patch and this field will be populated. Indicates degree to which, with respect to all other packages currently in the dataset, this edit will reduce the growth of the number of stale packages in the dataset. 

### reduceDownlevelPackagesIndex
If this edit is associated with a [datasetMetrics](./datasetMetrics.md) record that has the "isForecastRecommendationTaken" flag set this edit record indicates a PatchFox recommended patch and this field will be populated. Indicates degree to which, with respect to all other packages currently in the dataset, this edit will reduce the number of downlevel packages in the dataset. 

### reduceDownlevelPackagesGrowthIndex
If this edit is associated with a [datasetMetrics](./datasetMetrics.md) record that has the "isForecastRecommendationTaken" flag set this edit record indicates a PatchFox recommended patch and this field will be populated. Indicates degree to which, with respect to all other packages currently in the dataset, this edit will reduce the growth of the number of downlevel packages in the dataset. 

### growPatchEfficacyIndex
If this edit is associated with a [datasetMetrics](./datasetMetrics.md) record that has the "isForecastRecommendationTaken" flag set this edit record indicates a PatchFox recommended patch and this field will be populated. Indicates degree to which, with respect to all other packages currently in the dataset, this edit will improve patch efficacy. See [PatchFox Custom Metrics](./reference/custom_metrics.md) for more on Patch Efficacy.

### removeRedundantPackagesIndex
If this edit is associated with a [datasetMetrics](./datasetMetrics.md) record that has the "isForecastRecommendationTaken" flag set this edit record indicates a PatchFox recommended patch and this field will be populated. Indicates degree to which, with respect to all other packages currently in the dataset, this edit will reduce redundant packages. See [PatchFox Custom Metrics](./reference/custom_metrics.md) for more on Redundant Packages.

### decreaseBacklogRank
If this edit is associated with a [datasetMetrics](./datasetMetrics.md) record that has the "isForecastRecommendationTaken" flag set this edit record indicates a PatchFox recommended patch and this field will be populated. Indicates degree to which, with respect to all other edit recommentations, this edit will reduce the number of findings in the backlog the dataset. 

### decreaseVulnerabilityCountRank
If this edit is associated with a [datasetMetrics](./datasetMetrics.md) record that has the "isForecastRecommendationTaken" flag set this edit record indicates a PatchFox recommended patch and this field will be populated. Indicates degree to which, with respect to all other edit recommendations, this edit will reduce the number of findings in the dataset. 

### avoidsVulnerabilitiesRank
If this edit is associated with a [datasetMetrics](./datasetMetrics.md) record that has the "isForecastRecommendationTaken" flag set this edit record indicates a PatchFox recommended patch and this field will be populated. Indicates degree to which, with respect to all other edit recommendations, this edit will reduce the number of findings in the dataset. 

### increaseImpactRank
If this edit is associated with a [datasetMetrics](./datasetMetrics.md) record that has the "isForecastRecommendationTaken" flag set this edit record indicates a PatchFox recommended patch and this field will be populated. Indicates degree to which, with respect to all other edit recommendations, this edit will increase patch impact. See [PatchFox Custom Metrics](./reference/custom_metrics.md) for more on patch impact.

### sameEdit
Boolean indicating whether or not this edit has been made at least twice before in the previous 90 days. 

### pfRecommendedEdit
Indicates whether or not this is a PatchFox recommendation edit. 

### userEdit
Boolean indicating whether or not this is an actual edit made by way of a git commit to a datasource (ie - a source controlled build file). 

## example API response 

```
{
  "code": 200,
  "type": "SUCCESSFUL",
  "description": "OK",
  "txid": "1b4eb2da-274f-4e17-a15a-6167bbf223f8",
  "requestReceivedAt": "2025-07-15T17:38:40.179144211Z",
  "data": {
    "titlePage": {
      "content": [
        {
          "datasetMetrics": {
            "dataset": {
              "datasources": [],
              "id": 1,
              "latestTxid": "5a9c8d32-b132-45c0-9d6f-6c2b945799a9",
              "latestJobId": "d2c6c2f4-af25-4fdd-886f-79762847ffff",
              "name": "github",
              "updatedAt": 1748829388.675668000,
              "status": "PROCESSING"
            },
            "edits": [],
            "id": 1,
            "packageFamilies": [],
            "packageIndexes": [
              39813
            ],
            "datasourceCount": 702,
            "datasourceEventCount": 1,
            "commitDateTime": 1303260295.000000000,
            "eventDateTime": 1745739017.745176000,
            "forecastMaturityDate": null,
            "txid": "1f2468ed-8e51-40fe-a799-58f51d0e4fbb",
            "jobId": "d2c6c2f4-af25-4fdd-886f-79762847ffff",
            "recommendationType": null,
            "recommendationHeadline": null,
            "rpsScore": 0.0,
            "totalFindings": 0,
            "criticalFindings": 0,
            "highFindings": 0,
            "mediumFindings": 0,
            "lowFindings": 0,
            "findingsAvoidedByPatchingPastYear": 0,
            "criticalFindingsAvoidedByPatchingPastYear": 0,
            "highFindingsAvoidedByPatchingPastYear": 0,
            "mediumFindingsAvoidedByPatchingPastYear": 0,
            "lowFindingsAvoidedByPatchingPastYear": 0,
            "findingsInBacklogBetweenThirtyAndSixtyDays": 0.0,
            "criticalFindingsInBacklogBetweenThirtyAndSixtyDays": 0.0,
            "highFindingsInBacklogBetweenThirtyAndSixtyDays": 0.0,
            "mediumFindingsInBacklogBetweenThirtyAndSixtyDays": 0.0,
            "lowFindingsInBacklogBetweenThirtyAndSixtyDays": 0.0,
            "findingsInBacklogBetweenSixtyAndNinetyDays": 0.0,
            "criticalFindingsInBacklogBetweenSixtyAndNinetyDays": 0.0,
            "highFindingsInBacklogBetweenSixtyAndNinetyDays": 0.0,
            "mediumFindingsInBacklogBetweenSixtyAndNinetyDays": 0.0,
            "lowFindingsInBacklogBetweenSixtyAndNinetyDays": 0.0,
            "findingsInBacklogOverNinetyDays": 0.0,
            "criticalFindingsInBacklogOverNinetyDays": 0.0,
            "highFindingsInBacklogOverNinetyDays": 0.0,
            "mediumFindingsInBacklogOverNinetyDays": 0.0,
            "lowFindingsInBacklogOverNinetyDays": 0.0,
            "packages": 1,
            "packagesWithFindings": 0,
            "packagesWithCriticalFindings": 0,
            "packagesWithHighFindings": 0,
            "packagesWithMediumFindings": 0,
            "packagesWithLowFindings": 0,
            "downlevelPackages": 0,
            "downlevelPackagesMajor": 0,
            "downlevelPackagesMinor": 0,
            "downlevelPackagesPatch": 0,
            "stalePackages": 0,
            "stalePackagesSixMonths": 0,
            "stalePackagesOneYear": 0,
            "stalePackagesOneYearSixMonths": 0,
            "stalePackagesTwoYears": 0,
            "patches": 0,
            "samePatches": 0,
            "differentPatches": 0,
            "patchFoxPatches": 0,
            "patchEfficacyScore": 0.0,
            "patchImpact": 0.0,
            "patchEffort": 0.0,
            "current": true,
            "forecastSameCourse": false,
            "forecastRecommendationsTaken": false
          },
          "datasource": {
            "edits": [],
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
            "id": 239,
            "latestTxid": "d245198b-e0ab-4524-9c0e-74438fe79c52",
            "latestJobId": "fefe2ea7-1e6a-4289-ac2f-668e2cd5f721",
            "purl": "pkg:generic/github/albino%3A%3Amaster@gem",
            "domain": "github",
            "name": "albino::master",
            "commitBranch": "master",
            "type": "gem",
            "numberEventsReceived": 18.0,
            "numberEventProcessingErrors": 0.0,
            "firstEventReceivedAt": 1745739017.745176000,
            "lastEventReceivedAt": 1745739039.672965000,
            "lastEventReceivedStatus": "ACCEPTED",
            "status": "PROCESSING",
            "packageIndexes": [
              39813
            ]
          },
          "id": 1,
          "commitDateTime": 1303260295.000000000,
          "eventDateTime": 1745739017.745176000,
          "editType": "CREATE",
          "before": "",
          "after": "pkg:gem/albino@1.3.3",
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
          "pfRecommendedEdit": false,
          "userEdit": true
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