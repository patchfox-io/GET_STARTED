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

### STEP 2: PES Analysis

**2.1 Calculate PES Statistics**

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

**2.2 Identify PES Trends**

Track over time:
```python
for m in metrics:
    print(f"{m['commitDateTime'][:10]}: PES={m['patchEfficacyScore']:.2f}, Impact={m['patchImpact']:.2f}, Effort={m['patchEffort']:.2f}")
```

**2.3 Grade PES Performance**
- Average PES > 1: A (Excellent)
- Average PES 0 to 1: B (Good)
- Average PES -1 to 0: C (Acceptable)
- Average PES -2 to -1: D (Poor)
- Average PES < -2: F (Failing)

### STEP 3: RPS Analysis

**3.1 Calculate RPS Trend**

```python
rps_values = [(m['commitDateTime'][:10], m['rpsScore']) for m in metrics]
rps_sorted = sorted(rps_values, key=lambda x: x[0])

print(f"RPS Earliest: {rps_sorted[0]}")
print(f"RPS Latest: {rps_sorted[-1]}")
print(f"RPS Change: {rps_sorted[-1][1] - rps_sorted[0][1]:.2f}")
```

**3.2 Interpret RPS**
- Decreasing RPS: ✓ Good (consolidating versions)
- Stable RPS: ○ Neutral
- Increasing RPS: ✗ Bad (fragmenting versions)

### STEP 4: Findings Analysis

**4.1 Calculate Findings Trend**

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

**4.2 Severity Breakdown**

```python
latest = metrics[-1]
print(f"Current Findings (Severity):")
print(f"  CRITICAL: {latest['criticalFindings']}")
print(f"  HIGH: {latest['highFindings']}")
print(f"  MEDIUM: {latest['mediumFindings']}")
print(f"  LOW: {latest['lowFindings']}")
print(f"  TOTAL: {latest['totalFindings']}")
```

**4.3 Interpret Findings Trend**
- Decreasing findings: ✓ Good (effective remediation)
- Stable findings: ○ Neutral (treading water)
- Increasing findings: ✗ Bad (accumulating debt)

### STEP 5: Backlog Analysis

**5.1 Calculate Backlog Distribution**

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

**5.2 Interpret Backlog**
- Growing backlog: ✗ Bad (not resolving)
- Stable backlog: ○ Neutral (treading water)
- Shrinking backlog: ✓ Good (actively resolving)
- >70% in 90+ days: ✗ Very bad (old vulnerabilities)

### STEP 6: Edit Type Analysis

**6.1 Count Edit Types**

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

**6.2 Interpret Edit Types**

**Red Flags:**
- >50% same version edits: Wasting effort
- CREATE > DELETE: Growing attack surface uncontrolled
- High activity + negative PES: Busy work without benefit

**Good Patterns:**
- <30% same version edits
- CREATE ≈ DELETE (controlled growth)
- High different version % + positive PES

### STEP 7: CVE Age Analysis

**7.1 Analyze CVE Ages**

From `/tmp/findings.json`:
```python
from datetime import datetime

with open('/tmp/findings.json') as f:
    data = json.load(f)
findings = data['data']['titlePage']['content']

ages = []
for f in findings:
    published = f.get('publishedAt')
    if published:
        pub_dt = datetime.fromisoformat(published.replace('Z', '+00:00'))
        age_days = (datetime.now() - pub_dt.replace(tzinfo=None)).days
        ages.append({
            'cve': f.get('identifier'),
            'severity': f.get('severity'),
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

**7.2 Interpret CVE Ages**

**Red Flags:**
- CRITICAL CVEs >1 year old
- >50% of CVEs >1 year old
- CVEs >3 years old present

**Indicates:**
- Infrequent patching
- Packages stuck on old versions
- Technical debt accumulation

### STEP 8: Package Growth Analysis

**8.1 Track Package Count**

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

**8.2 Interpret Package Growth**

- Controlled growth (<10% over 6 months): ✓ Good
- Moderate growth (10-30%): ○ Acceptable
- High growth (>30%): ✗ Concerning
- Very high growth (>100%): ✗✗ Red flag

**Correlate with Findings:**
- Package growth > findings growth: Improving (good)
- Package growth = findings growth: Stable (neutral)
- Package growth < findings growth: Degrading (bad)

### STEP 9: Synthesize Findings

**9.1 Calculate Overall Grade**

Use this rubric:

| Category | Weight | Grade Criteria |
|----------|--------|----------------|
| PES | 30% | Avg PES: >1=A, 0-1=B, -1-0=C, -2--1=D, <-2=F |
| Findings Trend | 25% | Decreasing=A, Stable=C, Increasing=F |
| Backlog | 20% | <30% in 90+d=A, 30-50%=C, >70%=F |
| RPS Trend | 15% | Decreasing=A, Stable=C, Increasing=F |
| Edit Efficiency | 10% | <30% same=A, 30-50%=C, >66%=F |

**9.2 Identify Root Causes**

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
