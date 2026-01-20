# Step 3: PES Analysis - The Patch Effectiveness Story

**Purpose:** Tell the story of whether your patching efforts are actually improving security.

---

## Prerequisites

- Completed [Step 1: Data Collection](./01_data_collection.md)
- `/tmp/metrics.json` exists
- [Temporal Utilities](./reference/temporal_utilities.md) available

---

## The Story Question

This isn't about "what's the average PES?" - it's about:
- How has patch effectiveness evolved?
- What does that tell us about our patching strategy?

---

## 3.1 Tell the PES Story

```python
# Using the temporal utilities
from temporal_utilities import parse_metrics, extract_time_series, tell_metric_story, print_metric_story

metrics = parse_metrics('/tmp/metrics.json')
pes_series = extract_time_series(metrics, 'patchEfficacyScore')

# Get the full story - PES is special: HIGHER is better
pes_story = tell_metric_story(pes_series, "Patch Efficacy Score (PES)", lower_is_better=False)
print_metric_story(pes_story)
```

---

## 3.2 PES Story Interpretation

The PES story answers these questions:

| Question | Where to Look |
|----------|---------------|
| "Are patches getting more effective over time?" | `story_arc.journey.narrative` and `velocity.velocity_story` |
| "When did patch effectiveness change?" | `turning_points` - look for peaks and troughs |
| "What's the recent trend vs historical?" | `period_comparison` and `velocity.recent_vs_historical` |
| "Were there any dramatic shifts?" | `notable_events` - spikes or drops in PES |
| "What phases has PES gone through?" | `chapters` - periods of improvement, decline, stability |

---

## 3.3 Key PES Story Patterns

| Pattern | What It Means | Investigation |
|---------|---------------|---------------|
| Chapter of DECLINE followed by GROWTH | Something changed - policy? tooling? team? | Check what happened at the inflection point |
| Recent velocity reversal (was improving, now degrading) | Recent changes are hurting effectiveness | Review recent patch decisions |
| Spike in PES on specific date | A particularly effective patch batch | What made it effective? Replicate it |
| Consistent STABILITY near zero | Patches neither helping nor hurting | Patches may not be targeting real issues |
| VOLATILE chapters | Inconsistent patching approach | Need standardization |

---

## 3.4 Grading (Informed by Story)

The grade considers both current state AND trajectory:

| Grade | Criteria |
|-------|----------|
| A | Current PES > 1 AND improving or stable |
| B | Current PES 0-1 AND improving, OR PES > 1 but declining |
| C | Current PES -1 to 0, OR positive but rapidly declining |
| D | Current PES -1 to -2, OR near-zero with no improvement trend |
| F | Current PES < -2, OR any negative PES with worsening trend |

---

## Example Output

```
======================================================================
  THE STORY OF PATCH EFFICACY SCORE (PES)
======================================================================

  SUMMARY:
  Patch Efficacy Score (PES) started at -0.5 on 2024-01-15 and increased
  to 1.2 over 365 days (+340.0%). This is good. Recent improvement -
  was degrading or stable, now improving.

  CHAPTERS:
    * Chapter 1 (Jan 2024 - Apr 2024): DECLINE phase, -0.5 -> -1.8
    * Chapter 2 (Apr 2024 - Aug 2024): STABILITY phase, -1.8 -> -1.5
    * Chapter 3 (Aug 2024 - Jan 2025): GROWTH phase, -1.5 -> 1.2

  RECENT TREND:
    Last 30 days averaged 1.1 vs 0.3 prior (+266.7%)
    Velocity: Currently changing at +0.8/month vs historical +0.15/month

  KEY EVENTS TO INVESTIGATE:
    2024-08-12: Bottomed at -1.8, then began recovering
       -> What intervention worked? Can we replicate it?
    2024-11-03: Sudden increase of 0.9 (+150.0%)
       -> What happened on 2024-11-03? Check commits, releases, team changes.

  JOURNEY SUMMARY:
    Start:   Started at -0.5 on 2024-01-15
    Current: Now at 1.2 as of 2025-01-15
    Change:  +1.7 (+340.0%)
```

---

## Output

- PES story narrative
- Chapter breakdown
- Key turning points to investigate
- PES grade (A-F)

---

## Next Step

Proceed to [Step 4: RPS Analysis](./04_rps_analysis.md)
