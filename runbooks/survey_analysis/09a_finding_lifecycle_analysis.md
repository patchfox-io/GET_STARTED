# Step 9a: Finding Lifecycle Analysis - The Complete Security Story

**Purpose:** Track the complete lifecycle of every finding across the dataset's entire history: introduction, shadow exposure, discovery lag, remediation, and re-introduction.

---

## Prerequisites

- Completed [Step 1: Data Collection](./01_data_collection.md)
- Active dataset with analyzed findings
- Direct database access

---

## The Story Question

Finding lifecycle analysis reveals your complete security posture over time:
- **Shadow Findings:** How long were vulnerabilities present BEFORE the CVE was published? (Zero-day window)
- **Discovery Lag:** How long after CVE publication did you introduce the vulnerable package? (Adoption lag)
- **Remediation Velocity:** How long did it take to patch findings once introduced?
- **Re-introduction Rate:** How often are findings removed and then re-added?
- **Net Security Trend:** Are you introducing findings faster than you're remediating them?

This is the most comprehensive security analysis - it tracks every finding from introduction through remediation across the entire dataset history, accounting for packages that are removed and re-added multiple times.

---

## Critical Implementation Note

**The `package` and `finding` tables are append-only historical records.** You CANNOT query them directly for current state. Instead:

1. Walk through ALL `dataset_metrics` records in chronological order (ASC by `commit_date_time`)
2. Use the `package_indexes` field to track which packages are present at each snapshot
3. Track when package-finding pairs appear (introduced) and disappear (removed)
4. Account for re-introductions (same package-finding pair can appear multiple times)

---

## 9a.1 Extract Complete Finding Lifecycle

