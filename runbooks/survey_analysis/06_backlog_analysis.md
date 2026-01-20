# Step 6: Backlog Analysis - The Remediation Velocity Story

**Purpose:** Tell the story of remediation velocity - are you closing findings, or are they aging in place?

---

## IMPORTANT: Known Granularity Bug

> **Warning:** There is a known bug ([GitHub Issue #9](https://github.com/patchfox-io/GET_STARTED/issues/9)) where backlog counts and `totalFindings` use different granularities. Backlog counts findings per-datasource (correct), while `totalFindings` deduplicates across datasources (incorrect).
>
> **If `backlog_total > totalFindings`, use the backlog value as the true finding count.**
>
> See [Common Pitfalls #13](./reference/common_pitfalls.md#13-backlog-exceeds-total-findings-granularity-mismatch) for details and workaround code.

---

## Prerequisites

- Completed [Step 1: Data Collection](./01_data_collection.md)
- `/tmp/metrics.json` exists
- [Temporal Utilities](./reference/temporal_utilities.md) available

---

## The Story Question

The backlog tells you not just how many vulnerabilities exist, but **how long they've been sitting there**:
- Are findings being closed promptly?
- Or aging in place indefinitely?

---

## 6.1 Tell the Backlog Story

```python
from datetime import datetime
from temporal_utilities import tell_metric_story, print_metric_story

def extract_backlog_series(metrics):
    """Extract backlog as time series, plus 90+ day percentage."""
    series = []
    pct_90_series = []

    for m in metrics:
        dt = datetime.fromisoformat(m['commitDateTime'].replace('Z', '+00:00')).replace(tzinfo=None)
        backlog_30_60 = m.get('findingsInBacklogBetweenThirtyAndSixtyDays', 0) or 0
        backlog_60_90 = m.get('findingsInBacklogBetweenSixtyAndNinetyDays', 0) or 0
        backlog_90_plus = m.get('findingsInBacklogOverNinetyDays', 0) or 0
        total = backlog_30_60 + backlog_60_90 + backlog_90_plus

        series.append((dt, total))

        if total > 0:
            pct_90_series.append((dt, (backlog_90_plus / total) * 100))
        else:
            pct_90_series.append((dt, 0))

    return series, pct_90_series

backlog_series, pct_90_series = extract_backlog_series(metrics)

# Tell the stories
backlog_story = tell_metric_story(backlog_series, "Total Backlog", lower_is_better=True)
print_metric_story(backlog_story)

pct_90_story = tell_metric_story(pct_90_series, "Backlog % in 90+ Days", lower_is_better=True)
print_metric_story(pct_90_story)
```

---

## 6.2 The Aging Story - How Findings Move Through Buckets

The real story is in how findings flow through the aging buckets:

```python
def analyze_backlog_flow(metrics):
    """Analyze how findings flow through backlog aging buckets."""
    print("\nBACKLOG AGING FLOW OVER TIME:")
    print(f"{'Date':<12} {'30-60d':>8} {'60-90d':>8} {'90+d':>8} {'Total':>8} {'%90+':>7}")
    print("-" * 55)

    for m in metrics[::max(1, len(metrics)//10)]:  # Sample 10 points
        date = m['commitDateTime'][:10]
        b30 = m.get('findingsInBacklogBetweenThirtyAndSixtyDays', 0) or 0
        b60 = m.get('findingsInBacklogBetweenSixtyAndNinetyDays', 0) or 0
        b90 = m.get('findingsInBacklogOverNinetyDays', 0) or 0
        total = b30 + b60 + b90
        pct_90 = (b90 / total * 100) if total > 0 else 0

        print(f"{date:<12} {b30:>8.0f} {b60:>8.0f} {b90:>8.0f} {total:>8.0f} {pct_90:>6.1f}%")

    # Story interpretation
    first = metrics[0]
    last = metrics[-1]

    first_total = ((first.get('findingsInBacklogBetweenThirtyAndSixtyDays', 0) or 0) +
                   (first.get('findingsInBacklogBetweenSixtyAndNinetyDays', 0) or 0) +
                   (first.get('findingsInBacklogOverNinetyDays', 0) or 0))
    last_total = ((last.get('findingsInBacklogBetweenThirtyAndSixtyDays', 0) or 0) +
                  (last.get('findingsInBacklogBetweenSixtyAndNinetyDays', 0) or 0) +
                  (last.get('findingsInBacklogOverNinetyDays', 0) or 0))

    first_90_pct = ((first.get('findingsInBacklogOverNinetyDays', 0) or 0) / first_total * 100) if first_total > 0 else 0
    last_90_pct = ((last.get('findingsInBacklogOverNinetyDays', 0) or 0) / last_total * 100) if last_total > 0 else 0

    print(f"\nBACKLOG FLOW STORY:")
    print(f"  Total backlog: {first_total:.0f} -> {last_total:.0f} ({last_total - first_total:+.0f})")
    print(f"  90+ day percentage: {first_90_pct:.1f}% -> {last_90_pct:.1f}%")

    if last_total < first_total and last_90_pct < first_90_pct:
        print(f"  -> EXCELLENT: Backlog shrinking AND aging improving")
    elif last_total < first_total:
        print(f"  -> GOOD: Backlog shrinking (but watch the 90+ bucket)")
    elif last_90_pct > first_90_pct + 10:
        print(f"  -> CONCERNING: Findings aging in place - remediation stalled")
    elif last_total > first_total * 1.5:
        print(f"  -> CRITICAL: Backlog growing significantly")
    else:
        print(f"  -> STABLE: Holding steady")

analyze_backlog_flow(metrics)
```

---

## 6.3 Key Backlog Story Patterns

| Pattern | What It Means | Action |
|---------|---------------|--------|
| Total declining, 90+ % declining | Active remediation, prioritizing old | Excellent - continue |
| Total stable, 90+ % growing | New findings resolved, old ones stuck | Tackle legacy backlog |
| Total growing, 90+ % stable | Adding faster than resolving | Increase remediation capacity |
| Volatile backlog | Reactive remediation | Need proactive SLAs |
| 90+ % > 70% | Backlog is aging in place | Emergency remediation focus |

---

## 6.4 Grading (Informed by Story)

| Grade | Criteria |
|-------|----------|
| A | 90+ % < 30% AND declining |
| B | 90+ % 30-50% OR declining from higher |
| C | 90+ % 50-70% stable |
| D | 90+ % 50-70% growing |
| F | 90+ % > 70% |

---

## Output

- Total backlog story
- 90+ day percentage story
- Backlog flow analysis
- Backlog grade (A-F)

---

## Next Step

Proceed to [Step 7: Edit Analysis](./07_edit_analysis.md)
