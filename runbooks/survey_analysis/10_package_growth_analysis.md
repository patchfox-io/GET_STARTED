# Step 10: Package Growth Analysis - The Attack Surface Story

**Purpose:** Understand how your attack surface (total packages) has evolved.

---

## Prerequisites

- Completed [Step 1: Data Collection](./01_data_collection.md)
- `/tmp/metrics.json` exists

---

## The Story Question

Package growth tells the attack surface story:
- Is the attack surface growing or shrinking?
- What percentage of packages have vulnerabilities?

---

## 10.1 Analyze Package Growth

```python
import json
from datetime import datetime, timedelta

with open('/tmp/metrics.json') as f:
    data = json.load(f)

metrics = data['data']['titlePage']['content']
metrics.sort(key=lambda x: x['commitDateTime'])

# Extract package count series
package_series = []
for m in metrics:
    dt = datetime.fromisoformat(m['commitDateTime'].replace('Z', '+00:00')).replace(tzinfo=None)
    packages = m.get('packages', 0) or 0
    package_series.append((dt, packages))

if package_series:
    first = package_series[0]
    last = package_series[-1]

    print(f"PACKAGE COUNT JOURNEY:")
    print(f"  Start: {first[1]:,} packages on {first[0].strftime('%Y-%m-%d')}")
    print(f"  Now:   {last[1]:,} packages as of {last[0].strftime('%Y-%m-%d')}")
    print(f"  Change: {last[1] - first[1]:+,} packages ({((last[1] - first[1])/first[1]*100) if first[1] else 0:+.1f}%)")

    # Growth velocity
    days_total = (last[0] - first[0]).days
    growth_per_month = (last[1] - first[1]) / (days_total / 30) if days_total > 0 else 0

    # Recent 90 days
    cutoff_90 = last[0] - timedelta(days=90)
    recent_start = next((p for p in package_series if p[0] >= cutoff_90), last)
    recent_growth = last[1] - recent_start[1]
    recent_days = (last[0] - recent_start[0]).days
    recent_per_month = recent_growth / (recent_days / 30) if recent_days > 0 else 0

    print(f"\n  GROWTH VELOCITY:")
    print(f"    Historical: {growth_per_month:+.1f} packages/month")
    print(f"    Recent (90d): {recent_per_month:+.1f} packages/month")

    if recent_per_month > growth_per_month * 1.5:
        print(f"    -> ACCELERATING growth - attack surface expanding faster")
    elif recent_per_month < growth_per_month * 0.5:
        print(f"    -> DECELERATING growth - growth slowing down")
    elif recent_per_month < 0:
        print(f"    -> CONTRACTING - attack surface shrinking (good)")
    else:
        print(f"    -> STABLE growth rate")
```

---

## 10.2 Vulnerable Package Percentage

```python
print(f"\n\nVULNERABLE PACKAGE PERCENTAGE JOURNEY:")
print(f"{'Date':<12} {'Packages':>10} {'With CVEs':>12} {'Vuln %':>8} {'Downlevel':>10} {'DL %':>8}")
print("-" * 70)

# Sample 10 points
sample_indices = [int(i * (len(metrics) - 1) / 9) for i in range(10)]

for i in sample_indices:
    m = metrics[i]
    date = m['commitDateTime'][:10]
    pkgs = m.get('packages', 0) or 0
    vuln = m.get('packagesWithFindings', 0) or 0
    dl = m.get('downlevelPackages', 0) or 0
    vuln_pct = (vuln / pkgs * 100) if pkgs > 0 else 0
    dl_pct = (dl / pkgs * 100) if pkgs > 0 else 0
    print(f"{date:<12} {pkgs:>10,} {vuln:>12,} {vuln_pct:>7.1f}% {dl:>10,} {dl_pct:>7.1f}%")

# Current state
last_m = metrics[-1]
pkgs = last_m.get('packages', 0) or 0
vuln = last_m.get('packagesWithFindings', 0) or 0
dl = last_m.get('downlevelPackages', 0) or 0

print(f"\n\nCURRENT ATTACK SURFACE STATE:")
print(f"  Total packages: {pkgs:,}")
print(f"  Packages with CVEs: {vuln:,} ({(vuln/pkgs*100) if pkgs else 0:.1f}%)")
print(f"  Downlevel packages: {dl:,} ({(dl/pkgs*100) if pkgs else 0:.1f}%)")
```

---

## 10.3 Grading

| Vulnerable % | Downlevel % | Grade |
|--------------|-------------|-------|
| < 5% | < 20% | A |
| < 10% | < 40% | B |
| < 15% | < 60% | C |
| < 25% | any | D |
| >= 25% | any | F |

---

## Output

- Package growth trajectory
- Growth velocity (historical vs recent)
- Vulnerable package percentage over time
- Downlevel percentage
- Attack surface grade (A-F)

---

## Next Step

Proceed to [Step 11: Synthesis](./11_synthesis.md)