```python
#!/usr/bin/env python3
"""
Complete Finding Lifecycle Analysis
Track introduction, remediation, and shadow exposure across entire dataset history
"""

import psycopg2
from datetime import datetime
from collections import defaultdict
import json

conn = psycopg2.connect(
    host="localhost",
    port=54321,
    database="mrs_db",
    user="mr_data",
    password="omnomdata"
)

cur = conn.cursor()

# Get dataset_id
DATASET_NAME = 'YOUR_DATASET_NAME'  # CHANGE THIS
cur.execute("SELECT id FROM dataset WHERE name = %s", (DATASET_NAME,))
dataset_id = cur.fetchone()[0]

print("Fetching all dataset_metrics snapshots...")
cur.execute("""
    SELECT id, commit_date_time, package_indexes, total_findings
    FROM dataset_metrics
    WHERE dataset_id = %s AND is_current = true
    ORDER BY commit_date_time ASC
""", (dataset_id,))

snapshots = cur.fetchall()
first_date = snapshots[0][1]
last_date = snapshots[-1][1]
total_days = (last_date - first_date).days

print(f"Found {len(snapshots)} snapshots")
print(f"Period: {first_date} to {last_date} ({total_days} days / {total_days/365:.1f} years)\n")

# Pre-fetch all package-finding relationships and finding metadata
print("Loading package-finding relationships...")
cur.execute("""
    SELECT p.id, f.identifier, fd.severity, fd.published_at
    FROM package p
    JOIN package_finding pf ON p.id = pf.package_id
    JOIN finding f ON f.id = pf.finding_id
    LEFT JOIN finding_data fd ON fd.finding_id = f.id
""")

package_findings = defaultdict(list)
for pkg_id, cve, severity, published_at in cur.fetchall():
    package_findings[pkg_id].append({
        'cve': cve,
        'severity': severity,
        'published_at': published_at
    })

print(f"Loaded findings for {len(package_findings)} packages\n")

# Track ALL introductions and removals (can happen multiple times)
all_introductions = []  # List of (pkg_id, cve, commit_dt)
all_removals = []       # List of (pkg_id, cve, commit_dt)
shadow_findings = []
discovery_lag_findings = []

# Track first introduction for shadow/lag analysis
first_introduction = {}  # (pkg_id, cve) -> commit_date_time

previous_packages = set()
snapshot_count = 0

print("Processing snapshots...")
for snapshot_id, commit_dt, package_indexes, total_findings in snapshots:
    snapshot_count += 1
    if snapshot_count % 500 == 0:
        print(f"  Processed {snapshot_count}/{len(snapshots)}...")
    
    if not package_indexes:
        continue
    
    current_packages = set(package_indexes)
    
    # New packages introduced
    new_packages = current_packages - previous_packages
    for pkg_id in new_packages:
        for finding in package_findings.get(pkg_id, []):
            key = (pkg_id, finding['cve'])
            all_introductions.append((pkg_id, finding['cve'], commit_dt))
            
            # Only track shadow/lag on FIRST introduction
            if key not in first_introduction:
                first_introduction[key] = commit_dt
                
                if finding['published_at'] and commit_dt < finding['published_at']:
                    shadow_days = (finding['published_at'] - commit_dt).days
                    shadow_findings.append({
                        'pkg_id': pkg_id,
                        'cve': finding['cve'],
                        'severity': finding['severity'],
                        'commit_date': commit_dt,
                        'published_date': finding['published_at'],
                        'shadow_days': shadow_days
                    })
                
                elif finding['published_at'] and commit_dt > finding['published_at']:
                    lag_days = (commit_dt - finding['published_at']).days
                    discovery_lag_findings.append({
                        'pkg_id': pkg_id,
                        'cve': finding['cve'],
                        'severity': finding['severity'],
                        'commit_date': commit_dt,
                        'published_date': finding['published_at'],
                        'lag_days': lag_days
                    })
    
    # Packages removed
    removed_packages = previous_packages - current_packages
    for pkg_id in removed_packages:
        for finding in package_findings.get(pkg_id, []):
            all_removals.append((pkg_id, finding['cve'], commit_dt))
    
    previous_packages = current_packages

print(f"\nProcessed all {len(snapshots)} snapshots\n")

# Calculate current state from final snapshot
final_packages = previous_packages
currently_present = []
for pkg_id in final_packages:
    for finding in package_findings.get(pkg_id, []):
        currently_present.append((pkg_id, finding['cve']))

# Calculate remediation stats
remediation_times = []
for pkg_id, cve, removed_dt in all_removals:
    key = (pkg_id, cve)
    if key in first_introduction:
        intro_dt = first_introduction[key]
        days = (removed_dt - intro_dt).days
        if days >= 0:
            remediation_times.append(days)

# Save detailed results
output = {
    'period': {
        'start_date': str(first_date),
        'end_date': str(last_date),
        'total_days': total_days,
        'total_years': round(total_days / 365, 1)
    },
    'all_time': {
        'unique_findings_introduced': len(first_introduction),
        'total_introductions': len(all_introductions),
        'total_removals': len(all_removals),
        're_introductions': len(all_introductions) - len(first_introduction)
    },
    'current_state': {
        'findings_present': len(currently_present)
    },
    'shadow_findings': {
        'total': len(shadow_findings),
        'by_severity': {}
    },
    'discovery_lag': {
        'total': len(discovery_lag_findings)
    },
    'remediation': {
        'total_remediations': len(remediation_times)
    }
}

# Calculate averages
if remediation_times:
    output['remediation']['avg_days'] = round(sum(remediation_times) / len(remediation_times))
    output['remediation']['avg_years'] = round(output['remediation']['avg_days'] / 365, 1)

if shadow_findings:
    shadow_days = [f['shadow_days'] for f in shadow_findings]
    output['shadow_findings']['avg_days'] = round(sum(shadow_days) / len(shadow_days))
    output['shadow_findings']['avg_years'] = round(output['shadow_findings']['avg_days'] / 365, 1)
    output['shadow_findings']['max_days'] = max(shadow_days)
    output['shadow_findings']['max_years'] = round(max(shadow_days) / 365, 1)
    
    # By severity
    shadow_by_severity = defaultdict(list)
    for sf in shadow_findings:
        shadow_by_severity[sf['severity']].append(sf['shadow_days'])
    
    for sev in ['CRITICAL', 'HIGH', 'MEDIUM', 'LOW']:
        if sev in shadow_by_severity:
            days = shadow_by_severity[sev]
            output['shadow_findings']['by_severity'][sev] = {
                'count': len(days),
                'avg_days': round(sum(days) / len(days)),
                'avg_years': round(sum(days) / len(days) / 365, 1)
            }

if discovery_lag_findings:
    lag_days = [f['lag_days'] for f in discovery_lag_findings]
    output['discovery_lag']['avg_days'] = round(sum(lag_days) / len(lag_days))
    output['discovery_lag']['avg_years'] = round(output['discovery_lag']['avg_days'] / 365, 1)
    output['discovery_lag']['max_days'] = max(lag_days)
    output['discovery_lag']['max_years'] = round(max(lag_days) / 365, 1)

with open('/tmp/finding_lifecycle.json', 'w') as f:
    json.dump(output, f, indent=2)

print("Results saved to /tmp/finding_lifecycle.json")

cur.close()
conn.close()
```

---

## 9a.2 Generate Report

