## datasetMetrics Entity Reference

### field description 

### dataset
[dataset](./dataset.md) object. Nested datasources collection is always empty to prevent circular reference issues. 

### edits
Collection of [edit](./edit.md) objects associated with this dataset. Always empty to prevent circular reference issues. 

### id
Database record id. 

### packageFamilies
Collection of [package](./package.md) family purls (package purl sans a version). Always empty to reduce payload size.

### packageIndexes
REMOVED BY API TOOL TO PREVENT CONTEXT BUFFER OVERFLOW -- Collection of [package](./package.md) record indexes. Always empty to reduce payload size. 

### datasourceCount
Number of [datasource](./datasource.md)s associated with the [dataset](./dataset.md) represented by this datasetMetrics object. 

### datasourceEventCount
Number of [datasourceEvent](./datasourceEvent.md) objects associated with the [dataset](./dataset.md) represented by this datasetMetrics object. 

### commitDateTime
DateTime stamp in POSIX format indicating when the git commit represented by this datasourceEvent occured. 

### eventDateTime
DataTime stamp in POSIX format indicating when PatchFox received the datasourceEvent tied to this edit record.

### forecastMaturityDate
If populated, this datasetMetrics record represents a projection about the state of the [dataset](./dataset.md) represented by this datasetMetrics object at a future time. Field represents the datetime, in POSIX format, of the future date at which the projection is projected to occur. 

### txid
Transaction id associated with the [datasourceEvent](./datasourceEvent.md) being applied to to the [dataset](./dataset.md) represented by this datasetMetrics record. 

### jobId
The id of the PatchFox job that caused this datasetMetrics record to be created. 


### recommendationType
If this record is flagged as either "forecastSameCourse" or "forecastRecommendationsTaken" this field will be populated with one of 
```
    REDUCE_CVES
    REDUCE_CVE_GROWTH
    REDUCE_CVE_BACKLOG
    REDUCE_CVE_BACKLOG_GROWTH
    REDUCE_STALE_PACKAGES
    REDUCE_STALE_PACKAGES_GROWTH
    REDUCE_DOWNLEVEL_PACKAGES
    REDUCE_DOWNLEVEL_PACKAGES_GROWTH
    GROW_PATCH_EFFICACY
    REMOVE_REDUNDANT_PACKAGES
```
indicating the type of recommendation this record represents. For more on how PatchFox makes recommendations see [PatchFox Recommendations](./reference/pf_core_concepts/pf_how_rec.md)

### recommendationHeadline
The Headline text to be displayed by the recommend view to the user. This is germane only to the PatchFox Angular SaaS app. 

### rpsScore
Redundant Package Score (RPS) for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. For more information on RPS and other PatchFox custom metrics see [PatchFox Custom Metrics](./reference/custom_metrics.md).


### totalFindings
The total number of findings for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.


### criticalFindings
The total number of critical findings for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.


### highFindings
The total number of high findings for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### mediumFindings
The total number of medium findings for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### lowFindings
The total number of low findings for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### findingsAvoidedByPatchingPastYear
The total number of findings avoided by patching in the previous year for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. NOT CURRENTLY SUPPORTED BY THE DATA PIPELINE. 

### criticalFindingsAvoidedByPatchingPastYear
The total number of critical findings avoided by patching in the previous year for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. NOT CURRENTLY SUPPORTED BY THE DATA PIPELINE. 

### highFindingsAvoidedByPatchingPastYear
The total number of high findings avoided by patching in the previous year for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. NOT CURRENTLY SUPPORTED BY THE DATA PIPELINE. 

### mediumFindingsAvoidedByPatchingPastYear
The total number of medium findings avoided by patching in the previous year for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. NOT CURRENTLY SUPPORTED BY THE DATA PIPELINE. 

### lowFindingsAvoidedByPatchingPastYear
The total number of low findings avoided by patching in the previous year for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. NOT CURRENTLY SUPPORTED BY THE DATA PIPELINE. 

