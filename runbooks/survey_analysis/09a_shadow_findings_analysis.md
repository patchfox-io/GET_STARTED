# Step 9a: Shadow Findings Analysis - The Zero-Day Window

**Purpose:** Identify vulnerabilities that existed in your codebase BEFORE they were published to NVD - your "shadow" exposure period.

---

## Prerequisites

- Completed [Step 1: Data Collection](./01_data_collection.md)
- Active dataset with analyzed findings

---

## The Story Question

Shadow findings reveal your most dangerous exposure:
- How long were vulnerabilities present before they were even detectable?
- What's your average "zero-day window" - the time between introducing a vulnerable package and the CVE being published?
- Which packages introduced the most shadow risk?

A "shadow finding" is a vulnerability that exists in your dataset during the period between when the vulnerable package was committed (`commit_date_time`) and when the CVE was published to NVD (`published_at`). During this window, the vulnerability was present but undetectable by scanning tools.

---

## 9a.1 Extract Shadow Findings Data

```python
import json
import requests
from datetime import datetime
from collections import defaultdict

# Query for packages with findings, including commit history
url = "http://localhost:1702/api/v1/db/package/query"
params = {
    'size': 10000,
    'select': 'purl,name,version,type,commitDateTime,findings.identifier,findings.severity,findings.publishedAt'
}

response = requests.get(url, params=params)
data = response.json()
packages = data['data']['titlePage']['content']

print(f"Analyzing {len(packages)} packages for shadow findings...\n")

# Build shadow findings dataset
shadow_findings = []

for pkg in packages:
    commit_dt = pkg.get('commitDateTime')
    findings = pkg.get('findings', [])
    
    if not commit_dt or not findings:
        continue
    
    try:
        commit_date = datetime.fromisoformat(commit_dt.replace('Z', '+00:00')).replace(tzinfo=None)
    except:
        continue
    
    for finding in findings:
        published = finding.get('publishedAt')
        if not published:
            continue
        
        try:
            published_date = datetime.fromisoformat(published.replace('Z', '+00:00')).replace(tzinfo=None)
        except:
            continue
        
        # Shadow finding: package committed BEFORE CVE was published
        if commit_date < published_date:
            shadow_days = (published_date - commit_date).days
            shadow_findings.append({
                'package': f"{pkg.get('type')}:{pkg.get('name')}@{pkg.get('version')}",
                'purl': pkg.get('purl'),
                'cve': finding.get('identifier'),
                'severity': finding.get('severity', 'UNKNOWN'),
                'commit_date': commit_date,
                'published_date': published_date,
                'shadow_days': shadow_days
            })

print(f"Found {len(shadow_findings)} shadow findings\n")

# Save for further analysis
with open('/tmp/shadow_findings.json', 'w') as f:
    json.dump(shadow_findings, f, indent=2, default=str)
```

---

## 9a.2 Analyze Shadow Exposure Windows

```python
if not shadow_findings:
    print("No shadow findings detected - all vulnerabilities were published before package introduction")
else:
    shadow_days = [sf['shadow_days'] for sf in shadow_findings]
    avg_shadow = sum(shadow_days) / len(shadow_days)
    max_shadow = max(shadow_days)
    
    print("SHADOW EXPOSURE DISTRIBUTION:")
    print(f"{'Window':<20} {'Count':>8} {'%':>8}")
    print("-" * 40)
    
    buckets = [
        (0, 30, "< 30 days"),
        (30, 90, "30-90 days"),
        (90, 180, "90-180 days"),
        (180, 365, "6mo - 1yr"),
        (365, 730, "1-2 years"),
        (730, 1825, "2-5 years"),
        (1825, float('inf'), "5+ years")
    ]
    
    for low, high, label in buckets:
        count = sum(1 for d in shadow_days if low <= d < high)
        pct = count / len(shadow_days) * 100 if shadow_days else 0
        print(f"{label:<20} {count:>8} {pct:>7.1f}%")
    
    print(f"\nAVERAGE SHADOW WINDOW: {avg_shadow:.0f} days ({avg_shadow/365:.1f} years)")
    print(f"LONGEST SHADOW WINDOW: {max_shadow} days ({max_shadow/365:.1f} years)")
```

---

## 9a.3 Shadow Findings by Severity

```python
from collections import defaultdict

severity_shadows = defaultdict(list)
for sf in shadow_findings:
    severity_shadows[sf['severity']].append(sf['shadow_days'])

print("\n\nSHADOW EXPOSURE BY SEVERITY:")
print(f"{'Severity':<12} {'Count':>8} {'Avg Window':>12} {'Max Window':>12}")
print("-" * 50)

for severity in ['CRITICAL', 'HIGH', 'MEDIUM', 'LOW']:
    if severity in severity_shadows:
        days = severity_shadows[severity]
        avg = sum(days) / len(days)
        max_days = max(days)
        print(f"{severity:<12} {len(days):>8} {avg:>10.0f}d {max_days:>10}d")
```

