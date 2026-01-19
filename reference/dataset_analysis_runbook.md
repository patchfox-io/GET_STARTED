# PatchFox Dataset Security Analysis Runbook

## Purpose

This runbook provides a systematic methodology for analyzing the security posture and patching effectiveness of any PatchFox dataset. Follow this step-by-step guide to produce a comprehensive assessment.

---

## Table of Contents

1. [Overview & Objectives](#1-overview--objectives)
2. [Prerequisites](#2-prerequisites)
3. [Data Sources](#3-data-sources)
4. [Key Metrics & Definitions](#4-key-metrics--definitions)
5. [Analysis Methodology](#5-analysis-methodology)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interpretation Guidelines](#7-interpretation-guidelines)
8. [Report Structure Template](#8-report-structure-template)
9. [Example Analysis Script](#9-example-analysis-script)
10. [Checklist](#10-checklist)
11. [Appendix: Quick Reference](#appendix-quick-reference)

---

## 1. Overview & Objectives

### What This Analysis Reveals

A comprehensive dataset analysis answers:
- **Are patches effective?** (PES - Patch Efficacy Score)
- **Are vulnerabilities decreasing?** (Findings trend)
- **Are patches addressing vulnerabilities?** (Backlog analysis)
- **What types of patches are being made?** (Edit type breakdown)
- **How old are unaddressed vulnerabilities?** (CVE age analysis)
- **Is the attack surface growing or shrinking?** (Package count, RPS trend)

### Output

A comprehensive report with:
- Overall security grade (A-F)
- Patch efficacy assessment
- Findings and backlog trends
- Root cause analysis
- Prioritized recommendations

---

## 2. Prerequisites

### Required Access

- Data Service API: `http://localhost:1702`
- Orchestrate Service: `http://localhost:1707` (for peristalsis control)

### Required Knowledge

- Understanding of [Custom Metrics](./custom_metrics.md) (RPS, PES)
- Understanding of [Data Service API](./data_service_api.md)
- Understanding of [Entity Models](./entities/entities.md)

### Tools Needed

- `curl` for API queries
- `python3` for data analysis
- `jq` or Python for JSON processing

---

## 3. Data Sources

### 3.1 Primary Tables

#### datasetMetrics
**Endpoint:** `/api/v1/db/datasetMetrics/query?datasetName={DATASET}&isCurrent=true`

**Contains:**
- PES (patchEfficacyScore), RPS (rpsScore)
- Findings counts by severity
- Backlog aging (30-60d, 60-90d, 90+d)
- Package counts (total, downlevel, stale)
- Patch counts (patches, samePatches, differentPatches)

**Use For:** Temporal trend analysis, PES calculation

#### datasetMetrics/edit
**Endpoint:** `/api/v1/db/datasetMetrics/edit/query?datasetName={DATASET}&isCurrent=true&size=25000`

**Contains:**
- All edits (CREATE, UPDATE, DELETE)
- Before/after package versions
- Edit metadata (sameEdit, userEdit, pfRecommendedEdit)

**Use For:** Understanding patch behavior, edit type breakdown

#### findingData
**Endpoint:** `/api/v1/db/findingData/query?size=500`

**Contains:**
- CVE identifiers
- Severity levels
- NVD published dates (publishedAt)
- Dataset detection dates (reportedAt)
- Patch availability (patchedIn)

**Use For:** CVE age analysis, vulnerability severity breakdown

### 3.2 Query Examples

**Get dataset metrics (current state only):**
```bash
curl -s "http://localhost:1702/api/v1/db/datasetMetrics/query?datasetName=DATASET_NAME&isCurrent=true"
```

**Get all edits (last 6 months with date filter):**
```bash
curl -s "http://localhost:1702/api/v1/db/datasetMetrics/edit/query?datasetName=DATASET_NAME&isCurrent=true&commitDateTime=gte.2025-07-01T00:00:00Z&size=25000"
```

**Get finding data:**
```bash
curl -s "http://localhost:1702/api/v1/db/findingData/query?size=500"
```

**Using `select` parameter (AI-only feature):**
```bash
curl -s "http://localhost:1702/api/v1/db/datasetMetrics/query?datasetName=DATASET_NAME&isCurrent=true&select=commitDateTime,patchEfficacyScore,rpsScore,totalFindings,packages"
```

---

## 4. Key Metrics & Definitions

### 4.1 PES (Patch Efficacy Score)

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
- Excellent: +5 to +10
- Good: +1 to +5
- Acceptable: -1 to +1
- Poor: -1 to -5
- Failing: < -5

### 4.2 RPS (Redundant Package Score)

**Definition:** Percentage of packages that exist in multiple versions across the dataset.

**Range:** 0 to 100

**Example:**
- 100 instances of jackson-databind, all version 2.15.0 → RPS = 0 (perfect)
- 100 instances across 10 different versions → RPS = 10 (fragmented)

**Interpretation:**
- **Low RPS (0-5):** Excellent version consolidation
- **Medium RPS (5-15):** Some fragmentation
- **High RPS (15-25):** Significant fragmentation
- **Very High RPS (>25):** Severe fragmentation

**Goal:** Decrease over time (consolidating versions)

### 4.3 Findings

**Types:**
- `totalFindings`: All CVEs detected
- `criticalFindings`: CVSS 9.0-10.0
- `highFindings`: CVSS 7.0-8.9
- `mediumFindings`: CVSS 4.0-6.9
- `lowFindings`: CVSS 0.1-3.9

**Related:**
- `packagesWithFindings`: Number of packages with at least one CVE
- Vulnerable %: `packagesWithFindings / packages * 100`

**Goal:** Decrease over time

### 4.4 Backlog

**Categories:**
- `findingsInBacklogBetweenThirtyAndSixtyDays`: Known 30-60 days
- `findingsInBacklogBetweenSixtyAndNinetyDays`: Known 60-90 days
- `findingsInBacklogOverNinetyDays`: Known 90+ days

**Interpretation:**
- **Good:** <30% in 90+ day bucket
- **Acceptable:** 30-50% in 90+ day bucket
- **Poor:** 50-70% in 90+ day bucket
- **Failing:** >70% in 90+ day bucket

**Goal:** Decrease backlog, especially 90+ day bucket

### 4.5 Downlevel & Stale

**Downlevel:** Package not at most current vendor version
- `downlevelPackagesMajor`: Major version behind
- `downlevelPackagesMinor`: Minor version behind
- `downlevelPackagesPatch`: Patch version behind

**Stale:** Package not updated by vendor in X time
- `stalePackagesTwoYears`: Not updated in >2 years

**Goal:** Decrease both counts and percentages

### 4.6 Edit Types

**From datasetMetrics:**
- `patches`: Total patches in snapshot
- `samePatches`: "Same version" edits (re-evaluation, NO version change)
- `differentPatches`: "Different version" edits (actual version changes)

**From edit records:**
- `CREATE`: New package added
- `UPDATE`: Package version changed (or re-evaluated)
- `DELETE`: Package removed

**Key Distinction:**
- `sameEdit: true` in edit record = "same version" (no actual change)
- `sameEdit: false` = actual version change

**Impact:**
- **Same version edits:** ZERO security benefit (can't fix vulnerabilities)
- **Different version edits:** POTENTIAL security benefit (if targeting CVEs)
- **CREATE edits:** Potentially adds vulnerabilities (growth)
- **DELETE edits:** Removes potential vulnerabilities (reduction)

---

## 5. Analysis Methodology

### STEP 1: Initial Data Collection

**1.1 Get Dataset Name**
```bash
curl -s "http://localhost:1702/api/v1/db/dataset/query" | python3 -m json.tool | head -30
```

Extract `name` field from first dataset.

**1.2 Fetch Dataset Metrics (Current)**
```bash
curl -s "http://localhost:1702/api/v1/db/datasetMetrics/query?datasetName={DATASET}&isCurrent=true" -o /tmp/metrics.json
```

**1.3 Fetch All Edits (6 months)**

Calculate 6 months ago date:
```python
from datetime import datetime, timedelta
six_months = (datetime.now() - timedelta(days=180)).strftime('%Y-%m-%dT%H:%M:%SZ')
```

Fetch edits:
```bash
curl -s "http://localhost:1702/api/v1/db/datasetMetrics/edit/query?datasetName={DATASET}&isCurrent=true&commitDateTime=gte.{DATE}&size=25000" -o /tmp/edits.json
```

**1.4 Fetch Finding Data**
```bash
curl -s "http://localhost:1702/api/v1/db/findingData/query?size=500" -o /tmp/findings.json
```

### STEP 2: Dataset Composition Analysis

**2.1 Fetch Datasource Metrics**
```bash
curl -s "http://localhost:1702/api/v1/db/datasourceMetricsCurrent/query?size=1000" -o /tmp/dmc.json
```

**2.2 Analyze Composition by Type**

```python
import json
from collections import defaultdict

with open('/tmp/dmc.json') as f:
    data = json.load(f)

records = data['data']['titlePage']['content']
print(f"Total datasources: {len(records)}\n")

# Extract type from purl (last segment after @)
by_type = defaultdict(lambda: {'count': 0, 'packages': 0, 'findings': 0, 'critical': 0, 'high': 0})

for r in records:
    purl = r.get('purl', '')
    ds_type = purl.split('@')[-1] if '@' in purl else 'unknown'

    by_type[ds_type]['count'] += 1
    by_type[ds_type]['packages'] += r.get('packages', 0)
    by_type[ds_type]['findings'] += r.get('totalFindings', 0)
    by_type[ds_type]['critical'] += r.get('criticalFindings', 0)
    by_type[ds_type]['high'] += r.get('highFindings', 0)

# Calculate totals
total_ds = sum(v['count'] for v in by_type.values())
total_pkg = sum(v['packages'] for v in by_type.values())
total_findings = sum(v['findings'] for v in by_type.values())

print(f"{'Type':<12} {'DS Count':>10} {'DS %':>8} {'Packages':>10} {'Pkg %':>8} {'Findings':>10} {'Find %':>8}")
print("-" * 80)

for ds_type in sorted(by_type.keys(), key=lambda x: by_type[x]['findings'], reverse=True):
    v = by_type[ds_type]
    ds_pct = v['count'] / total_ds * 100 if total_ds else 0
    pkg_pct = v['packages'] / total_pkg * 100 if total_pkg else 0
    find_pct = v['findings'] / total_findings * 100 if total_findings else 0
    print(f"{ds_type:<12} {v['count']:>10,} {ds_pct:>7.1f}% {v['packages']:>10,} {pkg_pct:>7.1f}% {v['findings']:>10,} {find_pct:>7.1f}%")
```

**2.3 Interpret Composition**

Look for disproportionate relationships:

| Pattern | Meaning |
|---------|---------|
| Type has X% datasources but Y% packages (Y >> X) | Heavy dependency footprint per project (e.g., npm) |
| Type has X% packages but Y% findings (Y >> X) | Higher vulnerability density (riskier ecosystem) |
| Type has X% findings but Y% critical+high (Y >> X) | More severe vulnerabilities in that ecosystem |

**Red Flags:**
- One type dominates findings despite small datasource count → concentrated risk
- High finding % with low package % → vulnerable ecosystem, prioritize remediation
- Ecosystem with disproportionate critical+high → urgent attention needed

**Example Insight:** "npm is only 1.6% of datasources but 17% of packages - each npm project brings 10x more dependencies. pypi has 6x the vulnerability density of the dataset average."

### STEP 3: PES Analysis

**3.1 Calculate PES Statistics**

From `/tmp/metrics.json`:
```python
import json
with open('/tmp/metrics.json') as f:
    data = json.load(f)
metrics = data['data']['titlePage']['content']

# Sort chronologically
metrics.sort(key=lambda x: x['commitDateTime'])

# Calculate averages
avg_pes = sum(m['patchEfficacyScore'] for m in metrics) / len(metrics)
avg_impact = sum(m['patchImpact'] for m in metrics) / len(metrics)
avg_effort = sum(m['patchEffort'] for m in metrics) / len(metrics)

# Count distribution
positive = sum(1 for m in metrics if m['patchEfficacyScore'] > 0)
negative = sum(1 for m in metrics if m['patchEfficacyScore'] < 0)
neutral = sum(1 for m in metrics if m['patchEfficacyScore'] == 0)

print(f"Average PES: {avg_pes:.2f}")
print(f"Average Impact: {avg_impact:.2f}")
print(f"Average Effort: {avg_effort:.2f}")
print(f"Positive: {positive}/{len(metrics)} ({positive/len(metrics)*100:.1f}%)")
print(f"Negative: {negative}/{len(metrics)} ({negative/len(metrics)*100:.1f}%)")
```

**3.2 Identify PES Trends**

Track over time:
```python
for m in metrics:
    print(f"{m['commitDateTime'][:10]}: PES={m['patchEfficacyScore']:.2f}, Impact={m['patchImpact']:.2f}, Effort={m['patchEffort']:.2f}")
```

**3.3 Grade PES Performance**
- Average PES > 1: A (Excellent)
- Average PES 0 to 1: B (Good)
- Average PES -1 to 0: C (Acceptable)
- Average PES -2 to -1: D (Poor)
- Average PES < -2: F (Failing)

### STEP 4: RPS Analysis

**4.1 Calculate RPS Trend**

```python
rps_values = [(m['commitDateTime'][:10], m['rpsScore']) for m in metrics]
rps_sorted = sorted(rps_values, key=lambda x: x[0])

print(f"RPS Earliest: {rps_sorted[0]}")
print(f"RPS Latest: {rps_sorted[-1]}")
print(f"RPS Change: {rps_sorted[-1][1] - rps_sorted[0][1]:.2f}")
```

**4.2 Interpret RPS**
- Decreasing RPS: ✓ Good (consolidating versions)
- Stable RPS: ○ Neutral
- Increasing RPS: ✗ Bad (fragmenting versions)

### STEP 5: Findings Analysis

**5.1 Calculate Findings Trend**

```python
findings_data = [(m['commitDateTime'][:10], m['totalFindings'], m['packages']) for m in metrics]
findings_sorted = sorted(findings_data, key=lambda x: x[0])

print("Findings Trajectory:")
for date, findings, packages in findings_sorted:
    vuln_pct = (findings / packages * 100) if packages > 0 else 0
    print(f"  {date}: {findings} findings in {packages} packages ({vuln_pct:.2f}% vulnerable)")

# Calculate change
change = findings_sorted[-1][1] - findings_sorted[0][1]
print(f"\nFindings Change: {change:+d} ({(change/findings_sorted[0][1]*100):.1f}%)")
```

**5.2 Severity Breakdown**

```python
latest = metrics[-1]
print(f"Current Findings (Severity):")
print(f"  CRITICAL: {latest['criticalFindings']}")
print(f"  HIGH: {latest['highFindings']}")
print(f"  MEDIUM: {latest['mediumFindings']}")
print(f"  LOW: {latest['lowFindings']}")
print(f"  TOTAL: {latest['totalFindings']}")
```

**5.3 Interpret Findings Trend**
- Decreasing findings: ✓ Good (effective remediation)
- Stable findings: ○ Neutral (treading water)
- Increasing findings: ✗ Bad (accumulating debt)

### STEP 6: Backlog Analysis

**6.1 Calculate Backlog Distribution**

```python
for m in metrics[-5:]:  # Last 5 snapshots
    backlog_30_60 = m.get('findingsInBacklogBetweenThirtyAndSixtyDays', 0)
    backlog_60_90 = m.get('findingsInBacklogBetweenSixtyAndNinetyDays', 0)
    backlog_90_plus = m.get('findingsInBacklogOverNinetyDays', 0)
    total_backlog = backlog_30_60 + backlog_60_90 + backlog_90_plus

    if total_backlog > 0:
        pct_90_plus = (backlog_90_plus / total_backlog * 100)
        print(f"{m['commitDateTime'][:10]}: {total_backlog:.0f} backlog ({pct_90_plus:.0f}% in 90+ days)")
```

**6.2 Interpret Backlog**
- Growing backlog: ✗ Bad (not resolving)
- Stable backlog: ○ Neutral (treading water)
- Shrinking backlog: ✓ Good (actively resolving)
- >70% in 90+ days: ✗ Very bad (old vulnerabilities)

### STEP 7: Edit Type Analysis

**7.1 Count Edit Types**

From `/tmp/edits.json`:
```python
with open('/tmp/edits.json') as f:
    data = json.load(f)
edits = data['data']['titlePage']['content']

from collections import Counter

edit_types = Counter(e['editType'] for e in edits)
same_edits = sum(1 for e in edits if e['sameEdit'])
different_edits = len(edits) - same_edits

print(f"Total Edits: {len(edits)}")
print(f"\nEdit Type Breakdown:")
for et, count in edit_types.items():
    pct = (count / len(edits)) * 100
    print(f"  {et}: {count:,} ({pct:.1f}%)")

print(f"\nSame vs Different:")
print(f"  Same version: {same_edits:,} ({same_edits/len(edits)*100:.1f}%)")
print(f"  Different version: {different_edits:,} ({different_edits/len(edits)*100:.1f}%)")
```

**7.2 Interpret Edit Types**

**Red Flags:**
- >50% same version edits: Wasting effort
- CREATE > DELETE: Growing attack surface uncontrolled
- High activity + negative PES: Busy work without benefit

**Good Patterns:**
- <30% same version edits
- CREATE ≈ DELETE (controlled growth)
- High different version % + positive PES

### STEP 8: Package Family Remediation Analysis

**8.1 Calculate Aggregate Remediation Potential**

First, get the headline number - how many CVE instances could be eliminated by standardizing on package versions you already use elsewhere.

Run directly against the database:
```sql
-- Aggregate: How many findings could be eliminated via version consolidation?
WITH the_latest_metric AS (
    SELECT package_indexes
    FROM public.dataset_metrics
    WHERE is_current = true
    ORDER BY commit_date_time DESC
    LIMIT 1
),
latest_packages AS (
    SELECT p.id, p.name, p.namespace, p.version,
           COALESCE(p.namespace, '') || '/' || p.name AS family_key
    FROM public.package p, the_latest_metric m
    WHERE p.id = ANY(m.package_indexes)
),
package_families AS (
    SELECT family_key, COUNT(*) AS family_size
    FROM latest_packages
    GROUP BY family_key
    HAVING COUNT(*) > 1
),
packages_in_multi_member_families AS (
    SELECT lp.*
    FROM latest_packages lp
    JOIN package_families pf ON lp.family_key = pf.family_key
),
findings_with_patched_versions AS (
    SELECT pf.finding_id, pmf.family_key
    FROM public.package_finding pf
    JOIN packages_in_multi_member_families pmf ON pf.package_id = pmf.id
    JOIN package_families fam ON pmf.family_key = fam.family_key
    GROUP BY pf.finding_id, pmf.family_key, fam.family_size
    HAVING COUNT(DISTINCT pf.package_id) < fam.family_size
)
SELECT
    COUNT(pf.finding_id) AS total_finding_instances_in_families,
    COUNT(fpv.finding_id) AS instances_with_patched_versions_in_family,
    ROUND(COUNT(fpv.finding_id)::numeric / NULLIF(COUNT(pf.finding_id), 0) * 100, 1) AS pct_eliminable
FROM public.package_finding pf
JOIN packages_in_multi_member_families pmf ON pf.package_id = pmf.id
LEFT JOIN findings_with_patched_versions fpv
    ON pf.finding_id = fpv.finding_id
    AND pmf.family_key = fpv.family_key;
```

**Interpret Results:**
- `total_finding_instances_in_families`: CVEs in packages with version fragmentation
- `instances_with_patched_versions_in_family`: CVEs eliminable via consolidation
- `pct_eliminable`: Percentage of findings that are "low-hanging fruit"

**Example:** "847 CVE instances exist in fragmented packages. 612 (72%) could be eliminated by standardizing on versions already in use elsewhere."

**8.2 Identify Specific Consolidation Opportunities**

Now get the detailed breakdown - which specific package families offer the best opportunities.

Run directly against the database:
```sql
-- Find package families with remediation opportunities
WITH the_latest_metric AS (
    SELECT package_indexes
    FROM public.dataset_metrics
    WHERE is_current = true
    ORDER BY commit_date_time DESC
    LIMIT 1
),
latest_packages AS (
    SELECT p.id, p.name, p.namespace, p.version,
           COALESCE(p.namespace, '') || '/' || p.name AS family_key
    FROM public.package p
    INNER JOIN the_latest_metric m ON p.id = ANY(m.package_indexes)
),
package_families AS (
    SELECT family_key, COUNT(*) AS family_size
    FROM latest_packages
    GROUP BY family_key
    HAVING COUNT(*) > 1
),
packages_in_multi_member_families AS (
    SELECT lp.*, pf.family_size
    FROM latest_packages lp
    JOIN package_families pf ON lp.family_key = pf.family_key
),
packages_with_any_findings AS (
    SELECT DISTINCT package_id
    FROM public.package_finding
    WHERE package_id IN (SELECT id FROM packages_in_multi_member_families)
)
SELECT
    pmf.family_key,
    COUNT(pf.finding_id) AS total_finding_instances,
    MAX(pmf.family_size) AS total_family_members,
    COUNT(DISTINCT paf.package_id) AS family_members_with_findings,
    (MAX(pmf.family_size) - COUNT(DISTINCT paf.package_id)) AS family_members_without_findings
FROM packages_in_multi_member_families pmf
LEFT JOIN public.package_finding pf ON pmf.id = pf.package_id
LEFT JOIN packages_with_any_findings paf ON pmf.id = paf.package_id
GROUP BY pmf.family_key
HAVING COUNT(DISTINCT paf.package_id) > 0  -- Only families with at least one vulnerable version
   AND (MAX(pmf.family_size) - COUNT(DISTINCT paf.package_id)) > 0  -- And at least one clean version
ORDER BY total_finding_instances DESC
LIMIT 20;
```

**8.3 Interpret Results**

| Column | Meaning |
|--------|---------|
| `family_key` | Package family (namespace/name) |
| `total_finding_instances` | Total CVEs across all versions |
| `total_family_members` | Number of versions in dataset |
| `family_members_with_findings` | Versions that have vulnerabilities |
| `family_members_without_findings` | Versions that are clean |

**High-Value Targets:**
- Families with high `total_finding_instances` AND `family_members_without_findings > 0`
- These can be consolidated to clean versions, eliminating CVEs AND reducing RPS

**Example Output:**
```
family_key                                    | findings | members | vuln | clean
----------------------------------------------+----------+---------+------+------
com.fasterxml.jackson.core/jackson-databind   |       47 |       5 |    3 |     2
org.apache.commons/commons-compress           |       12 |       4 |    2 |     2
io.netty/netty-codec-http                     |        8 |       3 |    2 |     1
```

**Interpretation:** jackson-databind has 5 versions in the dataset, 3 with vulnerabilities (47 total findings), and 2 clean versions. Consolidating to one of the clean versions eliminates 47 CVEs and reduces RPS by 4.

**8.4 Grade Remediation Opportunity**

- `pct_eliminable` > 50%: Excellent - majority of fragmented-package CVEs are low-hanging fruit
- `pct_eliminable` 25-50%: Good - significant consolidation opportunity
- `pct_eliminable` < 25%: Limited - most CVEs require actual upgrades, not just consolidation

**Detailed breakdown grading:**
- Many high-value targets (>5 families with 10+ findings): Excellent opportunity, prioritize consolidation
- Some targets (2-5 families): Good opportunity, include in remediation plan
- Few targets (<2 families): Limited quick wins, focus on other strategies

**8.5 Generate Actionable Patch Recommendations**

This step transforms the analysis into specific, actionable patch recommendations suitable for JIRA tickets or automated PRs.

**8.5.1 Identify Emergency CVEs**

First, check for "drop everything" vulnerabilities - CRITICAL severity CVEs that need immediate attention.

```bash
curl -s "http://localhost:1702/api/v1/db/findingData/query?severity=CRITICAL&size=500" -o /tmp/critical_cves.json
```

```python
import json

with open('/tmp/critical_cves.json') as f:
    data = json.load(f)

critical = data['data']['titlePage']['content']

# Known catastrophic CVEs (log4shell, etc.)
EMERGENCY_CVES = {
    'CVE-2021-44228': 'Log4Shell - Remote Code Execution',
    'CVE-2021-45046': 'Log4Shell followup - RCE',
    'CVE-2021-45105': 'Log4Shell followup - DoS',
    'CVE-2022-22965': 'Spring4Shell - RCE',
    'CVE-2021-26855': 'ProxyLogon - RCE',
    'CVE-2023-44487': 'HTTP/2 Rapid Reset - DoS',
}

print("=== EMERGENCY CVE CHECK ===\n")
emergencies = []
for cve in critical:
    identifier = cve.get('identifier', '')
    if identifier in EMERGENCY_CVES:
        emergencies.append({
            'cve': identifier,
            'description': EMERGENCY_CVES[identifier],
            'detail': cve.get('description', '')
        })

if emergencies:
    print("🚨 CRITICAL: EMERGENCY CVEs DETECTED - IMMEDIATE ACTION REQUIRED 🚨\n")
    for e in emergencies:
        print(f"  {e['cve']}: {e['description']}")
        print(f"    {e['detail']}\n")
else:
    print("✓ No known emergency CVEs (log4shell, spring4shell, etc.) detected\n")

print(f"Total CRITICAL severity CVEs: {len(critical)}")
for cve in critical[:10]:
    print(f"  {cve.get('identifier')}: {cve.get('description', '')[:60]}...")
```

**8.5.2 Map Packages to Datasources**

Identify which datasources contain vulnerable packages.

```bash
curl -s "http://localhost:1702/api/v1/db/datasourceMetricsCurrent/query?size=1000" -o /tmp/dmc.json
```

```python
import json
import re

with open('/tmp/dmc.json') as f:
    dmc_data = json.load(f)

datasources = dmc_data['data']['titlePage']['content']

# Build lookup: package type -> datasources
ds_by_type = {}
for ds in datasources:
    purl = ds.get('purl', '')
    ds_type = purl.split('@')[-1] if '@' in purl else 'unknown'
    ds_name = purl.split('/')[2].split('%3A')[0] if '//' not in purl and len(purl.split('/')) > 2 else purl

    if ds_type not in ds_by_type:
        ds_by_type[ds_type] = []
    ds_by_type[ds_type].append({
        'name': ds_name,
        'purl': purl,
        'packages': ds.get('packages', 0),
        'findings': ds.get('totalFindings', 0),
        'critical': ds.get('criticalFindings', 0)
    })

print("Datasources by package type:")
for pkg_type, ds_list in sorted(ds_by_type.items()):
    print(f"  {pkg_type}: {len(ds_list)} datasources")
```

**8.5.3 Generate Patch Tickets**

Combine family analysis with datasource mapping to produce actionable recommendations.

```python
# Assuming you have the family analysis results from 8.2
# For each high-value family, generate a ticket-ready recommendation

def assess_breaking_risk(from_version, to_version, pkg_name=None, pkg_type=None):
    """Assess breaking change risk based on semver AND changelog analysis"""
    import urllib.request
    import json as json_lib

    risk_factors = []
    changelog_notes = []

    # 1. Semver analysis
    try:
        from_parts = re.split(r'[.\-]', from_version)[:3]
        to_parts = re.split(r'[.\-]', to_version)[:3]

        from_major = int(re.sub(r'[^\d]', '', from_parts[0])) if from_parts else 0
        to_major = int(re.sub(r'[^\d]', '', to_parts[0])) if to_parts else 0

        from_minor = int(re.sub(r'[^\d]', '', from_parts[1])) if len(from_parts) > 1 else 0
        to_minor = int(re.sub(r'[^\d]', '', to_parts[1])) if len(to_parts) > 1 else 0

        if to_major > from_major:
            semver_risk = "HIGH"
            risk_factors.append("Major version bump")
        elif to_minor > from_minor:
            semver_risk = "MEDIUM"
            risk_factors.append("Minor version bump")
        else:
            semver_risk = "LOW"
            risk_factors.append("Patch version bump")
    except:
        semver_risk = "UNKNOWN"
        risk_factors.append("Could not parse version numbers")

    # 2. Changelog analysis (where available)
    if pkg_name and pkg_type:
        try:
            changelog_text = ""

            # NPM: Fetch from registry
            if pkg_type == 'npm':
                url = f"https://registry.npmjs.org/{pkg_name}/{to_version}"
                with urllib.request.urlopen(url, timeout=5) as resp:
                    data = json_lib.loads(resp.read())
                    # Check for deprecation warnings
                    if data.get('deprecated'):
                        risk_factors.append("⚠️ VERSION DEPRECATED")
                        changelog_notes.append(f"Deprecation: {data['deprecated']}")

            # PyPI: Fetch release info
            elif pkg_type == 'pypi':
                url = f"https://pypi.org/pypi/{pkg_name}/{to_version}/json"
                with urllib.request.urlopen(url, timeout=5) as resp:
                    data = json_lib.loads(resp.read())
                    # Check classifiers for stability
                    classifiers = data.get('info', {}).get('classifiers', [])
                    for c in classifiers:
                        if 'Development Status' in c:
                            changelog_notes.append(f"Status: {c}")
                        if 'Alpha' in c or 'Beta' in c:
                            risk_factors.append("Pre-release version")

            # Maven: Check for release notes in description
            elif pkg_type == 'maven':
                # Maven Central search API
                group_id, artifact_id = pkg_name.rsplit('/', 1) if '/' in pkg_name else ('', pkg_name)
                group_id = group_id.replace('/', '.')
                url = f"https://search.maven.org/solrsearch/select?q=g:{group_id}+AND+a:{artifact_id}+AND+v:{to_version}&rows=1&wt=json"
                with urllib.request.urlopen(url, timeout=5) as resp:
                    data = json_lib.loads(resp.read())
                    docs = data.get('response', {}).get('docs', [])
                    if docs:
                        # Check timestamp - very old versions may have compatibility issues
                        timestamp = docs[0].get('timestamp', 0)
                        if timestamp > 0:
                            from datetime import datetime
                            release_date = datetime.fromtimestamp(timestamp/1000)
                            age_years = (datetime.now() - release_date).days / 365
                            if age_years > 3:
                                changelog_notes.append(f"Release is {age_years:.1f} years old")

        except Exception as e:
            changelog_notes.append(f"(Changelog fetch failed: {str(e)[:50]})")

    # 3. Keyword analysis in version string
    breaking_keywords = ['alpha', 'beta', 'rc', 'snapshot', 'pre', 'dev']
    if any(kw in to_version.lower() for kw in breaking_keywords):
        risk_factors.append("Pre-release/unstable version indicator")
        if semver_risk == "LOW":
            semver_risk = "MEDIUM"

    # Compile final assessment
    risk_reason = "; ".join(risk_factors)
    if changelog_notes:
        risk_reason += "\n    Changelog: " + "; ".join(changelog_notes)

    return semver_risk, risk_reason


def fetch_changelog_summary(pkg_name, pkg_type, from_version, to_version):
    """Attempt to fetch and summarize changelog between versions"""
    import json
    import urllib.request

    summary = []

    try:
        if pkg_type == 'npm':
            # GitHub releases are often linked in npm
            url = f"https://registry.npmjs.org/{pkg_name}"
            with urllib.request.urlopen(url, timeout=5) as resp:
                data = json.loads(resp.read())
                repo = data.get('repository', {})
                if isinstance(repo, dict) and 'url' in repo:
                    repo_url = repo['url']
                    if 'github.com' in repo_url:
                        summary.append(f"GitHub: {repo_url.replace('git+', '').replace('.git', '')}/releases")

        elif pkg_type == 'pypi':
            url = f"https://pypi.org/pypi/{pkg_name}/json"
            with urllib.request.urlopen(url, timeout=5) as resp:
                data = json.loads(resp.read())
                project_urls = data.get('info', {}).get('project_urls', {})
                for key in ['Changelog', 'Release Notes', 'History', 'Changes']:
                    if key in project_urls:
                        summary.append(f"{key}: {project_urls[key]}")
                        break

    except:
        pass

    return summary or ["No changelog URL found - manual review recommended"]

# Template for each recommendation
TICKET_TEMPLATE = """
================================================================================
PATCH RECOMMENDATION: {family_key}
================================================================================
PRIORITY: {priority}
RISK LEVEL: {risk_level}
RISK REASON: {risk_reason}

ACTION: Update {family_key}
  FROM: {vulnerable_versions}
  TO:   {target_version}

IMPACT:
  - CVEs eliminated: {cve_count}
  - Critical/High CVEs: {critical_high}
  - RPS reduction: {rps_reduction}

AFFECTED DATASOURCES ({ds_type}):
{datasource_list}

CVEs RESOLVED:
{cve_list}

CHANGELOG / RELEASE NOTES:
{changelog_links}

NOTES:
{notes}
================================================================================
"""

# Example output generation (customize based on your 8.2 query results)
def generate_ticket(family_key, vulnerable_versions, target_version, cve_count,
                    critical_high, pkg_type, cve_details):

    # Extract package name from family_key (namespace/name format)
    pkg_name = family_key.split('/')[-1] if '/' in family_key else family_key

    # Assess breaking risk with changelog analysis
    risk_level, risk_reason = assess_breaking_risk(
        vulnerable_versions[0], target_version,
        pkg_name=pkg_name, pkg_type=pkg_type
    )

    # Fetch changelog links for manual review
    changelog_links = fetch_changelog_summary(pkg_name, pkg_type, vulnerable_versions[0], target_version)

    # Set priority based on CVE severity
    if critical_high > 0:
        priority = "P1 - CRITICAL" if 'CRITICAL' in str(cve_details) else "P2 - HIGH"
    else:
        priority = "P3 - MEDIUM"

    # Get affected datasources
    affected_ds = ds_by_type.get(pkg_type, [])
    ds_list = "\n".join([f"  - {ds['name']} ({ds['findings']} findings)"
                         for ds in affected_ds[:10]])
    if len(affected_ds) > 10:
        ds_list += f"\n  ... and {len(affected_ds) - 10} more"

    return TICKET_TEMPLATE.format(
        family_key=family_key,
        priority=priority,
        risk_level=risk_level,
        risk_reason=risk_reason,
        vulnerable_versions=", ".join(vulnerable_versions),
        target_version=target_version,
        cve_count=cve_count,
        critical_high=critical_high,
        rps_reduction=len(vulnerable_versions),
        ds_type=pkg_type,
        datasource_list=ds_list or "  (Unable to determine specific datasources)",
        cve_list=cve_details[:500] + "..." if len(cve_details) > 500 else cve_details,
        changelog_links="\n".join(f"  - {link}" for link in changelog_links),
        notes="- Review changelog for BREAKING CHANGES before upgrading\n- Run full test suite after upgrade\n- Consider staged rollout for major version bumps"
    )

# Example usage with mock data - replace with actual 8.2 results:
# print(generate_ticket(
#     family_key="com.fasterxml.jackson.core/jackson-databind",
#     vulnerable_versions=["2.9.8", "2.9.10", "2.10.0"],
#     target_version="2.12.3",
#     cve_count=47,
#     critical_high=12,
#     pkg_type="maven",
#     cve_details="CVE-2020-36518, CVE-2020-25649, CVE-2019-14540..."
# ))
```

**8.5.4 Breaking Change Risk Matrix**

| Risk Level | Version Change | Action |
|------------|----------------|--------|
| LOW | Patch bump (1.2.3 → 1.2.4) | Safe to auto-merge |
| MEDIUM | Minor bump (1.2.x → 1.3.x) | Review changelog, run tests |
| HIGH | Major bump (1.x → 2.x) | Manual review required, expect API changes |
| UNKNOWN | Non-semver or unparseable | Manual investigation required |

**8.5.5 Output Formats**

For JIRA integration, export recommendations as CSV:
```python
import csv

# Write to CSV for JIRA import
with open('/tmp/patch_recommendations.csv', 'w', newline='') as f:
    writer = csv.writer(f)
    writer.writerow(['Summary', 'Priority', 'Component', 'Description', 'Labels'])
    # Add rows from your generated tickets
```

For automated PR creation, export as JSON:
```python
# Write to JSON for automation
recommendations = []
# ... populate from analysis ...
with open('/tmp/patch_recommendations.json', 'w') as f:
    json.dump(recommendations, f, indent=2)
```

### STEP 9: CVE Age Analysis

**9.1 Analyze CVE Ages**

From `/tmp/findings.json`:
```python
from datetime import datetime

with open('/tmp/findings.json') as f:
    data = json.load(f)
findings = data['data']['titlePage']['content']

ages = []
for finding in findings:
    published = finding.get('publishedAt')
    if published:
        pub_dt = datetime.fromisoformat(published.replace('Z', '+00:00'))
        age_days = (datetime.now() - pub_dt.replace(tzinfo=None)).days
        ages.append({
            'cve': finding.get('identifier'),
            'severity': finding.get('severity'),
            'age_days': age_days,
            'age_years': age_days / 365
        })

# Sort by age
ages.sort(key=lambda x: x['age_days'], reverse=True)

print("Oldest CVEs Still Present:")
for item in ages[:10]:
    print(f"  {item['cve']} ({item['severity']}): {item['age_years']:.1f} years old")

# Distribution
under_1yr = sum(1 for a in ages if a['age_days'] < 365)
under_2yr = sum(1 for a in ages if 365 <= a['age_days'] < 730)
over_2yr = sum(1 for a in ages if a['age_days'] >= 730)

print(f"\nAge Distribution:")
print(f"  <1 year: {under_1yr} ({under_1yr/len(ages)*100:.1f}%)")
print(f"  1-2 years: {under_2yr} ({under_2yr/len(ages)*100:.1f}%)")
print(f"  >2 years: {over_2yr} ({over_2yr/len(ages)*100:.1f}%)")
```

**9.2 Interpret CVE Ages**

**Red Flags:**
- CRITICAL CVEs >1 year old
- >50% of CVEs >1 year old
- CVEs >3 years old present

**Indicates:**
- Infrequent patching
- Packages stuck on old versions
- Technical debt accumulation

### STEP 10: Package Growth Analysis

**10.1 Track Package Count**

```python
pkg_data = [(m['commitDateTime'][:10], m['packages']) for m in metrics]
pkg_sorted = sorted(pkg_data, key=lambda x: x[0])

print("Package Count Trajectory:")
for date, count in pkg_sorted:
    print(f"  {date}: {count:,} packages")

# Calculate growth
growth = pkg_sorted[-1][1] - pkg_sorted[0][1]
growth_pct = (growth / pkg_sorted[0][1] * 100)
print(f"\nPackage Growth: +{growth:,} ({growth_pct:.1f}%)")
```

**10.2 Interpret Package Growth**

- Controlled growth (<10% over 6 months): ✓ Good
- Moderate growth (10-30%): ○ Acceptable
- High growth (>30%): ✗ Concerning
- Very high growth (>100%): ✗✗ Red flag

**Correlate with Findings:**
- Package growth > findings growth: Improving (good)
- Package growth = findings growth: Stable (neutral)
- Package growth < findings growth: Degrading (bad)

### STEP 11: Synthesize Findings

**11.1 Calculate Overall Grade**

Use this rubric:

| Category | Weight | Grade Criteria |
|----------|--------|----------------|
| PES | 30% | Avg PES: >1=A, 0-1=B, -1-0=C, -2--1=D, <-2=F |
| Findings Trend | 25% | Decreasing=A, Stable=C, Increasing=F |
| Backlog | 20% | <30% in 90+d=A, 30-50%=C, >70%=F |
| RPS Trend | 15% | Decreasing=A, Stable=C, Increasing=F |
| Edit Efficiency | 10% | <30% same=A, 30-50%=C, >66%=F |

**11.2 Identify Root Causes**

Common patterns:

**Pattern 1: High Activity, Negative PES**
- Likely: Too many "same version" edits
- Likely: Not targeting vulnerable packages
- Fix: Stop re-evaluation churn, vulnerability-driven patching

**Pattern 2: Growing Findings Despite Patches**
- Likely: Package growth > patch rate
- Likely: Patches not targeting CVEs
- Fix: Halt package growth, prioritize vulnerable packages

**Pattern 3: High RPS, Negative PES**
- Likely: Decentralized patching without consolidation
- Fix: Version pinning, coordinated upgrades

**Pattern 4: Old CVEs Persist**
- Likely: Technical debt, stuck on old versions
- Fix: Emergency remediation, dependency modernization

### STEP 12: Generate Report

**12.1 Report Generation Script**

This final step ties all analysis together and produces both terminal highlights and a full markdown report.

```python
from datetime import date, datetime
import json

def generate_report(dataset_name, metrics, edits, findings, composition, family_analysis):
    """
    Generate terminal summary and full markdown report.

    Args:
        dataset_name: Name of the dataset
        metrics: List of datasetMetrics records (sorted by commitDateTime)
        edits: List of edit records
        findings: List of findingData records
        composition: Dict from Step 2 composition analysis {type: {count, packages, findings, critical, high}}
        family_analysis: List of dicts from Step 8 [{family_key, findings, clean_versions, ...}]
    """

    # === CALCULATE ALL METRICS ===

    # PES
    avg_pes = sum(m['patchEfficacyScore'] for m in metrics) / len(metrics)
    latest_pes = metrics[-1]['patchEfficacyScore']
    positive_pes_pct = sum(1 for m in metrics if m['patchEfficacyScore'] > 0) / len(metrics) * 100

    # RPS
    rps_start = metrics[0]['rpsScore']
    rps_end = metrics[-1]['rpsScore']
    rps_trend = "Decreasing" if rps_end < rps_start else "Increasing" if rps_end > rps_start else "Stable"

    # Findings
    findings_start = metrics[0]['totalFindings']
    findings_end = metrics[-1]['totalFindings']
    findings_change = findings_end - findings_start
    findings_trend = "Decreasing" if findings_change < 0 else "Increasing" if findings_change > 0 else "Stable"

    # Backlog
    latest = metrics[-1]
    backlog_30_60 = latest.get('findingsInBacklogBetweenThirtyAndSixtyDays', 0)
    backlog_60_90 = latest.get('findingsInBacklogBetweenSixtyAndNinetyDays', 0)
    backlog_90_plus = latest.get('findingsInBacklogOverNinetyDays', 0)
    total_backlog = backlog_30_60 + backlog_60_90 + backlog_90_plus
    pct_90_plus = (backlog_90_plus / total_backlog * 100) if total_backlog > 0 else 0

    # Edit efficiency
    same_edits = sum(1 for e in edits if e.get('sameEdit', False))
    same_edit_pct = (same_edits / len(edits) * 100) if edits else 0

    # Severity breakdown
    critical = latest.get('criticalFindings', 0)
    high = latest.get('highFindings', 0)
    medium = latest.get('mediumFindings', 0)
    low = latest.get('lowFindings', 0)

    # === CALCULATE GRADES ===

    def grade_pes(avg):
        if avg > 1: return 'A'
        if avg > 0: return 'B'
        if avg > -1: return 'C'
        if avg > -2: return 'D'
        return 'F'

    def grade_findings(trend, change_pct):
        if trend == "Decreasing": return 'A'
        if trend == "Stable": return 'C'
        if change_pct < 20: return 'C'
        if change_pct < 50: return 'D'
        return 'F'

    def grade_backlog(pct_90):
        if pct_90 < 30: return 'A'
        if pct_90 < 50: return 'C'
        if pct_90 < 70: return 'D'
        return 'F'

    def grade_rps(trend):
        if trend == "Decreasing": return 'A'
        if trend == "Stable": return 'C'
        return 'F'

    def grade_edits(same_pct):
        if same_pct < 30: return 'A'
        if same_pct < 50: return 'C'
        if same_pct < 66: return 'D'
        return 'F'

    findings_change_pct = abs(findings_change / findings_start * 100) if findings_start > 0 else 0

    grades = {
        'pes': grade_pes(avg_pes),
        'findings': grade_findings(findings_trend, findings_change_pct),
        'backlog': grade_backlog(pct_90_plus),
        'rps': grade_rps(rps_trend),
        'edits': grade_edits(same_edit_pct)
    }

    # Weighted overall grade
    grade_values = {'A': 4, 'B': 3, 'C': 2, 'D': 1, 'F': 0}
    weighted_score = (
        grade_values[grades['pes']] * 0.30 +
        grade_values[grades['findings']] * 0.25 +
        grade_values[grades['backlog']] * 0.20 +
        grade_values[grades['rps']] * 0.15 +
        grade_values[grades['edits']] * 0.10
    )

    if weighted_score >= 3.5: overall_grade = 'A'
    elif weighted_score >= 2.5: overall_grade = 'B'
    elif weighted_score >= 1.5: overall_grade = 'C'
    elif weighted_score >= 0.5: overall_grade = 'D'
    else: overall_grade = 'F'

    # === IDENTIFY CRITICAL ISSUES ===

    critical_issues = []
    if grades['pes'] in ['D', 'F']:
        critical_issues.append(f"PES is negative ({avg_pes:.2f}) - patches making things worse")
    if grades['backlog'] == 'F':
        critical_issues.append(f"Backlog critical: {pct_90_plus:.0f}% of findings are 90+ days old")
    if critical > 0:
        critical_issues.append(f"{critical} CRITICAL severity CVEs present")
    if grades['findings'] == 'F':
        critical_issues.append(f"Findings increased {findings_change_pct:.0f}% - accumulating debt")

    # === TOP RECOMMENDATIONS ===

    recommendations = []
    if family_analysis:
        top_family = max(family_analysis, key=lambda x: x.get('total_finding_instances', 0))
        if top_family.get('total_finding_instances', 0) > 10:
            recommendations.append(f"Consolidate {top_family['family_key']} - eliminates {top_family['total_finding_instances']} CVE instances")
    if grades['backlog'] in ['D', 'F']:
        recommendations.append("Prioritize remediation of 90+ day backlog items")
    if grades['pes'] in ['D', 'F']:
        recommendations.append("Target patches at vulnerable packages, not just version bumps")
    if rps_trend == "Increasing":
        recommendations.append("Implement version pinning to reduce fragmentation")

    # === TERMINAL OUTPUT ===

    print()
    print("=" * 70)
    print(f"  PATCHFOX SECURITY ANALYSIS: {dataset_name}")
    print(f"  Generated: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
    print("=" * 70)
    print()
    print(f"  OVERALL GRADE: {overall_grade}")
    print()
    print(f"  PES: {avg_pes:.2f} (Grade: {grades['pes']})  |  RPS: {rps_end:.1f} ({rps_trend}, Grade: {grades['rps']})")
    print(f"  Findings: {findings_end} ({findings_trend} {abs(findings_change):+d}, Grade: {grades['findings']})")
    print(f"  Backlog: {total_backlog:.0f} total, {pct_90_plus:.0f}% in 90+d (Grade: {grades['backlog']})")
    print(f"  Edit Efficiency: {100-same_edit_pct:.0f}% different-version (Grade: {grades['edits']})")
    print()

    if critical_issues:
        print("  CRITICAL ISSUES:")
        for issue in critical_issues:
            print(f"    ⚠ {issue}")
        print()

    if recommendations:
        print("  TOP RECOMMENDATIONS:")
        for i, rec in enumerate(recommendations[:5], 1):
            print(f"    {i}. {rec}")
        print()

    print("=" * 70)

    # === MARKDOWN REPORT ===

    report_date = date.today().isoformat()
    report_path = f"/tmp/{dataset_name}_analysis_{report_date}.md"

    # Build composition table
    comp_table = "| Type | Datasources | Packages | Findings | Critical | High |\n"
    comp_table += "|------|-------------|----------|----------|----------|------|\n"
    for pkg_type, stats in sorted(composition.items(), key=lambda x: x[1]['findings'], reverse=True):
        comp_table += f"| {pkg_type} | {stats['count']} | {stats['packages']} | {stats['findings']} | {stats['critical']} | {stats['high']} |\n"

    # Build family remediation table
    family_table = ""
    if family_analysis:
        family_table = "| Package Family | CVE Instances | Versions | Vulnerable | Clean |\n"
        family_table += "|----------------|---------------|----------|------------|-------|\n"
        for fam in sorted(family_analysis, key=lambda x: x.get('total_finding_instances', 0), reverse=True)[:15]:
            family_table += f"| {fam.get('family_key', 'N/A')} | {fam.get('total_finding_instances', 0)} | {fam.get('total_family_members', 0)} | {fam.get('family_members_with_findings', 0)} | {fam.get('family_members_without_findings', 0)} |\n"

    report_content = f"""# PatchFox Security Analysis Report

**Dataset:** {dataset_name}
**Generated:** {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}
**Analysis Period:** {metrics[0]['commitDateTime'][:10]} to {metrics[-1]['commitDateTime'][:10]}

---

## Executive Summary

**Overall Grade: {overall_grade}**

This dataset has a Patch Efficacy Score (PES) of **{avg_pes:.2f}**, indicating {"effective patching that improves security posture" if avg_pes > 0 else "patches that are not effectively improving security"}.
The vulnerability backlog shows **{pct_90_plus:.0f}%** of findings aged 90+ days, {"indicating active remediation" if pct_90_plus < 30 else "indicating remediation lag"}.
{"Key concerns include: " + "; ".join(critical_issues[:3]) + "." if critical_issues else "No critical issues identified."}

---

## Part 1: Dataset Composition

{comp_table}

**Analysis Period:** {len(metrics)} snapshots analyzed

---

## Part 2: Patch Efficacy (PES)

| Metric | Value | Grade |
|--------|-------|-------|
| Average PES | {avg_pes:.2f} | {grades['pes']} |
| Latest PES | {latest_pes:.2f} | - |
| Positive PES % | {positive_pes_pct:.1f}% | - |
| Average Impact | {sum(m['patchImpact'] for m in metrics) / len(metrics):.2f} | - |
| Average Effort | {sum(m['patchEffort'] for m in metrics) / len(metrics):.2f} | - |

**Interpretation:** {"Patches are effectively reducing vulnerabilities and complexity." if avg_pes > 1 else "Patches are having neutral to negative security impact." if avg_pes < 0 else "Patches are having modest positive impact."}

---

## Part 3: Findings & Backlog

### Current Severity Breakdown

| Severity | Count |
|----------|-------|
| CRITICAL | {critical} |
| HIGH | {high} |
| MEDIUM | {medium} |
| LOW | {low} |
| **TOTAL** | **{findings_end}** |

### Findings Trend

- **Start:** {findings_start} findings
- **End:** {findings_end} findings
- **Change:** {findings_change:+d} ({findings_change_pct:.1f}%)
- **Trend:** {findings_trend}
- **Grade:** {grades['findings']}

### Backlog Aging

| Bucket | Count | Percentage |
|--------|-------|------------|
| 30-60 days | {backlog_30_60:.0f} | {(backlog_30_60/total_backlog*100) if total_backlog > 0 else 0:.1f}% |
| 60-90 days | {backlog_60_90:.0f} | {(backlog_60_90/total_backlog*100) if total_backlog > 0 else 0:.1f}% |
| 90+ days | {backlog_90_plus:.0f} | {pct_90_plus:.1f}% |
| **Total** | **{total_backlog:.0f}** | 100% |

**Grade:** {grades['backlog']}

---

## Part 4: Version Fragmentation (RPS)

| Metric | Value |
|--------|-------|
| Starting RPS | {rps_start:.1f} |
| Ending RPS | {rps_end:.1f} |
| Change | {rps_end - rps_start:+.1f} |
| Trend | {rps_trend} |
| Grade | {grades['rps']} |

**Interpretation:** {"Version consolidation is improving - fewer duplicate package versions." if rps_trend == "Decreasing" else "Version fragmentation is increasing - more duplicate package versions across the codebase."}

---

## Part 5: Edit Analysis

| Metric | Value |
|--------|-------|
| Total Edits | {len(edits):,} |
| Same-Version Edits | {same_edits:,} ({same_edit_pct:.1f}%) |
| Different-Version Edits | {len(edits) - same_edits:,} ({100-same_edit_pct:.1f}%) |
| Grade | {grades['edits']} |

---

## Part 6: Package Family Remediation Opportunities

{family_table if family_table else "*No multi-version package families with remediation opportunities identified.*"}

{"**Top Opportunity:** " + family_analysis[0]['family_key'] + f" - consolidating to a clean version could eliminate {family_analysis[0].get('total_finding_instances', 0)} CVE instances." if family_analysis and family_analysis[0].get('total_finding_instances', 0) > 0 else ""}

---

## Part 7: Grading Summary

| Category | Weight | Grade | Score |
|----------|--------|-------|-------|
| PES | 30% | {grades['pes']} | {grade_values[grades['pes']]} |
| Findings Trend | 25% | {grades['findings']} | {grade_values[grades['findings']]} |
| Backlog | 20% | {grades['backlog']} | {grade_values[grades['backlog']]} |
| RPS Trend | 15% | {grades['rps']} | {grade_values[grades['rps']]} |
| Edit Efficiency | 10% | {grades['edits']} | {grade_values[grades['edits']]} |
| **Overall** | 100% | **{overall_grade}** | **{weighted_score:.2f}** |

---

## Part 8: Recommendations

### Critical (0-30 days)

{chr(10).join(f"- {issue}" for issue in critical_issues) if critical_issues else "- No critical issues requiring immediate action"}

### High Priority (30-90 days)

{chr(10).join(f"- {rec}" for rec in recommendations) if recommendations else "- Continue current practices"}

### Medium Priority (90-180 days)

- Review and update dependency policies
- Implement automated vulnerability scanning in CI/CD
- Establish version pinning standards

---

## Conclusion

{"This dataset demonstrates **effective security practices** with positive patch efficacy and controlled vulnerability growth." if overall_grade in ['A', 'B'] else "This dataset shows **areas for improvement** in patch targeting and remediation velocity." if overall_grade == 'C' else "This dataset has **significant security concerns** that require immediate attention and process changes."}

**Bottom Line:** Grade **{overall_grade}** - {
    "Excellent security posture, maintain current practices" if overall_grade == 'A' else
    "Good security posture with minor improvements needed" if overall_grade == 'B' else
    "Acceptable but needs focused improvement" if overall_grade == 'C' else
    "Poor security posture, significant changes required" if overall_grade == 'D' else
    "Critical security failure, emergency intervention needed"
}

---

*Report generated by PatchFox Dataset Analysis Runbook v1.0*
"""

    with open(report_path, 'w') as f:
        f.write(report_content)

    print(f"  Full report written to: {report_path}")
    print()

    return report_path, overall_grade, grades
```

**12.2 Usage Example**

```python
# After completing Steps 1-11, call generate_report with collected data:

# Fetch all required data
metrics = fetch_metrics()  # From Step 1
metrics.sort(key=lambda x: x['commitDateTime'])

edits = fetch_edits()  # From Step 1
findings = fetch_findings()  # From Step 1

# Build composition dict from Step 2
composition = {}  # {type: {count, packages, findings, critical, high}}
# ... populate from datasourceMetricsCurrent analysis

# Build family analysis from Step 8
family_analysis = []  # [{family_key, total_finding_instances, ...}]
# ... populate from SQL query results

# Generate report
report_path, grade, grades = generate_report(
    dataset_name="MyDataset",
    metrics=metrics,
    edits=edits,
    findings=findings,
    composition=composition,
    family_analysis=family_analysis
)

print(f"Analysis complete. Grade: {grade}")
```

**12.3 Sample Terminal Output**

```
======================================================================
  PATCHFOX SECURITY ANALYSIS: nationalsecurityagency
  Generated: 2026-01-19 10:30:45
======================================================================

  OVERALL GRADE: C

  PES: -0.42 (Grade: C)  |  RPS: 12.3 (Increasing, Grade: F)
  Findings: 111 (Increasing +43, Grade: D)
  Backlog: 89 total, 72% in 90+d (Grade: D)
  Edit Efficiency: 34% different-version (Grade: D)

  CRITICAL ISSUES:
    ⚠ Backlog critical: 72% of findings are 90+ days old
    ⚠ 3 CRITICAL severity CVEs present
    ⚠ Findings increased 63% - accumulating debt

  TOP RECOMMENDATIONS:
    1. Consolidate com.fasterxml.jackson.core/jackson-databind - eliminates 47 CVE instances
    2. Prioritize remediation of 90+ day backlog items
    3. Target patches at vulnerable packages, not just version bumps
    4. Implement version pinning to reduce fragmentation

======================================================================
  Full report written to: /tmp/nationalsecurityagency_analysis_2026-01-19.md
```

---

## 6. Common Pitfalls

### ❌ PITFALL #1: Misunderstanding the `sameEdit` Flag

**WRONG:**
"66% of edits are 'same version' re-evaluations with NO version change and ZERO security potential"

**CORRECT:**
"66% of edits have `sameEdit=true`, meaning these changes were made 2+ times in the previous 90 days (coordinated/repeated changes)"

**Key:** The `sameEdit` flag does NOT mean "no version change occurred." It means "this same version change was applied 2+ times recently" (e.g., bulk monorepo upgrades). These edits DO change versions and CAN improve security.

**Example:**
```
Before: datawave-core@7.27.1-SNAPSHOT
After: datawave-core@7.23.6
sameEdit: true (change made 3x in 90 days)
sameEditCount: 3
```

**Why This Matters:** Misinterpreting `sameEdit` leads to wrongly concluding that 66% of patches are useless. In reality, they're coordinated releases that may or may not target security issues.

### ❌ PITFALL #2: Misunderstanding CVE Timing (Two Scenarios)

**WRONG:**
"Findings appeared 574 days after CVE publication - they're slow to detect vulnerabilities"

**CORRECT:**
"There are TWO types of CVE timing scenarios you must distinguish:"

**Scenario A: Old CVEs (Remediation Lag)**
- CVE published: 2021-04-23 (CVE-2021-26291, CRITICAL)
- Package added: Sometime between 2021-2025 (unknown)
- Most recent commit: Nov 18, 2025
- PatchFox scan: Jan 18, 2026
- **Interpretation:** Package with 4.7-year-old CVE still present = **remediation failure**

**Scenario B: Recent CVEs (Normal Lifecycle)**
- Package added: July 2025 (urllib3 2.3.0, safe at the time)
- Most recent commit: Nov 18, 2025 (urllib3 still present)
- CVE published: Jan 7, 2026 (CVE-2026-21441 discovered AFTER package was added)
- PatchFox scan: Jan 18, 2026
- **Interpretation:** New vulnerability in existing package = **normal lifecycle, needs remediation**

**Key:**
- `publishedAt` = when NVD published the CVE (public knowledge date)
- `reportedAt` = when PatchFox detected it in their packages
- Compare `publishedAt` to `commitDateTime` (when package was added) NOT to `reportedAt`
- If CVE published BEFORE commit: remediation lag (bad)
- If CVE published AFTER most recent commit: ongoing discovery (normal, but needs response)

### ❌ PITFALL #3: Confusing Activity with Outcomes

**WRONG:**
"High patch count = good security"

**CORRECT:**
"High patch count + negative PES + growing findings = busy work without security impact"

**Key:** Measure outcomes (PES, findings trend, RPS), not activity (patch count). You can make 20,000 patches and still have worsening security.

### ❌ PITFALL #4: Ignoring Package Growth

**WRONG:**
"Vulnerable % decreased from 2.5% to 1.5%, security improved"

**CORRECT:**
"Vulnerable % decreased, but package count doubled. Absolute vulnerable packages increased 36 → 45 (+25%)"

**Key:** Track both percentage AND absolute counts. Fast package growth can hide security degradation. A lower percentage with more total vulnerabilities is NOT improvement.

### ❌ PITFALL #5: Misinterpreting Negative PES

**WRONG:**
"Negative PES means patches introduce more CVEs"

**CORRECT:**
"Negative PES means patches INCREASE complexity (RPS, downlevel) without reducing vulnerabilities"

**Key:** PES = Patch Impact / Patch Effort. Negative PES means impact is negative (making metrics worse) even though effort is positive (patches are being made). This happens when:
- RPS increases (version fragmentation)
- Findings grow
- Downlevel packages increase
- Stale packages accumulate

### ❌ PITFALL #6: Not Checking Backlog Aging

**WRONG:**
"90 findings is acceptable for a large codebase"

**CORRECT:**
"90 findings with 82 (91%) aged 90+ days means NO remediation velocity - findings accumulate but never close"

**Key:** Backlog aging shows whether vulnerabilities are being addressed or accumulating. If 90%+ are in the 90+ day bucket and the number is growing, there's no remediation happening.

### ❌ PITFALL #7: Assuming Low Vuln % = Good Security

**WRONG:**
"1.39% vulnerable packages is excellent, top-tier security"

**CORRECT:**
"1.39% vulnerable WITH 22 CVEs over 3 years old means packages aren't being upgraded, not that security is excellent"

**Key:** Low vulnerable % combined with ancient CVEs is a RED FLAG. It means:
- Old dependencies not being upgraded
- Possibly infrequent scanning
- Remediation lag, not security success

Look at CVE AGE DISTRIBUTION, not just vulnerable package percentage.

### ❌ PITFALL #8: Not Understanding the Vulnerability Lifecycle

**WRONG:**
"Findings grew from 68 → 111, they must be introducing vulnerable code"

**CORRECT:**
"Findings grew from 68 → 111 because (a) new CVEs are discovered in existing packages over time, AND (b) old CVEs aren't being remediated"

**Key:** Even if you FREEZE all code changes, findings will continue to grow as security researchers discover new vulnerabilities in existing dependencies. This is normal. What matters is:
- **Remediation velocity:** Are old CVEs being fixed?
- **CVE age distribution:** Are most CVEs recent (normal) or ancient (remediation failure)?

**Example:**
- Package `urllib3 2.3.0` added July 2025 (safe)
- November 2025: No code changes
- January 2026: New CVE discovered affecting urllib3 2.3.0
- Finding count increases WITHOUT any new code

This is NOT a failure to avoid vulnerable code - it's the reality of dependency management.

---

## 7. Interpretation Guidelines

### 7.1 PES Interpretation Matrix

| PES Range | Impact | Effort | Meaning | Action |
|-----------|--------|--------|---------|--------|
| +5 to +10 | High positive | Low-Medium | Excellent efficiency | Replicate approach |
| +1 to +5 | Positive | Medium | Good progress | Continue strategy |
| -1 to +1 | Near zero | Low-High | Treading water | Reassess priorities |
| -1 to -5 | Negative | Medium-High | Making worse | Major change needed |
| < -5 | High negative | High | Severe degradation | Emergency intervention |

### 7.2 Findings Trend Interpretation

| Change | Package Growth | Meaning | Grade |
|--------|----------------|---------|-------|
| Decreasing | Any | Active remediation | A |
| Stable | <10% | Keeping pace | B |
| Stable | >30% | Falling behind | D |
| Increasing | <10% | New CVEs detected | C |
| Increasing | >30% | Accumulating debt | F |

### 7.3 Edit Type Interpretation

| Same Version % | Different % | CREATE vs DELETE | Assessment |
|----------------|-------------|------------------|------------|
| <30% | >70% | DELETE > CREATE | Excellent |
| 30-50% | 50-70% | Balanced | Good |
| 50-70% | 30-50% | CREATE > DELETE | Poor |
| >70% | <30% | CREATE >> DELETE | Failing |

### 7.4 Backlog Interpretation

| 90+ Day % | Trend | Meaning | Action |
|-----------|-------|---------|--------|
| <30% | Decreasing | Active resolution | Maintain |
| 30-50% | Stable | Treading water | Increase capacity |
| 50-70% | Increasing | Falling behind | Prioritize remediation |
| >70% | Increasing | Critical failure | Emergency response |

---

## 8. Report Structure Template

### Executive Summary (1 paragraph)

State:
1. Overall grade (A-F)
2. Key finding (PES score)
3. Main problem (e.g., "66% of patches ineffective")
4. Bottom line verdict

### Part 1: Patch Activity

- Total edit count
- Edit type breakdown (UPDATE, CREATE, DELETE)
- Same vs different version breakdown
- Temporal distribution
- Most updated packages

### Part 2: Patch Efficacy

- PES calculation and trend
- RPS trend
- Why positive or negative
- Grade and distribution

### Part 3: Findings & Backlog

- Findings trajectory
- Severity breakdown
- Backlog aging analysis
- CVE age analysis
- Trend interpretation

### Part 4: Root Cause Analysis

- Why patches don't reduce findings
- Why RPS increases/decreases
- Why backlog accumulates/resolves
- Systemic issues identified

### Part 5: Detailed Metrics

- RPS deep dive
- Downlevel/stale analysis
- Package growth correlation
- Edit type patterns

### Part 6: Grading

- Individual component grades
- Overall grade with rationale
- Comparison to industry standards

### Part 7: Recommendations

**Critical (0-30 days):**
- Emergency fixes
- Stop doing harmful things

**High Priority (30-90 days):**
- Structural changes
- New processes

**Medium Priority (90-180 days):**
- Optimization
- Tooling improvements

### Part 8: Conclusion

- What's working
- What's failing
- Bottom line assessment
- Grade justification

---

## 9. Example Analysis Script

Here's a complete Python script that performs the core analysis:

```python
#!/usr/bin/env python3
import json
import requests
from datetime import datetime, timedelta
from collections import Counter

DATA_SERVICE_URL = "http://localhost:1702"
DATASET_NAME = "YOUR_DATASET_NAME"

def fetch_metrics():
    url = f"{DATA_SERVICE_URL}/api/v1/db/datasetMetrics/query"
    params = {'datasetName': DATASET_NAME, 'isCurrent': 'true'}
    response = requests.get(url, params=params, timeout=30)
    response.raise_for_status()
    data = response.json()
    return data['data']['titlePage']['content']

def fetch_edits():
    six_months_ago = (datetime.now() - timedelta(days=180)).strftime('%Y-%m-%dT%H:%M:%SZ')
    url = f"{DATA_SERVICE_URL}/api/v1/db/datasetMetrics/edit/query"
    params = {
        'datasetName': DATASET_NAME,
        'isCurrent': 'true',
        'commitDateTime': f'gte.{six_months_ago}',
        'size': 25000
    }
    response = requests.get(url, params=params, timeout=60)
    response.raise_for_status()
    data = response.json()
    return data['data']['titlePage']['content']

def fetch_findings():
    url = f"{DATA_SERVICE_URL}/api/v1/db/findingData/query"
    params = {'size': 500}
    response = requests.get(url, params=params, timeout=30)
    response.raise_for_status()
    data = response.json()
    return data['data']['titlePage']['content']

def analyze_pes(metrics):
    metrics.sort(key=lambda x: x['commitDateTime'])

    avg_pes = sum(m['patchEfficacyScore'] for m in metrics) / len(metrics)
    avg_impact = sum(m['patchImpact'] for m in metrics) / len(metrics)
    avg_effort = sum(m['patchEffort'] for m in metrics) / len(metrics)

    positive = sum(1 for m in metrics if m['patchEfficacyScore'] > 0)
    negative = sum(1 for m in metrics if m['patchEfficacyScore'] < 0)

    print("=== PES ANALYSIS ===")
    print(f"Average PES: {avg_pes:.2f}")
    print(f"Average Impact: {avg_impact:.2f}")
    print(f"Average Effort: {avg_effort:.2f}")
    print(f"Positive PES: {positive}/{len(metrics)} ({positive/len(metrics)*100:.1f}%)")
    print(f"Negative PES: {negative}/{len(metrics)} ({negative/len(metrics)*100:.1f}%)")

def analyze_edits(edits):
    edit_types = Counter(e['editType'] for e in edits)
    same_edits = sum(1 for e in edits if e['sameEdit'])
    different_edits = len(edits) - same_edits

    print("\n=== EDIT ANALYSIS ===")
    print(f"Total Edits: {len(edits):,}")
    for et, count in edit_types.most_common():
        print(f"  {et}: {count:,} ({count/len(edits)*100:.1f}%)")
    print(f"Same version: {same_edits:,} ({same_edits/len(edits)*100:.1f}%)")
    print(f"Different version: {different_edits:,} ({different_edits/len(edits)*100:.1f}%)")

def analyze_findings_trend(metrics):
    metrics.sort(key=lambda x: x['commitDateTime'])

    print("\n=== FINDINGS TREND ===")
    for m in metrics:
        print(f"{m['commitDateTime'][:10]}: {m['totalFindings']} findings, {m['packages']} packages")

    change = metrics[-1]['totalFindings'] - metrics[0]['totalFindings']
    print(f"\nChange: {change:+d} findings")

def main():
    print(f"Analyzing dataset: {DATASET_NAME}\n")

    metrics = fetch_metrics()
    edits = fetch_edits()
    findings = fetch_findings()

    analyze_pes(metrics)
    analyze_edits(edits)
    analyze_findings_trend(metrics)

    print("\n=== Analysis complete ===")

if __name__ == "__main__":
    main()
```

---

## 10. Checklist

Before finalizing your report, verify:

- [ ] Fetched all required data (metrics, edits, findings)
- [ ] Calculated PES average and distribution
- [ ] Analyzed RPS trend (increasing/decreasing)
- [ ] Analyzed findings trend (increasing/decreasing)
- [ ] Calculated edit type breakdown (same vs different %)
- [ ] Analyzed backlog aging (% in 90+ days)
- [ ] Analyzed CVE ages (oldest vulnerabilities)
- [ ] Tracked package growth correlation
- [ ] Identified root causes (why negative/positive trends)
- [ ] Provided specific recommendations with timelines
- [ ] Assigned component grades and overall grade
- [ ] Verified interpretations against common pitfalls
- [ ] Included data sources and analysis date

---

## Appendix: Quick Reference

### API Endpoints Quick Copy

```bash
# Dataset name
curl -s "http://localhost:1702/api/v1/db/dataset/query"

# Metrics
curl -s "http://localhost:1702/api/v1/db/datasetMetrics/query?datasetName={NAME}&isCurrent=true"

# Edits
curl -s "http://localhost:1702/api/v1/db/datasetMetrics/edit/query?datasetName={NAME}&isCurrent=true&size=25000"

# Findings
curl -s "http://localhost:1702/api/v1/db/findingData/query?size=500"
```

### Grading Scale

| Grade | PES Range | Findings | Backlog | RPS |
|-------|-----------|----------|---------|-----|
| A | >+1 | Decreasing | <30% in 90+d | Decreasing |
| B | 0 to +1 | Stable | 30-50% in 90+d | Stable |
| C | -1 to 0 | Slight increase | 50-70% in 90+d | Slight increase |
| D | -2 to -1 | Increasing | >70% in 90+d | Increasing |
| F | <-2 | Rapidly increasing | >90% in 90+d | Rapidly increasing |

### Key Metrics Interpretation

- **PES < 0:** Patches making things worse
- **RPS increasing:** Version fragmentation
- **Findings increasing:** Accumulating vulnerabilities
- **>70% same version edits:** Wasting effort
- **>70% backlog in 90+ days:** Not remediating
- **CVEs >2 years old:** Technical debt

---

**Document Version:** 1.0
**Last Updated:** January 18, 2026
**Maintainer:** PatchFox Team
