# Step 9: CVE Age Analysis - The Technical Debt Story

**Purpose:** Understand how old your unpatched vulnerabilities are - this is your technical debt.

---

## Prerequisites

- Completed [Step 1: Data Collection](./01_data_collection.md)
- `/tmp/findings.json` exists

---

## The Story Question

CVE age tells you about technical debt:
- How long have vulnerabilities been sitting unpatched?
- Are critical CVEs being prioritized?

---

## 9.1 Analyze CVE Ages

```python
import json
from datetime import datetime
from collections import defaultdict

with open('/tmp/findings.json') as f:
    data = json.load(f)

findings = data['data']['titlePage']['content']
print(f"Analyzing {len(findings)} unique CVEs...\n")

now = datetime.now()
cve_ages = []
severity_ages = defaultdict(list)

for f in findings:
    published = f.get('publishedAt')
    severity = f.get('severity', 'UNKNOWN')
    cve_id = f.get('identifier', 'UNKNOWN')

    if published:
        try:
            pub_date = datetime.fromisoformat(published.replace('Z', '+00:00')).replace(tzinfo=None)
            age_days = (now - pub_date).days
            cve_ages.append({'cve': cve_id, 'age_days': age_days, 'severity': severity, 'published': pub_date})
            severity_ages[severity].append(age_days)
        except:
            pass

if cve_ages:
    ages = [c['age_days'] for c in cve_ages]
    avg_age = sum(ages) / len(ages)
    cve_ages.sort(key=lambda x: x['age_days'], reverse=True)

    print("CVE AGE DISTRIBUTION:")
    print(f"{'Age Bucket':<20} {'Count':>8} {'%':>8}")
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
        count = sum(1 for a in ages if low <= a < high)
        pct = count / len(ages) * 100
        print(f"{label:<20} {count:>8} {pct:>7.1f}%")

    print(f"\nAVERAGE CVE AGE: {avg_age:.0f} days ({avg_age/365:.1f} years)")

    # By severity
    print("\n\nCVE AGE BY SEVERITY:")
    print(f"{'Severity':<12} {'Count':>8} {'Avg Age':>12} {'Oldest':>12}")
    print("-" * 50)

    for severity in ['CRITICAL', 'HIGH', 'MEDIUM', 'LOW']:
        if severity in severity_ages:
            ages_sev = severity_ages[severity]
            avg = sum(ages_sev) / len(ages_sev)
            oldest = max(ages_sev)
            print(f"{severity:<12} {len(ages_sev):>8} {avg:>10.0f}d {oldest:>10}d")

    # Oldest CVEs
    print("\n\nOLDEST UNPATCHED CVES (TECHNICAL DEBT):")
    print(f"{'CVE':<20} {'Severity':<10} {'Age (days)':<12} {'Published'}")
    print("-" * 70)

    for c in cve_ages[:10]:
        print(f"{c['cve']:<20} {c['severity']:<10} {c['age_days']:<12} {c['published'].strftime('%Y-%m-%d')}")
```

---

## 9.2 Grading

Based on oldest CRITICAL CVE age:

| Oldest Critical CVE | Grade |
|---------------------|-------|
| < 90 days | A |
| 90-180 days | B |
| 180-365 days | C |
| 1-2 years | D |
| > 2 years | F |

---

## 9.3 Key Patterns

| Pattern | What It Means | Action |
|---------|---------------|--------|
| Most CVEs < 90 days | Good remediation velocity | Continue |
| Critical CVEs older than others | Wrong prioritization | Focus on severity |
| CVEs > 2 years | Significant technical debt | Emergency remediation plan |
| Average age increasing | Falling behind | Increase remediation capacity |

---

## Output

- CVE age distribution
- Age by severity breakdown
- Top 10 oldest CVEs
- CVE age grade (A-F)

---

## Next Step

Proceed to [Step 10: Package Growth Analysis](./10_package_growth_analysis.md)
