# datasourceMetricsCurrent Entity Reference

### id
Database record id. 

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

### patches
Total number of patches for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### samePatches
Total number of patches where that patch has been previously made at least twice before in the previous six months for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### differentPatches
Total number of patches where that patch has been previously made less than twice before in the previous six months for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.

### patchFoxPatches
Total number of patches that were recommended by PatchFox to make in the previous six months for the [dataset](./dataset.md) represented by this datasetMetrics record at the point this point in time given the update represented by the [datasourceEvent](./datasourceEvent.md) associated with the txid specified in this record.


## example API response 

```
{
  "code": 200,
  "type": "SUCCESSFUL",
  "description": "OK",
  "txid": "e1462b4e-91dc-4bb4-b131-13c1aef092c3",
  "requestReceivedAt": "2025-08-23T00:42:13.354072657Z",
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
          "patches": 1,
          "samePatches": 0,
          "differentPatches": 1,
          "patchFoxPatches": 0
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
