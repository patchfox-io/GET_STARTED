# Step 4: RPS Analysis - The Version Fragmentation Story

**Purpose:** Tell the story of version consolidation - are you converging on standard versions, or fragmenting into chaos?

---

## Prerequisites

- Completed [Step 1: Data Collection](./01_data_collection.md)
- `/tmp/metrics.json` exists
- [Temporal Utilities](./reference/temporal_utilities.md) available

---

## The Story Question

RPS (Redundant Package Score) tells us about version diversity across your codebase:
- Are we consolidating on standard versions?
- Or is fragmentation growing?

---

## 4.1 Tell the RPS Story

```python
from temporal_utilities import parse_metrics, extract_time_series, tell_metric_story, print_metric_story

metrics = parse_metrics('/tmp/metrics.json')
rps_series = extract_time_series(metrics, 'rpsScore')

# RPS: LOWER is better (less fragmentation)
rps_story = tell_metric_story(rps_series, "Redundant Package Score (RPS)", lower_is_better=True)
print_metric_story(rps_story)
```

---

## 4.2 RPS Story Interpretation

| Question | Where to Look |
|----------|---------------|
| "Is version fragmentation getting better or worse?" | `story_arc.journey.narrative` |
| "When did fragmentation patterns change?" | `turning_points` and `chapters` |
| "Are consolidation efforts working?" | `velocity` - is recent velocity negative (improving)? |
| "Any sudden fragmentation events?" | `notable_events` - spikes indicate bulk additions of new versions |

---

## 4.3 Key RPS Story Patterns

| Pattern | What It Means | Action |
|---------|---------------|--------|
| Steady DECLINE over time | Consolidation efforts working | Continue approach |
| GROWTH chapters | Periods of uncontrolled additions | Investigate what drove new versions |
| Spike events | Bulk imports or major version bumps | Check if intentional |
| VOLATILE with no trend | No consolidation strategy | Implement version pinning policy |
| Recent velocity reversal (was declining, now growing) | Consolidation efforts stalled | Re-engage standardization |

---

## 4.4 Grading (Informed by Story)

| Grade | Criteria |
|-------|----------|
| A | RPS < 5 AND declining or stable |
| B | RPS 5-15 AND declining, OR RPS < 5 but growing slowly |
| C | RPS 5-15 stable, OR 15-25 declining |
| D | RPS 15-25 stable or growing, OR any RPS with accelerating growth |
| F | RPS > 25, OR any RPS with rapid growth trend |

---

## Output

- RPS story narrative
- Chapter breakdown showing consolidation/fragmentation phases
- Key events (spikes in fragmentation)
- RPS grade (A-F)

---

## Next Step

Proceed to [Step 5: Findings Analysis](./05_findings_analysis.md)
