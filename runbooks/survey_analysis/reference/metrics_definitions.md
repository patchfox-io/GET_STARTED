# Key Metrics & Definitions

This document defines all metrics used in the PatchFox Dataset Security Analysis.

---

## PES (Patch Efficacy Score)

**Formula:** `PES = Patch Impact / Patch Effort`

**Impact:** Benefit from reducing:
- Findings (vulnerabilities)
- RPS (version diversity)
- Stale packages
- Downlevel packages

**Effort:** Default 1 per patch, decreases with repetition (learning effect)

**Interpretation:**
- **Positive PES:** Patches improving security (good)
- **Zero PES:** Patches have no net benefit (neutral)
- **Negative PES:** Patches making things worse (bad)

**Typical Values:**
| Range | Assessment |
|-------|------------|
| +5 to +10 | Excellent |
| +1 to +5 | Good |
| -1 to +1 | Acceptable |
| -1 to -5 | Poor |
| < -5 | Failing |

---

## RPS (Redundant Package Score)

**Definition:** Percentage of packages that exist in multiple versions across the dataset.

**Range:** 0 to 100

**Example:**
- 100 instances of jackson-databind, all version 2.15.0 → RPS = 0 (perfect)
- 100 instances across 10 different versions → RPS = 10 (fragmented)

**Interpretation:**
| Range | Assessment |
|-------|------------|
| 0-5 | Excellent version consolidation |
| 5-15 | Some fragmentation |
| 15-25 | Significant fragmentation |
| >25 | Severe fragmentation |

**Goal:** Decrease over time (consolidating versions)

---

## Findings

**Types:**
| Field | Description | CVSS Range |
|-------|-------------|------------|
| `totalFindings` | All CVEs detected | - |
| `criticalFindings` | Critical severity | 9.0-10.0 |
| `highFindings` | High severity | 7.0-8.9 |
| `mediumFindings` | Medium severity | 4.0-6.9 |
| `lowFindings` | Low severity | 0.1-3.9 |

**Related:**
- `packagesWithFindings`: Number of packages with at least one CVE
- Vulnerable %: `packagesWithFindings / packages * 100`

**Goal:** Decrease over time

> **Known Bug:** These fields may be undercounted due to a granularity mismatch ([GitHub Issue #9](https://github.com/patchfox-io/GET_STARTED/issues/9)). They count unique (package, finding) pairs, while backlog metrics count per-datasource occurrences. If backlog totals exceed these values, use the backlog totals as the true count. See [Common Pitfalls #13](./common_pitfalls.md#13-backlog-exceeds-total-findings-granularity-mismatch).

---

## Backlog

Findings that have been known for a period of time but not yet remediated.

**Categories:**
| Field | Age |
|-------|-----|
| `findingsInBacklogBetweenThirtyAndSixtyDays` | 30-60 days |
| `findingsInBacklogBetweenSixtyAndNinetyDays` | 60-90 days |
| `findingsInBacklogOverNinetyDays` | 90+ days |

**Interpretation (% in 90+ day bucket):**
| Percentage | Assessment |
|------------|------------|
| <30% | Good |
| 30-50% | Acceptable |
| 50-70% | Poor |
| >70% | Failing |

**Goal:** Decrease backlog, especially 90+ day bucket

---

## Downlevel & Stale

### Downlevel
Package not at most current vendor version.

| Field | Description |
|-------|-------------|
| `downlevelPackagesMajor` | Major version behind |
| `downlevelPackagesMinor` | Minor version behind |
| `downlevelPackagesPatch` | Patch version behind |

### Stale
Package not updated by vendor in X time.

| Field | Description |
|-------|-------------|
| `stalePackagesTwoYears` | Not updated in >2 years |

**Goal:** Decrease both counts and percentages

---

## Edit Types

### From datasetMetrics
| Field | Description |
|-------|-------------|
| `patches` | Total patches in snapshot |
| `samePatches` | "Same version" edits (re-evaluation, NO version change) |
| `differentPatches` | "Different version" edits (actual version changes) |

### From edit records
| Type | Description |
|------|-------------|
| `CREATE` | New package added |
| `UPDATE` | Package version changed (or re-evaluated) |
| `DELETE` | Package removed |

### Key Distinction
- `sameEdit: true` in edit record = "same version" (no actual change)
- `sameEdit: false` = actual version change

### Impact
| Edit Type | Security Impact |
|-----------|-----------------|
| Same version edits | ZERO security benefit (can't fix vulnerabilities) |
| Different version edits | POTENTIAL security benefit (if targeting CVEs) |
| CREATE edits | Potentially adds vulnerabilities (growth) |
| DELETE edits | Removes potential vulnerabilities (reduction) |

---

## Data Sources

### Primary Endpoints

| Endpoint | Use For |
|----------|---------|
| `/api/v1/db/datasetMetrics/query` | Temporal trend analysis, PES calculation |
| `/api/v1/db/datasetMetrics/edit/query` | Understanding patch behavior, edit type breakdown |
| `/api/v1/db/findingData/query` | CVE age analysis, vulnerability severity breakdown |
| `/api/v1/db/datasourceMetricsCurrent/query` | Dataset composition analysis |