---

## 9a.4 Worst Shadow Offenders

```python
# Sort by shadow window length
shadow_findings.sort(key=lambda x: x['shadow_days'], reverse=True)

print("\n\nLONGEST SHADOW EXPOSURES:")
print(f"{'Package':<40} {'CVE':<20} {'Severity':<10} {'Shadow Days':<12} {'Committed':<12} {'Published'}")
print("-" * 130)

for sf in shadow_findings[:20]:
    pkg_short = sf['package'][:38] if len(sf['package']) > 38 else sf['package']
    print(f"{pkg_short:<40} {sf['cve']:<20} {sf['severity']:<10} {sf['shadow_days']:<12} "
          f"{sf['commit_date'].strftime('%Y-%m-%d'):<12} {sf['published_date'].strftime('%Y-%m-%d')}")
```

---

## 9a.5 Package-Level Shadow Risk

```python
# Which packages introduced the most shadow risk?
package_shadows = defaultdict(list)
for sf in shadow_findings:
    package_shadows[sf['package']].append(sf)

package_risk = []
for pkg, findings in package_shadows.items():
    total_shadow_days = sum(f['shadow_days'] for f in findings)
    avg_shadow = total_shadow_days / len(findings)
    critical_count = sum(1 for f in findings if f['severity'] == 'CRITICAL')
    high_count = sum(1 for f in findings if f['severity'] == 'HIGH')
    
    package_risk.append({
        'package': pkg,
        'finding_count': len(findings),
        'total_shadow_days': total_shadow_days,
        'avg_shadow_days': avg_shadow,
        'critical': critical_count,
        'high': high_count
    })

package_risk.sort(key=lambda x: x['total_shadow_days'], reverse=True)

print("\n\nPACKAGES WITH HIGHEST SHADOW RISK:")
print(f"{'Package':<40} {'Findings':>10} {'Total Shadow':>14} {'Avg Shadow':>12} {'Critical':>10} {'High':>10}")
print("-" * 110)

for pr in package_risk[:15]:
    pkg_short = pr['package'][:38] if len(pr['package']) > 38 else pr['package']
    print(f"{pkg_short:<40} {pr['finding_count']:>10} {pr['total_shadow_days']:>12}d {pr['avg_shadow_days']:>10.0f}d "
          f"{pr['critical']:>10} {pr['high']:>10}")
```

---

## 9a.6 Grading

Based on average shadow window for CRITICAL findings:

| Avg Critical Shadow Window | Grade | Risk Level |
|---------------------------|-------|------------|
| < 90 days | A | Low - Recent vulnerabilities |
| 90-180 days | B | Moderate - Some lag |
| 180-365 days | C | Elevated - Significant exposure |
| 1-2 years | D | High - Long undetectable period |
| > 2 years | F | Critical - Ancient vulnerabilities |

---

## 9a.7 Key Patterns

| Pattern | What It Means | Action |
|---------|---------------|--------|
| Most shadows < 90 days | Using recent packages | Good - low zero-day risk |
| Critical shadows > 1 year | Old vulnerable packages | Audit dependency selection process |
| Few/no shadow findings | Vulnerabilities found quickly OR using very old packages | Check if packages predate CVE publication |
| Long shadows on popular packages | Slow to adopt patches | Improve update cadence |
| Shadows increasing over time | Dependency hygiene degrading | Implement dependency review |

---

## 9a.8 The Shadow Risk Story

Shadow findings tell you about your **undetectable exposure window**:

1. **Zero-Day Risk**: Packages with long shadow windows were vulnerable for extended periods before any scanner could detect them
2. **Dependency Age**: Long shadows often indicate using old package versions at time of commit
3. **Update Lag**: If you're introducing packages that already have published CVEs, you're behind
4. **Supply Chain Risk**: Shadow findings reveal how long you're exposed to vulnerabilities that attackers might know about but scanners can't detect yet

**Critical Insight**: A package committed in 2015 with a CVE published in 2020 means you had 5 years of undetectable exposure. This is your true security debt.

---

## Output

- Shadow findings count and distribution
- Average shadow window by severity
- Top 20 longest shadow exposures
- Packages with highest cumulative shadow risk
- Shadow risk grade (A-F)
- `/tmp/shadow_findings.json` for further analysis

---

## Next Step

Proceed to [Step 10: Package Growth Analysis](./10_package_growth_analysis.md)
