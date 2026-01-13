# findingData Entity Reference

## field description 

### finding 
[finding](./finding.md) object to which this findingData object is associated. Nested packages and findings collections are always empty to prevent nested reference issues. 

### id
Database record id

### identifier
Unique identifier for this finding (CVE-XXXX, GHSA-XXXX, etc).

### severity
Severity of the finding (CRITICAL, HIGH, MEDIUM, LOW, etc).

### description
Description text describing the finding.

### cpes
Collection of Common Platform Enumeration identifiers.

### reportedAt
POSIX timestamp describing when the finding was reported to PatchFox.

### publishedAt
POSTIX timestamp describing when the finding was first published to the finding index. 

### patchedIn
Collection of package versions where the finding is patched. 


## example API response 

```
{
  "code": 200,
  "type": "SUCCESSFUL",
  "description": "OK",
  "txid": "cea75715-05cf-4342-abae-9bc1e5b85af7",
  "requestReceivedAt": "2025-07-14T20:43:01.399993682Z",
  "data": {
    "titlePage": {
      "content": [
        {
          "finding": {
            "packages": [],
            "reporters": [
              {
                "findings": [],
                "id": 1,
                "name": "GRYPE"
              }
            ],
            "data": null,
            "id": 1,
            "identifier": "CVE-2025-22872"
          },
          "id": 1,
          "identifier": "CVE-2025-22872",
          "severity": "MEDIUM",
          "description": "golang.org/x/net vulnerable to Cross-site Scripting",
          "cpes": [
            "cpe:2.3:a:golang:networking:v0.23.0:*:*:*:*:go:*:*"
          ],
          "reportedAt": 1748829389.332207000,
          "publishedAt": null,
          "patchedIn": [
            "0.38.0"
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