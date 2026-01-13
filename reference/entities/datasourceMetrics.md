## datasourceMetrics Entity Reference

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

### txid
Transaction id associated with the [datasourceEvent](./datasourceEvent.md) being applied to to the [dataset](./dataset.md) represented by this datasetMetrics record. 

### jobId
The id of the PatchFox job that caused this datasetMetrics record to be created. 

### purl
The identifier for this datasource. See [PatchFox Data Nomenclature](./reference/pf_core_concepts/pf_data_nomenclature.md) for how PatchFox uses the pURL spec to identify datasources and datasourceEvents. 

### rpsScore
The delta of Redundant Package Score (RPS) for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. For more information on RPS and other PatchFox custom metrics see [PatchFox Custom Metrics](./reference/custom_metrics.md).

### totalFindings
The delta represented by this commit to this datasource of findings for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### criticalFindings
The delta represented by this commit to this datasource of critical findings for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### highFindings
The delta represented by this commit to this datasource of high findings for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### mediumFindings
The delta represented by this commit to this datasource of medium findings for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### lowFindings
The delta represented by this commit to this datasource of low findings for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### findingsAvoidedByPatchingPastYear
The delta represented by this commit to this datasource of findings avoided by patching in the previous year for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. NOT CURRENTLY SUPPORTED BY THE DATA PIPELINE. 

### criticalFindingsAvoidedByPatchingPastYear
The delta represented by this commit to this datasource of critical findings avoided by patching in the previous year for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. NOT CURRENTLY SUPPORTED BY THE DATA PIPELINE. 

### highFindingsAvoidedByPatchingPastYear
The delta represented by this commit to this datasource of high findings avoided by patching in the previous year for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. NOT CURRENTLY SUPPORTED BY THE DATA PIPELINE. 

### mediumFindingsAvoidedByPatchingPastYear
The delta represented by this commit to this datasource of medium findings avoided by patching in the previous year for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. NOT CURRENTLY SUPPORTED BY THE DATA PIPELINE. 

### lowFindingsAvoidedByPatchingPastYear
The delta represented by this commit to this datasource of low findings avoided by patching in the previous year for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. NOT CURRENTLY SUPPORTED BY THE DATA PIPELINE. 

### findingsInBacklogBetweenThirtyAndSixtyDays
Delta findings that have been present in the dataset between thirty and sixty days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### criticalFindingsInBacklogBetweenThirtyAndSixtyDays
Delta critical findings that have been present in the dataset between thirty and sixty days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### highFindingsInBacklogBetweenThirtyAndSixtyDays
Delta high findings that have been present in the dataset between thirty and sixty days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### mediumFindingsInBacklogBetweenThirtyAndSixtyDays
Delta medium findings that have been present in the dataset between thirty and sixty days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### lowFindingsInBacklogBetweenThirtyAndSixtyDays
Delta low findings that have been present in the dataset between thirty and sixty days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### findingsInBacklogBetweenSixtyAndNinetyDays
Delta findings that have been present in the dataset between sixty and ninety days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### criticalFindingsInBacklogBetweenSixtyAndNinetyDays
Delta critical findings that have been present in the dataset between sixty and ninety days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### highFindingsInBacklogBetweenSixtyAndNinetyDays
Delta high findings that have been present in the dataset between sixty and ninety days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### mediumFindingsInBacklogBetweenSixtyAndNinetyDays
Delta medium findings that have been present in the dataset between sixty and ninety days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### lowFindingsInBacklogBetweenSixtyAndNinetyDays
Delta low findings that have been present in the dataset between sixty and ninety days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### findingsInBacklogOverNinetyDays
Delta findings that have been present in the dataset over ninety days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### criticalFindingsInBacklogOverNinetyDays
Delta critical findings that have been present in the dataset over ninety days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### highFindingsInBacklogOverNinetyDays
Delta high findings that have been present in the dataset over ninety days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### mediumFindingsInBacklogOverNinetyDays
Delta medium findings that have been present in the dataset over ninety days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### lowFindingsInBacklogOverNinetyDays
Delta low findings that have been present in the dataset over ninety days for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### packages
delta represented by this commit to this datasource of packages for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### packagesWithFindings
delta represented by this commit to this datasource of packages with findings for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### packagesWithCriticalFindings
delta represented by this commit to this datasource of packages with critical findings for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### packagesWithHighFindings
delta represented by this commit to this datasource of packages with high findings for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### packagesWithMediumFindings
delta represented by this commit to this datasource of packages with medium findings for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### packagesWithLowFindings
delta represented by this commit to this datasource of packages with low findings for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### downlevelPackages
delta represented by this commit to this datasource of downlevel packages for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. See [PatchFox Custom Metrics](./reference/custom_metrics.md) for more information on what a "downlevel package" is. 

