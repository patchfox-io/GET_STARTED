# Step 2: Dataset Composition Analysis

**Purpose:** Understand the makeup of your dataset by ecosystem type.

---

## Prerequisites

- Completed [Step 1: Data Collection](./01_data_collection.md)
- `/tmp/dmc.json` exists

---

## 2.1 Analyze Composition by Type

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
    by_type[ds_type]['packages'] += r.get('packages', 0) or 0
    by_type[ds_type]['findings'] += r.get('totalFindings', 0) or 0
    by_type[ds_type]['critical'] += r.get('criticalFindings', 0) or 0
    by_type[ds_type]['high'] += r.get('highFindings', 0) or 0

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

---

## 2.2 Interpret Composition

Look for disproportionate relationships:

| Pattern | Meaning |
|---------|---------|
| Type has X% datasources but Y% packages (Y >> X) | Heavy dependency footprint per project (e.g., npm) |
| Type has X% packages but Y% findings (Y >> X) | Higher vulnerability density (riskier ecosystem) |
| Type has X% findings but Y% critical+high (Y >> X) | More severe vulnerabilities in that ecosystem |

---

## Red Flags

- **One type dominates findings despite small datasource count** → concentrated risk
- **High finding % with low package %** → vulnerable ecosystem, prioritize remediation
- **Ecosystem with disproportionate critical+high** → urgent attention needed

---

## Example Insight

> "npm is only 1.6% of datasources but 17% of packages - each npm project brings 10x more dependencies. pypi has 6x the vulnerability density of the dataset average."

---

## Output

A table showing:
- Datasource count and percentage by ecosystem
- Package count and percentage by ecosystem
- Finding count and percentage by ecosystem
- Identification of high-risk ecosystems

---

## Next Step

Proceed to [Step 3: PES Analysis](./03_pes_analysis.md)
