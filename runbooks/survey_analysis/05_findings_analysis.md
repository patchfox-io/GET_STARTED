# Step 5: Findings Analysis - The Vulnerability Story

**Purpose:** Tell the story of how your vulnerability exposure has evolved over time.

---

## IMPORTANT: Known Granularity Bug

> **Warning:** There is a known bug ([GitHub Issue #9](https://github.com/patchfox-io/GET_STARTED/issues/9)) where `totalFindings` (and `criticalFindings`, `highFindings`, etc.) may be **undercounted** because they deduplicate across datasources.
>
> **If backlog totals exceed findings totals, use the LARGER value as the true count.**
>
> For accurate findings counts, compare against the sum of backlog buckets:
> ```python
> backlog_total = (findingsInBacklogBetweenThirtyAndSixtyDays +
>                  findingsInBacklogBetweenSixtyAndNinetyDays +
>                  findingsInBacklogOverNinetyDays)
> true_findings = max(totalFindings, backlog_total)
> ```
>
> See [Common Pitfalls #13](./reference/common_pitfalls.md#13-backlog-exceeds-total-findings-granularity-mismatch) for details.

---

## Prerequisites

- Completed [Step 1: Data Collection](./01_data_collection.md)
- `/tmp/metrics.json` exists
- [Temporal Utilities](./reference/temporal_utilities.md) available

---

## The Story Question

This is often the most important story:
- How has vulnerability exposure evolved?
- Is remediation outpacing discovery?

---

## 5.1 Tell the Findings Story

```python
from temporal_utilities import parse_metrics, extract_time_series, tell_metric_story, print_metric_story

metrics = parse_metrics('/tmp/metrics.json')

# Total findings story
findings_series = extract_time_series(metrics, 'totalFindings')
findings_story = tell_metric_story(findings_series, "Total Findings", lower_is_better=True)
print_metric_story(findings_story)

# Critical findings story (separate - they have their own story)
critical_series = extract_time_series(metrics, 'criticalFindings')
critical_story = tell_metric_story(critical_series, "Critical Findings", lower_is_better=True)
print_metric_story(critical_story)
```

---

## 5.2 Discovery vs Remediation: The Two Forces

Findings change due to two opposing forces:
1. **Discovery** - new CVEs found in existing packages (increases findings)
2. **Remediation** - fixing/removing vulnerable packages (decreases findings)

```python
from datetime import datetime

def analyze_discovery_vs_remediation(metrics):
    """Separate the discovery and remediation components."""
    results = []

    for i in range(1, len(metrics)):
        prev = metrics[i-1]
        curr = metrics[i]

        findings_change = curr['totalFindings'] - prev['totalFindings']
        packages_change = curr['packages'] - prev['packages']

        if findings_change < 0:
            remediation_estimate = abs(findings_change)
            discovery_estimate = 0
        else:
            discovery_estimate = findings_change
            remediation_estimate = 0

        results.append({
            'findings_change': findings_change,
            'packages_change': packages_change,
            'discovery_estimate': discovery_estimate,
            'remediation_estimate': remediation_estimate
        })

    total_discovery = sum(r['discovery_estimate'] for r in results)
    total_remediation = sum(r['remediation_estimate'] for r in results)

    print(f"\nDISCOVERY vs REMEDIATION STORY:")
    print(f"  Estimated new discoveries: +{total_discovery}")
    print(f"  Estimated remediations: -{total_remediation}")
    print(f"  Net effect: {total_discovery - total_remediation:+d}")

    if total_remediation > total_discovery:
        print(f"  -> Remediation outpacing discovery (GOOD)")
    elif total_discovery > total_remediation * 1.5:
        print(f"  -> Discovery significantly outpacing remediation (CONCERNING)")
    else:
        print(f"  -> Roughly balanced (STABLE)")

analyze_discovery_vs_remediation(metrics)
```

---

## 5.3 Severity Trends

Track how the severity composition has evolved:

```python
def analyze_severity_story(metrics):
    """Show how the severity composition has evolved."""
    print("\nSEVERITY COMPOSITION OVER TIME:")
    print(f"{'Date':<12} {'CRIT':>6} {'HIGH':>6} {'MED':>6} {'LOW':>6} {'TOTAL':>7} {'%CRIT+HIGH':>10}")
    print("-" * 60)

    for m in metrics[::max(1, len(metrics)//10)]:  # Sample 10 points
        date = m['commitDateTime'][:10]
        crit = m.get('criticalFindings', 0)
        high = m.get('highFindings', 0)
        med = m.get('mediumFindings', 0)
        low = m.get('lowFindings', 0)
        total = m.get('totalFindings', 0)

        crit_high_pct = ((crit + high) / total * 100) if total > 0 else 0
        print(f"{date:<12} {crit:>6} {high:>6} {med:>6} {low:>6} {total:>7} {crit_high_pct:>9.1f}%")

analyze_severity_story(metrics)
```

---

## 5.4 Key Findings Story Patterns

| Pattern | What It Means | Action |
|---------|---------------|--------|
| Steady DECLINE | Active remediation working | Continue and celebrate |
| GROWTH despite patches | Discovery outpacing remediation | Increase remediation velocity |
| Spike followed by decline | Major discovery event, then cleanup | Good response pattern |
| Critical findings GROWTH while total stable | Prioritizing wrong things | Shift focus to severity |
| VOLATILE with no trend | Reactive, not proactive | Need systematic approach |

---

## 5.5 Grading (Informed by Story)

| Grade | Criteria |
|-------|----------|
| A | Findings declining AND critical findings declining faster |
| B | Findings stable or slowly declining |
| C | Findings slowly growing but critical stable/declining |
| D | Findings growing AND critical growing |
| F | Rapid findings growth OR critical findings exploding |

---

## Output

- Total findings story
- Critical findings story (separate)
- Discovery vs remediation analysis
- Severity composition trend
- Findings grade (A-F)

---

## Next Step

Proceed to [Step 6: Backlog Analysis](./06_backlog_analysis.md)
