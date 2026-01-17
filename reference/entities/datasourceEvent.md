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

### packageWrapper
PatchFox internal representation of the package tree and (in the packageData subfield) any metadata documents associated with this datasourceEvent. These typically are the SBOM, Grype OSS report, and git commit metadata, including who made the commit as well as the preceding commit history. 


## example API response 

```
{
  "code": 200,
  "type": "SUCCESSFUL",
  "description": "OK",
  "txid": "bc6ed2a2-67b6-4fad-a131-388a70ac01a9",
  "requestReceivedAt": "2026-01-15T07:44:56.459208241Z",
  "data": {
    "titlePage": {
      "content": [
        {
          "id": 17744,
          "purl": "pkg:generic/ibm/openkommander__frontend%3A%3Amain@npm?commitdatetime=2025-11-07T07%3A30%3A08%2B00%3A00&commithash=9e40900a62cd83ab75c341b669147aa533354fbd",
          "txid": "e8d20b31-c663-43b4-a00a-f00431c12079",
          "jobId": "66494628-cf97-4d1e-b158-0d72a4576d5e",
          "commitHash": "9e40900a62cd83ab75c341b669147aa533354fbd",
          "commitBranch": "main",
          "commitDateTime": "2025-11-07T07:30:08Z",
          "eventDateTime": "2025-11-11T00:22:50Z",
          "status": "READY_FOR_NEXT_PROCESSING",
          "processingError": null,
          "ossEnriched": true,
          "packageIndexEnriched": true,
          "analyzed": true,
          "forecasted": false,
          "recommended": false,
          "datasourceId": 1411,
          "packageWrapper": {
            "purl": {
              "scheme": "pkg",
              "type": "generic",
              "namespace": "ibm",
              "name": "openkommander__frontend::main",
              "version": "npm",
              "qualifiers": {
                "commitdatetime": "2025-11-07T07:30:08+00:00",
                "commithash": "9e40900a62cd83ab75c341b669147aa533354fbd"
              },
              "subpath": null,
              "coordinates": "pkg:generic/ibm/openkommander__frontend%3A%3Amain@npm"
            },
            "dependencyType": "UNRECOGNIZED",
            "projectName": "UNKNOWN",
            "packageData": {
              "SBOM": [
                {
                  "@class": "io.patchfox.package_utils.data.sbom.syft.SyftSbomPackageData",
                  "data": {
                    "artifacts": [
                      {
                        "id": "fe7feae37656f953",
                        "name": "@babel/runtime",
                        "version": "7.28.4",
                        "type": "npm",
                        "foundBy": "javascript-lock-cataloger",
                        "locations": [
                          {
                            "path": "/package-lock.json",
                            "accessPath": "/package-lock.json",
                            "annotations": {
                              "evidence": "primary"
                            }
                          }
                        ],
                        "licenses": [
                          {
                            "value": "MIT",
                            "spdxExpression": "MIT",
                            "type": "declared",
                            "urls": [],
                            "locations": [
                              {
                                "path": "/package-lock.json",
                                "accessPath": "/package-lock.json",
                                "annotations": {
                                  "evidence": "primary"
                                }
                              }
                            ]
                          }
                        ],
                        "language": "javascript",
                        "cpes": [
                          {
                            "cpe": "cpe:2.3:a:\\@babel\\/runtime:\\@babel\\/runtime:7.28.4:*:*:*:*:*:*:*",
                            "source": "syft-generated"
                          }
                        ],
                        "purl": "pkg:npm/%40babel/runtime@7.28.4",
                        "metadataType": "javascript-npm-package-lock-entry",
                        "metadata": {
                          "resolved": "https://registry.npmjs.org/@babel/runtime/-/runtime-7.28.4.tgz",
                          "integrity": "sha512-Q/N6JNWvIvPnLDvjlE1OUBLPQHH6l3CltCEsHIujp45zQUSSh8K+gHnaEX45yAT1nyngnINhvWtzN+Nb9D8RAQ=="
                        }
                      },
                      {
                        "id": "83a4289fd94a1cc1",
                        "name": "@carbon/colors",
                        "version": "11.42.0",
                        "type": "npm",
                        "foundBy": "javascript-lock-cataloger",
                        "locations": [
                          {
                            "path": "/package-lock.json",
                            "accessPath": "/package-lock.json",
                            "annotations": {
                              "evidence": "primary"
                            }
                          }
                        ],
                        "licenses": [
                          {
                            "value": "Apache-2.0",
                            "spdxExpression": "Apache-2.0",
                            "type": "declared",
                            "urls": [],
                            "locations": [
                              {
                                "path": "/package-lock.json",
                                "accessPath": "/package-lock.json",
                                "annotations": {
                                  "evidence": "primary"
                                }
                              }
                            ]
                          }
                        ],
                        "language": "javascript",
                        "cpes": [
                          {
                            "cpe": "cpe:2.3:a:\\@carbon\\/colors:\\@carbon\\/colors:11.42.0:*:*:*:*:*:*:*",
                            "source": "syft-generated"
                          }
                        ],
                        "purl": "pkg:npm/%40carbon/colors@11.42.0",
                        "metadataType": "javascript-npm-package-lock-entry",
                        "metadata": {
                          "resolved": "https://registry.npmjs.org/@carbon/colors/-/colors-11.42.0.tgz",
                          "integrity": "sha512-6F1LkQO1FwPwFP+PJkAh9d6c1x3iFJJqVGj3L6n8bEkO+ifeJBRB5Bqnn3BVlCNq7XQTZiYJTR7D3x9CRCY5og=="
                        }
                      }
                      // ... (truncated ~200+ additional artifacts for brevity) ...
                    ],
                    "artifactRelationships": [
                      {
                        "parent": "006934455461b3fa",
                        "child": "fd71c2238fc07657",
                        "type": "evident-by",
                        "metadata": {
                          "kind": "primary"
                        }
                      },
                      {
                        "parent": "028bff0971468aeb",
                        "child": "fd71c2238fc07657",
                        "type": "evident-by",
                        "metadata": {
                          "kind": "primary"
                        }
                      }
                      // ... (truncated ~1000+ additional relationships for brevity) ...
                    ],
                    "files": [],
                    "source": {
                      "id": "fd71c2238fc07657",
                      "name": ".",
                      "version": "",
                      "type": "directory",
                      "foundBy": "directory-cataloger",
                      "locations": [
                        {
                          "path": "/",
                          "accessPath": "/",
                          "annotations": {
                            "evidence": "primary"
                          }
                        }
                      ],
                      "licenses": [],
                      "language": "",
                      "cpes": [],
                      "purl": "",
                      "metadataType": "",
                      "metadata": null
                    },
                    "distro": null,
                    "descriptor": {
                      "name": "syft",
                      "version": "1.20.0",
                      "configuration": null
                    },
                    "schema": {
                      "version": "20.1.7",
                      "url": "https://raw.githubusercontent.com/anchore/syft/main/schema/json/schema-20.1.7.json"
                    }
                  }
                }
              ]
            },
            "children": [
              {
                "purl": {
                  "scheme": "pkg",
                  "type": "npm",
                  "namespace": "null",
                  "name": "@babel/runtime",
                  "version": "7.28.4",
                  "qualifiers": null,
                  "subpath": null,
                  "coordinates": "pkg:npm/null/@babel/runtime@7.28.4"
                },
                "dependencyType": "NPM",
                "projectName": "UNKNOWN",
                "packageData": {},
                "children": [],
                "isRoot": false
              },
              {
                "purl": {
                  "scheme": "pkg",
                  "type": "npm",
                  "namespace": "null",
                  "name": "@carbon/colors",
                  "version": "11.42.0",
                  "qualifiers": null,
                  "subpath": null,
                  "coordinates": "pkg:npm/null/@carbon/colors@11.42.0"
                },
                "dependencyType": "NPM",
                "projectName": "UNKNOWN",
                "packageData": {},
                "children": [],
                "isRoot": false
              }
              // ... (truncated ~200+ additional package dependencies for brevity) ...
            ],
            "isRoot": true
          }
        }
      ],
      "pageable": {
        "pageNumber": 0,
        "pageSize": 1,
        "sort": {
          "sorted": true,
          "empty": false,
          "unsorted": false
        },
        "offset": 0,
        "unpaged": false,
        "paged": true
      },
      "totalElements": 39716,
      "totalPages": 39716,
      "last": false,
      "size": 1,
      "number": 0,
      "sort": {
        "sorted": true,
        "empty": false,
        "unsorted": false
      },
      "numberOfElements": 1,
      "first": true,
      "empty": false
    }
  }
}
```

