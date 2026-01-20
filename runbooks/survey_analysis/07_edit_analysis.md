# Step 7: Edit Type Analysis - The Patching Behavior Story

**Purpose:** Understand **how** you're patching, not just **that** you're patching.

---

## Prerequisites

- Completed [Step 1: Data Collection](./01_data_collection.md)
- `/tmp/edits.json` exists

---

## The Story Question

Edits tell the story of patching behavior:
- Are you making meaningful changes?
- Or churning without impact?

---

## 7.1 Tell the Edit Behavior Story

```python
import json
from collections import defaultdict

with open('/tmp/edits.json') as f:
    data = json.load(f)
edits = data['data']['titlePage']['content']

# Sort edits by date
edits.sort(key=lambda x: x.get('commitDateTime', ''))

# Group edits by month for trend analysis
by_month = defaultdict(lambda: {'total': 0, 'same': 0, 'different': 0,
                                 'create': 0, 'update': 0, 'delete': 0})

for e in edits:
    month = e.get('commitDateTime', '')[:7]  # YYYY-MM
    by_month[month]['total'] += 1
    if e.get('sameEdit'):
        by_month[month]['same'] += 1
    else:
        by_month[month]['different'] += 1

    edit_type = e.get('editType', 'UNKNOWN')
    if edit_type == 'CREATE':
        by_month[month]['create'] += 1
    elif edit_type == 'UPDATE':
        by_month[month]['update'] += 1
    elif edit_type == 'DELETE':
        by_month[month]['delete'] += 1

print("PATCHING BEHAVIOR OVER TIME:")
print(f"{'Month':<10} {'Total':>7} {'Same%':>7} {'Diff%':>7} {'CREATE':>8} {'UPDATE':>8} {'DELETE':>8}")
print("-" * 65)

months = sorted(by_month.keys())
for month in months:
    m = by_month[month]
    same_pct = (m['same'] / m['total'] * 100) if m['total'] > 0 else 0
    diff_pct = (m['different'] / m['total'] * 100) if m['total'] > 0 else 0
    print(f"{month:<10} {m['total']:>7} {same_pct:>6.1f}% {diff_pct:>6.1f}% {m['create']:>8} {m['update']:>8} {m['delete']:>8}")

# Overall story
total = len(edits)
same_total = sum(1 for e in edits if e.get('sameEdit'))
create_total = sum(1 for e in edits if e.get('editType') == 'CREATE')
delete_total = sum(1 for e in edits if e.get('editType') == 'DELETE')

print(f"\nPATCHING BEHAVIOR STORY:")
print(f"  Total edits: {total:,}")
print(f"  Same-version edits: {same_total:,} ({same_total/total*100:.1f}%)")
print(f"  Different-version edits: {total - same_total:,} ({(total-same_total)/total*100:.1f}%)")
print(f"  CREATE vs DELETE ratio: {create_total:,} vs {delete_total:,}")

# Growth story
if create_total > delete_total * 1.5:
    print(f"\n  GROWTH STORY: Adding packages faster than removing ({create_total} vs {delete_total})")
    print(f"    -> Attack surface expanding")
elif delete_total > create_total:
    print(f"\n  GROWTH STORY: Removing more than adding ({delete_total} vs {create_total})")
    print(f"    -> Attack surface contracting (good)")
else:
    print(f"\n  GROWTH STORY: Balanced adds/removes")
    print(f"    -> Controlled growth")
```

---

## 7.2 Key Edit Story Patterns

| Pattern | What It Means | Action |
|---------|---------------|--------|
| Same% declining over time | Learning - patches getting more targeted | Continue improvement |
| Same% increasing over time | Regression - more churn, less impact | Review patch selection criteria |
| CREATE >> DELETE | Uncontrolled growth | Implement removal discipline |
| DELETE >> CREATE | Active pruning | Good - may indicate cleanup initiative |
| High volume + high same% | Busy work | Stop and reassess what's being patched |
| Low volume + high different% | Selective, meaningful patches | Ideal state |

---

## 7.3 Grading (Informed by Story)

| Grade | Criteria |
|-------|----------|
| A | Same% < 30% AND declining trend AND CREATE ~ DELETE |
| B | Same% 30-50% OR Same% < 30% but CREATE > DELETE |
| C | Same% 50-66% with stable trend |
| D | Same% > 66% OR Same% increasing significantly |
| F | Same% > 80% OR CREATE >> DELETE with high volume |

---

## Key Insight

**Same-version edits have ZERO security benefit** - they can't fix vulnerabilities. High same% means lots of activity without addressing actual security issues.

---

## Output

- Monthly edit breakdown
- Same vs different version percentages
- CREATE/UPDATE/DELETE ratios
- Edit behavior grade (A-F)

---

## Next Step

Proceed to [Step 8: Package Family Analysis](./08_package_family_analysis.md)
