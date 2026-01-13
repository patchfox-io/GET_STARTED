# Package Entity Reference 

## field description

### findings 
A collection of Finding entities associated with this package. See [finding Entity Reference](./finding.md) for finding entity description.

### criticalFindings
A collection of Finding entities associated with this package where Finding.severity is "CRITICAL". See [finding Entity Reference](./finding.md) for Finding entity description.

### highFindings
A collection of Finding entities associated with this package where Finding.severity is "HIGH". See [finding Entity Reference](./finding.md) for Finding entity description.

### mediumFindings
A collection of Finding entities associated with this package where Finding.severity is "MEDIUM". See [finding Entity Reference](./finding.md) for Finding entity description.

### lowFindings
A collection of Finding entities associated with this package where Finding.severity is "LOW". See [finding Entity Reference](./finding.md) for Finding entity description.

### datasourceEvents
A collection of DatasourceEvent objects associated with this package. WILL ALWAYS BE AN EMPTY LIST. See [datasourceEvent Entity Reference](./datasourceEvent.md) for DatasourceEvent entity description. See [5c How Does PatchFox work?](../../workflow/5c_how_does_patchfox_work.md) for what a datasourceEvent means to PatchFox. 

### id
Database record id.

### purl
Package identifier in package URL (pURL) format. 

### type
The package type according to the pURL spec.

### name
The package name according to the pURL spec. 

### version
The package version.

### numberVersionsBehindHead
The number of versions of this package that have been released after this one. If (-1) PatchFox has no affirmitive data. 

### numberMajorVersionsBehindHead
If the package is versioned according to semantic versioning, this represents number of major versions of this package that have been released after this one. If (-1) PatchFox has no affirmitive data. 

### numberMinorVersionsBehindHead
If the package is versioned according to semantic versioning, this represents number of minor versions of this package that have been released after this one. If (-1) PatchFox has no affirmitive data. 

### numberPatchVersionsBehindHead
If the package is versioned according to semantic versioning, this represents number of patch versions of this package that have been released after this one. If (-1) PatchFox has no affirmitive data. 

### mostRecentVersion
The most recent version of this package published to the package index. If null PatchFox has no affirmative data. 

### mostRecentVersionPublishedAt
The date and time of when the most recent version of this package published to the package index. If null PatchFox has no affirmative data. 

### updatedAt
POSIX timestamp of when the record was last updated. 


## example API response

```
{
  "code": 200,
  "type": "SUCCESSFUL",
  "description": "OK",
  "txid": "0398cc51-bf87-4334-88e5-e8199fcdfb1b",
  "requestReceivedAt": "2025-07-14T02:17:44.907473924Z",
  "data": {
    "titlePage": {
      "content": [
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
                "id": 10,
                "identifier": "CVE-2022-29526",
                "severity": "MEDIUM",
                "description": "golang.org/x/sys/unix has Incorrect privilege reporting in syscall",
                "cpes": [
                  "cpe:2.3:a:golang:x/sys:v0.0.0-20190412213103-97732733099d:*:*:*:*:*:*:*"
                ],
                "reportedAt": 1748829390.289238000,
                "publishedAt": null,
                "patchedIn": [
                  "0.0.0-20220412211240-33da011f77ad"
                ]
              },
              "id": 10,
              "identifier": "CVE-2022-29526"
            }
          ],
          "criticalFindings": [],
          "highFindings": [],
          "mediumFindings": [],
          "lowFindings": [],
          "datasourceEvents": [],
          "id": 9,
          "purl": "pkg:golang/golang.org/x/sys@v0.0.0-20200124204421-9fbb57f87de9",
          "type": "golang",
          "namespace": "golang.org/x",
          "name": "sys",
          "version": "v0.0.0-20200124204421-9fbb57f87de9",
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