```python
#!/usr/bin/env python3
import json

with open('/tmp/finding_lifecycle.json') as f:
    data = json.load(f)

print("=" * 80)
print("FINDING LIFECYCLE ANALYSIS")
print("=" * 80)

# Period
p = data['period']
print(f"\n📅 ANALYSIS PERIOD")
print(f"   {p['start_date']} to {p['end_date']}")
print(f"   {p['total_days']:,} days ({p['total_years']} years)")

# All-time metrics
at = data['all_time']
print(f"\n📊 ALL-TIME METRICS")
print(f"   Unique findings introduced: {at['unique_findings_introduced']:,}")
print(f"   Total introductions: {at['total_introductions']:,}")
print(f"   Total removals: {at['total_removals']:,}")
print(f"   Re-introductions: {at['re_introductions']:,}")

# Current state
cs = data['current_state']
print(f"\n📍 CURRENT STATE")
print(f"   Findings present: {cs['findings_present']:,}")
print(f"   Net change: {cs['findings_present'] - 0:+,} (from 0 at start)")

# Shadow findings
sf = data['shadow_findings']
print(f"\n🌑 SHADOW FINDINGS (Zero-Day Exposure)")
print(f"   Total: {sf['total']:,}")
if 'avg_days' in sf:
    print(f"   Average window: {sf['avg_days']} days ({sf['avg_years']} years)")
    print(f"   Longest window: {sf['max_days']} days ({sf['max_years']} years)")
    
    print(f"\n   By Severity:")
    for sev in ['CRITICAL', 'HIGH', 'MEDIUM', 'LOW']:
        if sev in sf['by_severity']:
            s = sf['by_severity'][sev]
            print(f"     {sev:8s}: {s['count']:5,} findings, avg {s['avg_days']:4} days ({s['avg_years']} years)")

# Discovery lag
dl = data['discovery_lag']
print(f"\n⏰ DISCOVERY LAG (Introduced After CVE Published)")
print(f"   Total: {dl['total']:,}")
if 'avg_days' in dl:
    print(f"   Average lag: {dl['avg_days']} days ({dl['avg_years']} years)")
    print(f"   Longest lag: {dl['max_days']} days ({dl['max_years']} years)")

# Remediation
rem = data['remediation']
print(f"\n🔧 REMEDIATION VELOCITY")
print(f"   Total remediations: {rem['total_remediations']:,}")
if 'avg_days' in rem:
    print(f"   Average time to remediate: {rem['avg_days']} days ({rem['avg_years']} years)")

print("\n" + "=" * 80)
```

---

## 9a.3 Interpretation

### Shadow Findings Grade

Based on average shadow window for CRITICAL findings:

| Avg Critical Shadow Window | Grade | Risk Level |
|---------------------------|-------|------------|
| < 90 days | A | Low - Recent vulnerabilities |
| 90-180 days | B | Moderate - Some lag |
| 180-365 days | C | Elevated - Significant exposure |
| 1-2 years | D | High - Long undetectable period |
| > 2 years | F | Critical - Ancient vulnerabilities |

### Discovery Lag Grade

Based on average discovery lag for all findings:

| Avg Discovery Lag | Grade | Adoption Speed |
|------------------|-------|----------------|
| < 30 days | A | Excellent - Fast adoption |
| 30-90 days | B | Good - Reasonable lag |
| 90-180 days | C | Moderate - Slow adoption |
| 180-365 days | D | Poor - Very slow |
| > 1 year | F | Critical - Extremely slow |

### Remediation Velocity Grade

Based on average time to remediate:

| Avg Remediation Time | Grade | Velocity |
|---------------------|-------|----------|
| < 30 days | A | Excellent |
| 30-90 days | B | Good |
| 90-180 days | C | Moderate |
| 180-365 days | D | Poor |
| > 1 year | F | Critical failure |

---

## 9a.4 Key Patterns

| Pattern | What It Means | Action |
|---------|---------------|--------|
| High shadow findings | Using old packages with latent vulnerabilities | Audit dependency selection |
| High discovery lag | Slow to adopt new packages/patches | Improve update cadence |
| High re-introduction rate | Packages being added/removed repeatedly | Investigate churn causes |
| Long remediation time | Slow response to known vulnerabilities | Implement SLAs |
| Net positive findings | Introducing faster than remediating | Emergency intervention needed |

---

## Output

- Complete lifecycle metrics (all-time, averages, current state)
- Shadow findings analysis with severity breakdown
- Discovery lag analysis
- Remediation velocity metrics
- `/tmp/finding_lifecycle.json` for further analysis

---

## Next Step

Proceed to [Step 10: Package Growth Analysis](./10_package_growth_analysis.md)