### findingsInBacklogBetweenThirtyAndSixtyDays
Total findings that have been present in the dataset between thirty and sixty days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### criticalFindingsInBacklogBetweenThirtyAndSixtyDays
Total critical findings that have been present in the dataset between thirty and sixty days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### highFindingsInBacklogBetweenThirtyAndSixtyDays
Total high findings that have been present in the dataset between thirty and sixty days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### mediumFindingsInBacklogBetweenThirtyAndSixtyDays
Total medium findings that have been present in the dataset between thirty and sixty days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### lowFindingsInBacklogBetweenThirtyAndSixtyDays
Total low findings that have been present in the dataset between thirty and sixty days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### findingsInBacklogBetweenSixtyAndNinetyDays
Total findings that have been present in the dataset between sixty and ninety days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### criticalFindingsInBacklogBetweenSixtyAndNinetyDays
Total critical findings that have been present in the dataset between sixty and ninety days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### highFindingsInBacklogBetweenSixtyAndNinetyDays
Total high findings that have been present in the dataset between sixty and ninety days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### mediumFindingsInBacklogBetweenSixtyAndNinetyDays
Total medium findings that have been present in the dataset between sixty and ninety days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### lowFindingsInBacklogBetweenSixtyAndNinetyDays
Total low findings that have been present in the dataset between sixty and ninety days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### findingsInBacklogOverNinetyDays
Total findings that have been present in the dataset over ninety days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### criticalFindingsInBacklogOverNinetyDays
Total critical findings that have been present in the dataset over ninety days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### highFindingsInBacklogOverNinetyDays
Total high findings that have been present in the dataset over ninety days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### mediumFindingsInBacklogOverNinetyDays
Total medium findings that have been present in the dataset over ninety days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### lowFindingsInBacklogOverNinetyDays
Total low findings that have been present in the dataset over ninety days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### packages
Total number of packages for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### packagesWithFindings
Total number of packages with findings for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### packagesWithCriticalFindings
Total number of packages with critical findings for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### packagesWithHighFindings
Total number of packages with high findings for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### packagesWithMediumFindings
Total number of packages with medium findings for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### packagesWithLowFindings
Total number of packages with low findings for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### downlevelPackages
Total number of downlevel packages for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. See [PatchFox Custom Metrics](./reference/custom_metrics.md) for more information on what a "downlevel package" is. 

### downlevelPackagesMajor
Total number of downlevel packages by semantic versioning major for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. See [PatchFox Custom Metrics](./reference/custom_metrics.md) for more information on what a "downlevel package" is. 

### downlevelPackagesMinor
Total number of downlevel packages by semantic versioning minor for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. See [PatchFox Custom Metrics](./reference/custom_metrics.md) for more information on what a "downlevel package" is. 

### downlevelPackagesPatch
Total number of downlevel packages by semantic versioning patch for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. See [PatchFox Custom Metrics](./reference/custom_metrics.md) for more information on what a "downlevel package" is. 

### stalePackages
Total number of stale packages for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. See [PatchFox Custom Metrics](./reference/custom_metrics.md) for more information on what a "stale package" is. 

### stalePackagesSixMonths
Total number of stale packages for more than six months for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. See [PatchFox Custom Metrics](./reference/custom_metrics.md) for more information on what a "stale package" is. 


### stalePackagesOneYear
Total number of stale packages for more than one year for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. See [PatchFox Custom Metrics](./reference/custom_metrics.md) for more information on what a "stale package" is. 


### stalePackagesOneYearSixMonths
Total number of stale packages for more than one year six months for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. See [PatchFox Custom Metrics](./reference/custom_metrics.md) for more information on what a "stale package" is. 


### stalePackagesTwoYears
Total number of stale packages for more than two years for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. See [PatchFox Custom Metrics](./reference/custom_metrics.md) for more information on what a "stale package" is. 


### patches
Total number of patches for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### samePatches
Total number of patches where that patch has been previously made at least twice before in the previous six months for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### differentPatches
Total number of patches where that patch has been previously made less than twice before in the previous six months for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### patchFoxPatches
Total number of patches that were recommended by PatchFox to make in the previous six months for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### patchEfficacyScore
The current PES score for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. For more on PES see [PatchFox Custome Metrics](./reference/custom_metrics.md).

### patchImpact
The current patch impact score for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. Patch impact score is a sub measure of PES that measures the positive effect of a given patch. For more on PES see [PatchFox Custome Metrics](./reference/custom_metrics.md).

### patchEffort
The current patch effort score for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. Patch effort score is a sub measure of PES that measures the positive effect of a given patch. For more on PES see [PatchFox Custome Metrics](./reference/custom_metrics.md).

### current
Boolean field indicating whether or not this datasetMetrics record is representative of actual events. 

### forecastSameCourse
Booelan field indicating whether or not this datasetMetrics record is representative of PatchFox's forecast as to where the metrics will be at "forecastMaturityDate".

### forecastRecommendationsTaken
Booelan field indicating whether or not this datasetMetrics record is representative of PatchFox's forecast as to where the metrics will be if the user applies all recommendations made by PatchFox for this dataset. 

## example API response 

```
{
  "code": 200,
  "type": "SUCCESSFUL",
  "description": "OK",
  "txid": "3d231d33-d516-47ac-83d8-f56f22c4f36b",
  "requestReceivedAt": "2025-07-15T18:50:28.250318513Z",
  "data": {
    "titlePage": {
      "content": [
        {
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
          "packageIndexes": [],
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