### downlevelPackagesMajor
delta represented by this commit to this datasource of downlevel packages by semantic versioning major for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. See [PatchFox Custom Metrics](./reference/custom_metrics.md) for more information on what a "downlevel package" is. 

### downlevelPackagesMinor
delta represented by this commit to this datasource of downlevel packages by semantic versioning minor for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. See [PatchFox Custom Metrics](./reference/custom_metrics.md) for more information on what a "downlevel package" is. 

### downlevelPackagesPatch
delta represented by this commit to this datasource of downlevel packages by semantic versioning patch for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. See [PatchFox Custom Metrics](./reference/custom_metrics.md) for more information on what a "downlevel package" is. 

### stalePackages
delta represented by this commit to this datasource of stale packages for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. See [PatchFox Custom Metrics](./reference/custom_metrics.md) for more information on what a "stale package" is. 

### stalePackagesSixMonths
delta represented by this commit to this datasource of stale packages for more than six months for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. See [PatchFox Custom Metrics](./reference/custom_metrics.md) for more information on what a "stale package" is. 

### stalePackagesOneYear
delta represented by this commit to this datasource of stale packages for more than one year for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. See [PatchFox Custom Metrics](./reference/custom_metrics.md) for more information on what a "stale package" is. 

### stalePackagesOneYearSixMonths
delta represented by this commit to this datasource of stale packages for more than one year six months for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. See [PatchFox Custom Metrics](./reference/custom_metrics.md) for more information on what a "stale package" is. 

### stalePackagesTwoYears
delta represented by this commit to this datasource of stale packages for more than two years for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. See [PatchFox Custom Metrics](./reference/custom_metrics.md) for more information on what a "stale package" is. 

### patches
delta represented by this commit to this datasource of patches for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### samePatches
delta represented by this commit to this datasource of patches where that patch has been previously made at least twice before in the previous six months for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### differentPatches
delta represented by this commit to this datasource of patches where that patch has been previously made less than twice before in the previous six months for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### patchFoxPatches
delta represented by this commit to this datasource of patches that were recommended by PatchFox to make in the previous six months for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### patchEfficacyScore
The current PES score for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. For more on PES see [PatchFox Custome Metrics](./reference/custom_metrics.md).

### patchImpact
The current patch impact score for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. Patch impact score is a sub measure of PES that measures the positive effect of a given patch. For more on PES see [PatchFox Custome Metrics](./reference/custom_metrics.md).

### patchEffort
The current patch effort score for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record. Patch effort score is a sub measure of PES that measures the positive effect of a given patch. For more on PES see [PatchFox Custome Metrics](./reference/custom_metrics.md).


## example API response 

```
{
  "code": 200,
  "type": "SUCCESSFUL",
  "description": "OK",
  "txid": "5a000839-6c3f-4d3b-99f0-06ddc35c0ecc",
  "requestReceivedAt": "2025-08-23T00:59:02.440824709Z",
  "data": {
    "titlePage": {
      "content": [
        {
          "id": 1,
          "datasourceEventCount": 1,
          "commitDateTime": 1303260295.000000000,
          "eventDateTime": 1745739017.745176000,
          "txid": "1f2468ed-8e51-40fe-a799-58f51d0e4fbb",
          "jobId": "d2c6c2f4-af25-4fdd-886f-79762847ffff",
          "purl": "pkg:generic/github/albino%3A%3Amaster@gem",
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
          "patches": 1,
          "samePatches": 0,
          "differentPatches": 1,
          "patchFoxPatches": 0,
          "patchEfficacyScore": 0.0,
          "patchImpact": 0.0,
          "patchEffort": 0.0
        }
      ],
      "pageable": {
        "pageNumber": 0,
        "pageSize": 1,
        "sort": {
          "sorted": false,
          "empty": true,
          "unsorted": true
        },
        "offset": 0,
        "unpaged": false,
        "paged": true
      },
      "totalPages": 600,
      "totalElements": 600,
      "last": false,
      "size": 1,
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
