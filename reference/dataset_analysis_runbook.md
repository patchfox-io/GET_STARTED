# PatchFox Dataset Security Analysis Runbook

## Purpose

This runbook provides a systematic methodology for analyzing the security posture and patching effectiveness of any PatchFox dataset. Follow this step-by-step guide to produce a comprehensive assessment.

---

## Table of Contents

1. [Overview & Objectives](#1-overview--objectives)
2. [Prerequisites](#2-prerequisites)
3. [Data Sources](#3-data-sources)
4. [Key Metrics & Definitions](#4-key-metrics--definitions)
5. [Analysis Methodology](#5-analysis-methodology)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interpretation Guidelines](#7-interpretation-guidelines)
8. [Report Structure Template](#8-report-structure-template)
9. [Example Analysis Script](#9-example-analysis-script)
10. [Checklist](#10-checklist)
11. [Appendix: Quick Reference](#appendix-quick-reference)

---

## 1. Overview & Objectives

### Complete Analysis Methodology

**⚠️ MANDATORY: This document is an overview only. When performing dataset analysis, you MUST follow ALL steps in ALL detailed runbooks listed below. ⚠️**

The complete step-by-step methodology is located in:

**`../runbooks/survey_analysis/`**

#### Detailed Runbook Manifest

**YOU MUST EXECUTE EVERY STEP IN EACH OF THESE RUNBOOKS:**

1. **[Data Collection](../runbooks/survey_analysis/01_data_collection.md)** - Gather all required dataset metrics and entities
2. **[Composition Analysis](../runbooks/survey_analysis/02_composition_analysis.md)** - Breakdown by ecosystem (npm, maven, pypi, etc.)
3. **[PES Analysis](../runbooks/survey_analysis/03_pes_analysis.md)** - Patch Efficacy Score deep dive
4. **[RPS Analysis](../runbooks/survey_analysis/04_rps_analysis.md)** - Version fragmentation assessment
5. **[Findings Analysis](../runbooks/survey_analysis/05_findings_analysis.md)** - Vulnerability trends and severity breakdown
6. **[Backlog Analysis](../runbooks/survey_analysis/06_backlog_analysis.md)** - Aging analysis and remediation velocity
7. **[Edit Analysis](../runbooks/survey_analysis/07_edit_analysis.md)** - Patch behavior and edit type breakdown
8. **[Package Family Analysis](../runbooks/survey_analysis/08_package_family_analysis.md)** - Consolidation opportunities
9. **[Package Deep Dive](../runbooks/survey_analysis/08a_package_deep_dive.md)** - Individual package investigation
10. **[CVE Age Analysis](../runbooks/survey_analysis/09_cve_age_analysis.md)** - How old are unaddressed vulnerabilities
11. **[Finding Lifecycle Analysis](../runbooks/survey_analysis/09a_finding_lifecycle_analysis.md)** - ⚠️ CRITICAL: Complete finding lifecycle tracking (shadow findings, discovery lag, remediation velocity, re-introductions)
12. **[Package Growth Analysis](../runbooks/survey_analysis/10_package_growth_analysis.md)** - Attack surface expansion patterns
13. **[Synthesis](../runbooks/survey_analysis/11_synthesis.md)** - Integrate all findings into coherent narrative
14. **[Report Generation](../runbooks/survey_analysis/12_report_generation.md)** - Final report assembly

**Start here:** [Survey Analysis README](../runbooks/survey_analysis/README.md)

**Skipping any runbook = incomplete analysis. Follow the complete methodology.**

---

### What This Analysis Reveals

A comprehensive dataset analysis answers:
- **Are patches effective?** (PES - Patch Efficacy Score)
- **Are vulnerabilities decreasing?** (Findings trend)
- **Are patches addressing vulnerabilities?** (Backlog analysis)
- **What types of patches are being made?** (Edit type breakdown)
- **How old are unaddressed vulnerabilities?** (CVE age analysis)
- **How long were vulnerabilities present BEFORE CVE publication?** (Shadow findings - zero-day window)
- **How long after CVE publication were vulnerable packages introduced?** (Discovery lag)
- **How fast are findings remediated once introduced?** (Remediation velocity)
- **Is the attack surface growing or shrinking?** (Package count, RPS trend)

### Output

A comprehensive report with:
- Overall security grade (A-F)
- Patch efficacy assessment
- Findings and backlog trends
- Root cause analysis
- Prioritized recommendations

---

## 2. Prerequisites

### Required Access

- Data Service API: `http://localhost:1702`
- Orchestrate Service: `http://localhost:1707` (for peristalsis control)

### Required Knowledge

- Understanding of [Custom Metrics](./custom_metrics.md) (RPS, PES)
- Understanding of [Data Service API](./data_service_api.md)
- Understanding of [Entity Models](./entities/entities.md)

### Tools Needed

- `curl` for API queries
- `python3` for data analysis
- `jq` or Python for JSON processing

---

## 3. Data Sources

### 3.1 Primary Tables

#### datasetMetrics
**Endpoint:** `/api/v1/db/datasetMetrics/query?datasetName={DATASET}&isCurrent=true`

**Contains:**
- PES (patchEfficacyScore), RPS (rpsScore)
- Findings counts by severity
- Backlog aging (30-60d, 60-90d, 90+d)
- Package counts (total, downlevel, stale)
- Patch counts (patches, samePatches, differentPatches)

**Use For:** Temporal trend analysis, PES calculation

#### datasetMetrics/edit
**Endpoint:** `/api/v1/db/datasetMetrics/edit/query?datasetName={DATASET}&isCurrent=true&size=25000`

**Contains:**
- All edits (CREATE, UPDATE, DELETE)
- Before/after package versions
- Edit metadata (sameEdit, userEdit, pfRecommendedEdit)

**Use For:** Understanding patch behavior, edit type breakdown

#### findingData
**Endpoint:** `/api/v1/db/findingData/query?size=500`

**Contains:**
- CVE identifiers
- Severity levels
- NVD published dates (publishedAt)
- Dataset detection dates (reportedAt)
- Patch availability (patchedIn)

**Use For:** CVE age analysis, vulnerability severity breakdown

### 3.2 Query Examples

**Get dataset metrics (current state only):**
```bash
curl -s "http://localhost:1702/api/v1/db/datasetMetrics/query?datasetName=DATASET_NAME&isCurrent=true"
```

**Get all edits (last 6 months with date filter):**
```bash
curl -s "http://localhost:1702/api/v1/db/datasetMetrics/edit/query?datasetName=DATASET_NAME&isCurrent=true&commitDateTime=gte.2025-07-01T00:00:00Z&size=25000"
```

**Get finding data:**
```bash
curl -s "http://localhost:1702/api/v1/db/findingData/query?size=500"
```

**Using `select` parameter (AI-only feature):**
```bash
curl -s "http://localhost:1702/api/v1/db/datasetMetrics/query?datasetName=DATASET_NAME&isCurrent=true&select=commitDateTime,patchEfficacyScore,rpsScore,totalFindings,packages"
```

---

## 4. Key Metrics & Definitions

### 4.1 PES (Patch Efficacy Score)

**Formula:** `PES = Patch Impact / Patch Effort`

**Impact:** Benefit from reducing:
- Findings (vulnerabilities)
- RPS (version diversity)
- Stale packages
- Downlevel packages

**Effort:** Default 1 per patch, decreases with repetition (learning effect)

**Interpretation:**
- **Positive PES:** Patches improving security (good)
- **Zero PES:** Patches have no net benefit (neutral)
- **Negative PES:** Patches making things worse (bad)

**Typical Values:**
- Excellent: +5 to +10
- Good: +1 to +5
- Acceptable: -1 to +1
- Poor: -1 to -5
- Failing: < -5

### 4.2 RPS (Redundant Package Score)

**Definition:** Percentage of packages that exist in multiple versions across the dataset.

**Range:** 0 to 100

**Example:**
- 100 instances of jackson-databind, all version 2.15.0 → RPS = 0 (perfect)
- 100 instances across 10 different versions → RPS = 10 (fragmented)

**Interpretation:**
- **Low RPS (0-5):** Excellent version consolidation
- **Medium RPS (5-15):** Some fragmentation
- **High RPS (15-25):** Significant fragmentation
- **Very High RPS (>25):** Severe fragmentation

**Goal:** Decrease over time (consolidating versions)

### 4.3 Findings

**Types:**
- `totalFindings`: All CVEs detected
- `criticalFindings`: CVSS 9.0-10.0
- `highFindings`: CVSS 7.0-8.9
- `mediumFindings`: CVSS 4.0-6.9
- `lowFindings`: CVSS 0.1-3.9

**Related:**
- `packagesWithFindings`: Number of packages with at least one CVE
- Vulnerable %: `packagesWithFindings / packages * 100`

**Goal:** Decrease over time

### 4.4 Backlog

**Categories:**
- `findingsInBacklogBetweenThirtyAndSixtyDays`: Known 30-60 days
- `findingsInBacklogBetweenSixtyAndNinetyDays`: Known 60-90 days
- `findingsInBacklogOverNinetyDays`: Known 90+ days

**Interpretation:**
- **Good:** <30% in 90+ day bucket
- **Acceptable:** 30-50% in 90+ day bucket
- **Poor:** 50-70% in 90+ day bucket
- **Failing:** >70% in 90+ day bucket

**Goal:** Decrease backlog, especially 90+ day bucket

### 4.5 Downlevel & Stale

**Downlevel:** Package not at most current vendor version
- `downlevelPackagesMajor`: Major version behind
- `downlevelPackagesMinor`: Minor version behind
- `downlevelPackagesPatch`: Patch version behind

**Stale:** Package not updated by vendor in X time
- `stalePackagesTwoYears`: Not updated in >2 years

**Goal:** Decrease both counts and percentages

### 4.6 Edit Types

**From datasetMetrics:**
- `patches`: Total patches in snapshot
- `samePatches`: "Same version" edits (re-evaluation, NO version change)
- `differentPatches`: "Different version" edits (actual version changes)

**From edit records:**
- `CREATE`: New package added
- `UPDATE`: Package version changed (or re-evaluated)
- `DELETE`: Package removed

**Key Distinction:**
- `sameEdit: true` in edit record = "same version" (no actual change)
- `sameEdit: false` = actual version change

**Impact:**
- **Same version edits:** ZERO security benefit (can't fix vulnerabilities)
- **Different version edits:** POTENTIAL security benefit (if targeting CVEs)
- **CREATE edits:** Potentially adds vulnerabilities (growth)
- **DELETE edits:** Removes potential vulnerabilities (reduction)

---

## 5. Analysis Methodology

> **THE PATCHFOX ADVANTAGE: THE STORY IN YOUR DATA**
>
> PatchFox doesn't just show you where you are - it shows you **how you got here**. Every metric is a time series that tells a story: the beginning state, the journey, the inflection points, and the current chapter. When you see 100 findings, the question isn't "is 100 good or bad?" - it's "how did we get to 100, what happened along the way, and what does that journey tell us about what's working?"
>
> This runbook treats your data as a **narrative** - identifying chapters, turning points, and the organizational behaviors that drove change. Understanding the story is how you learn what to repeat and what to fix.

### Temporal Analysis Utilities

These utilities extract the story from your time series data.

```python
import json
from datetime import datetime, timedelta
from collections import defaultdict
from typing import List, Dict, Tuple, Optional

def parse_metrics(filepath: str) -> List[Dict]:
    """Load and sort metrics chronologically - the raw material for our story."""
    with open(filepath) as f:
        data = json.load(f)
    metrics = data['data']['titlePage']['content']
    metrics.sort(key=lambda x: x['commitDateTime'])
    return metrics

def extract_time_series(metrics: List[Dict], field: str) -> List[Tuple[datetime, float]]:
    """Extract a single metric as time series [(datetime, value), ...]."""
    series = []
    for m in metrics:
        dt = datetime.fromisoformat(m['commitDateTime'].replace('Z', '+00:00')).replace(tzinfo=None)
        value = m.get(field, 0) or 0
        series.append((dt, float(value)))
    return series


# =============================================================================
# STORY STRUCTURE: Beginning, Journey, Current State
# =============================================================================

def get_story_arc(series: List[Tuple[datetime, float]]) -> Dict:
    """
    Extract the narrative arc: where we started, where we are, and the journey between.

    Handles edge cases:
    - Datasets starting from zero (uses first non-zero value for percentage calc)
    - Very short time series
    """
    if len(series) < 2:
        return {'error': 'Insufficient data for story'}

    start_date, start_value = series[0]
    end_date, end_value = series[-1]
    total_days = (end_date - start_date).days

    # Find the midpoint for "middle of the story"
    mid_idx = len(series) // 2
    mid_date, mid_value = series[mid_idx]

    # Calculate total change
    total_change = end_value - start_value

    # Handle datasets starting from zero - find first meaningful baseline
    # This is common for new datasets where metrics accumulate over time
    if start_value == 0:
        # Find first non-zero value as baseline for percentage calculation
        baseline_value = None
        baseline_date = None
        for dt, val in series:
            if val > 0:
                baseline_value = val
                baseline_date = dt
                break

        if baseline_value and baseline_value != end_value:
            # Calculate change from first meaningful value
            total_change_pct = ((end_value - baseline_value) / baseline_value * 100)
            pct_note = f" (from first non-zero: {baseline_date.strftime('%Y-%m-%d')})"
        else:
            total_change_pct = 0
            pct_note = " (started from zero)"
    else:
        total_change_pct = (total_change / start_value * 100)
        pct_note = ""

    # Determine the overall narrative based on absolute change when starting from zero
    if start_value == 0:
        if end_value == 0:
            narrative = "no activity"
        elif end_value > 0:
            narrative = "grew from nothing"
        else:
            narrative = "declined into negative"
    elif abs(total_change_pct) < 10:
        narrative = "relatively stable"
    elif total_change > 0:
        narrative = "trending upward"
    else:
        narrative = "trending downward"

    return {
        'beginning': {
            'date': start_date,
            'value': start_value,
            'description': f"Started at {start_value:.1f} on {start_date.strftime('%Y-%m-%d')}"
        },
        'middle': {
            'date': mid_date,
            'value': mid_value,
            'description': f"Mid-journey: {mid_value:.1f} on {mid_date.strftime('%Y-%m-%d')}"
        },
        'current': {
            'date': end_date,
            'value': end_value,
            'description': f"Now at {end_value:.1f} as of {end_date.strftime('%Y-%m-%d')}"
        },
        'journey': {
            'total_days': total_days,
            'total_change': total_change,
            'total_change_pct': total_change_pct,
            'pct_note': pct_note,
            'narrative': narrative
        }
    }


# =============================================================================
# CHAPTERS: Identify distinct phases in the story
# =============================================================================

def identify_chapters(series: List[Tuple[datetime, float]], min_chapter_days: int = 30) -> List[Dict]:
    """
    Break the time series into chapters - distinct phases with consistent behavior.
    Each chapter has a character: growth, decline, stability, volatility.
    """
    if len(series) < 10:
        return [{'chapter': 1, 'description': 'Insufficient data for chapter analysis'}]

    chapters = []
    chapter_start = 0
    chapter_num = 1

    def get_chapter_character(segment):
        """Determine the character of a chapter segment."""
        if len(segment) < 2:
            return 'brief'

        values = [v for _, v in segment]
        start_val, end_val = values[0], values[-1]
        change_pct = ((end_val - start_val) / start_val * 100) if start_val else 0

        # Calculate volatility
        mean_val = sum(values) / len(values)
        variance = sum((v - mean_val) ** 2 for v in values) / len(values)
        volatility = (variance ** 0.5) / mean_val if mean_val else 0

        if volatility > 0.3:
            return 'volatile'
        elif change_pct > 15:
            return 'growth'
        elif change_pct < -15:
            return 'decline'
        else:
            return 'stability'

    # Sliding window to detect character changes
    window_size = max(5, len(series) // 20)
    prev_character = None

    for i in range(window_size, len(series)):
        window = series[i-window_size:i]
        character = get_chapter_character(window)

        if prev_character and character != prev_character:
            # Chapter transition detected
            chapter_segment = series[chapter_start:i]
            if len(chapter_segment) >= 3:
                start_date = chapter_segment[0][0]
                end_date = chapter_segment[-1][0]
                days = (end_date - start_date).days

                if days >= min_chapter_days:
                    chapters.append({
                        'chapter': chapter_num,
                        'start_date': start_date,
                        'end_date': end_date,
                        'days': days,
                        'start_value': chapter_segment[0][1],
                        'end_value': chapter_segment[-1][1],
                        'character': prev_character,
                        'data_points': len(chapter_segment)
                    })
                    chapter_num += 1
                    chapter_start = i

        prev_character = character

    # Don't forget the final chapter
    final_segment = series[chapter_start:]
    if len(final_segment) >= 3:
        chapters.append({
            'chapter': chapter_num,
            'start_date': final_segment[0][0],
            'end_date': final_segment[-1][0],
            'days': (final_segment[-1][0] - final_segment[0][0]).days,
            'start_value': final_segment[0][1],
            'end_value': final_segment[-1][1],
            'character': get_chapter_character(final_segment),
            'data_points': len(final_segment)
        })

    return chapters if chapters else [{'chapter': 1, 'character': get_chapter_character(series),
                                        'start_date': series[0][0], 'end_date': series[-1][0]}]


# =============================================================================
# INFLECTION POINTS: The turning points in our story
# =============================================================================

def find_turning_points(series: List[Tuple[datetime, float]], sensitivity: float = 0.15) -> List[Dict]:
    """
    Find significant turning points where the trend direction changed.
    These are the "plot twists" - moments worth investigating.
    """
    if len(series) < 10:
        return []

    turning_points = []
    window = max(3, len(series) // 20)

    for i in range(window, len(series) - window):
        before = series[i-window:i]
        after = series[i:i+window]

        # Calculate slopes
        before_change = (before[-1][1] - before[0][1]) / (before[-1][1] or 1)
        after_change = (after[-1][1] - after[0][1]) / (after[0][1] or 1)

        # Detect direction change
        if before_change > sensitivity and after_change < -sensitivity:
            turning_points.append({
                'date': series[i][0],
                'value': series[i][1],
                'type': 'peak_then_decline',
                'description': f"Peaked at {series[i][1]:.1f}, then began declining",
                'before_trend': f"+{before_change*100:.1f}%",
                'after_trend': f"{after_change*100:.1f}%",
                'investigation_prompt': "What changed? New policy? Team change? Tool adoption?"
            })
        elif before_change < -sensitivity and after_change > sensitivity:
            turning_points.append({
                'date': series[i][0],
                'value': series[i][1],
                'type': 'trough_then_recovery',
                'description': f"Bottomed at {series[i][1]:.1f}, then began recovering",
                'before_trend': f"{before_change*100:.1f}%",
                'after_trend': f"+{after_change*100:.1f}%",
                'investigation_prompt': "What intervention worked? Can we replicate it?"
            })

    return turning_points


# =============================================================================
# NOTABLE EVENTS: Anomalies that are part of the story
# =============================================================================

def find_notable_events(series: List[Tuple[datetime, float]], threshold_stddev: float = 2.0) -> List[Dict]:
    """
    Find notable events - sudden changes that stand out from the normal pattern.
    These are moments that demand explanation.
    """
    if len(series) < 5:
        return []

    # Calculate day-over-day changes
    changes = []
    for i in range(1, len(series)):
        prev_date, prev_val = series[i-1]
        curr_date, curr_val = series[i]
        change = curr_val - prev_val
        change_pct = (change / prev_val * 100) if prev_val else 0
        changes.append({
            'date': curr_date,
            'value': curr_val,
            'prev_value': prev_val,
            'change': change,
            'change_pct': change_pct
        })

    # Calculate statistics
    change_values = [c['change'] for c in changes]
    mean_change = sum(change_values) / len(change_values)
    variance = sum((c - mean_change) ** 2 for c in change_values) / len(change_values)
    stddev = variance ** 0.5

    # Find notable events
    events = []
    for c in changes:
        if stddev > 0:
            z_score = abs(c['change'] - mean_change) / stddev
            if z_score > threshold_stddev:
                event_type = 'spike' if c['change'] > 0 else 'drop'
                events.append({
                    'date': c['date'],
                    'value': c['value'],
                    'change': c['change'],
                    'change_pct': c['change_pct'],
                    'type': event_type,
                    'severity': z_score,
                    'description': f"{'Sudden increase' if event_type == 'spike' else 'Sudden decrease'} of {abs(c['change']):.1f} ({c['change_pct']:+.1f}%)",
                    'investigation_prompt': f"What happened on {c['date'].strftime('%Y-%m-%d')}? Check commits, releases, team changes."
                })

    return sorted(events, key=lambda x: x['severity'], reverse=True)


# =============================================================================
# VELOCITY: How fast things are changing (behavioral indicator)
# =============================================================================

def analyze_velocity(series: List[Tuple[datetime, float]], period_days: int = 30) -> Dict:
    """
    Analyze the velocity (rate of change) over time.
    Velocity changes tell us about organizational behavior.
    """
    if len(series) < 2:
        return {'error': 'Insufficient data'}

    # Overall velocity
    total_days = (series[-1][0] - series[0][0]).days or 1
    total_change = series[-1][1] - series[0][1]
    overall_velocity = (total_change / total_days) * period_days

    # Recent velocity (last period)
    recent_cutoff = series[-1][0] - timedelta(days=period_days)
    recent_points = [(d, v) for d, v in series if d >= recent_cutoff]

    if len(recent_points) >= 2:
        recent_days = (recent_points[-1][0] - recent_points[0][0]).days or 1
        recent_change = recent_points[-1][1] - recent_points[0][1]
        recent_velocity = (recent_change / recent_days) * period_days
    else:
        recent_velocity = overall_velocity

    # Historical velocity (before recent period)
    historical_points = [(d, v) for d, v in series if d < recent_cutoff]
    if len(historical_points) >= 2:
        hist_days = (historical_points[-1][0] - historical_points[0][0]).days or 1
        hist_change = historical_points[-1][1] - historical_points[0][1]
        historical_velocity = (hist_change / hist_days) * period_days
    else:
        historical_velocity = overall_velocity

    # Interpret the velocity story
    if abs(recent_velocity) < 0.1 and abs(historical_velocity) < 0.1:
        velocity_story = "Consistently stable - minimal change throughout"
    elif recent_velocity > 0 and historical_velocity <= 0:
        velocity_story = "Recent reversal - was improving, now degrading"
    elif recent_velocity < 0 and historical_velocity >= 0:
        velocity_story = "Recent improvement - was degrading or stable, now improving"
    elif abs(recent_velocity) > abs(historical_velocity) * 1.5:
        direction = "improvement" if recent_velocity < 0 else "degradation"
        velocity_story = f"Accelerating {direction} - change is speeding up"
    elif abs(recent_velocity) < abs(historical_velocity) * 0.5:
        velocity_story = "Decelerating - rate of change is slowing"
    else:
        velocity_story = "Consistent pace of change"

    return {
        'overall_velocity': overall_velocity,
        'recent_velocity': recent_velocity,
        'historical_velocity': historical_velocity,
        'velocity_story': velocity_story,
        'recent_vs_historical': recent_velocity / historical_velocity if historical_velocity else 1.0,
        'interpretation': f"Currently changing at {recent_velocity:+.1f}/month vs historical {historical_velocity:+.1f}/month"
    }


# =============================================================================
# PERIOD COMPARISON: This period vs last period
# =============================================================================

def compare_periods(series: List[Tuple[datetime, float]], period_days: int = 30) -> Dict:
    """
    Compare current period to previous period - a simple but powerful story element.
    """
    if len(series) < 2:
        return {'error': 'Insufficient data'}

    latest_date = series[-1][0]
    current_cutoff = latest_date - timedelta(days=period_days)
    previous_cutoff = current_cutoff - timedelta(days=period_days)

    current_values = [v for d, v in series if d > current_cutoff]
    previous_values = [v for d, v in series if previous_cutoff < d <= current_cutoff]

    current_avg = sum(current_values) / len(current_values) if current_values else 0
    previous_avg = sum(previous_values) / len(previous_values) if previous_values else current_avg

    change = current_avg - previous_avg
    change_pct = (change / previous_avg * 100) if previous_avg else 0

    return {
        'current_period_avg': current_avg,
        'previous_period_avg': previous_avg,
        'change': change,
        'change_pct': change_pct,
        'story': f"Last {period_days} days averaged {current_avg:.1f} vs {previous_avg:.1f} prior ({change_pct:+.1f}%)"
    }


# =============================================================================
# COMPLETE STORY: Pull it all together
# =============================================================================

def tell_metric_story(series: List[Tuple[datetime, float]], metric_name: str,
                      lower_is_better: bool = True) -> Dict:
    """
    Generate the complete narrative for a metric.
    This is the main function that tells the story of how we got here.
    """
    if not series or len(series) < 3:
        return {'error': 'Insufficient data to tell story', 'metric_name': metric_name}

    # Gather all story elements
    arc = get_story_arc(series)
    chapters = identify_chapters(series)
    turning_points = find_turning_points(series)
    notable_events = find_notable_events(series)
    velocity = analyze_velocity(series)
    period_comparison = compare_periods(series)

    # Build the narrative summary
    direction_word = "decreased" if arc['journey']['total_change'] < 0 else "increased"
    if not lower_is_better:
        good_bad = "good" if arc['journey']['total_change'] > 0 else "concerning"
    else:
        good_bad = "good" if arc['journey']['total_change'] < 0 else "concerning"

    narrative_summary = (
        f"{metric_name} started at {arc['beginning']['value']:.1f} on {arc['beginning']['date'].strftime('%Y-%m-%d')} "
        f"and {direction_word} to {arc['current']['value']:.1f} over {arc['journey']['total_days']} days "
        f"({arc['journey']['total_change_pct']:+.1f}%). This is {good_bad}. "
        f"{velocity['velocity_story']}."
    )

    # Add chapter summaries
    chapter_summary = []
    for ch in chapters:
        if isinstance(ch.get('start_date'), datetime):
            chapter_summary.append(
                f"Chapter {ch['chapter']} ({ch['start_date'].strftime('%b %Y')} - {ch['end_date'].strftime('%b %Y')}): "
                f"{ch['character'].upper()} phase, {ch['start_value']:.1f} → {ch['end_value']:.1f}"
            )

    # Key events to investigate
    investigate = []
    for tp in turning_points[:3]:
        investigate.append({
            'date': tp['date'].strftime('%Y-%m-%d'),
            'event': tp['description'],
            'prompt': tp['investigation_prompt']
        })
    for ne in notable_events[:3]:
        if not any(i['date'] == ne['date'].strftime('%Y-%m-%d') for i in investigate):
            investigate.append({
                'date': ne['date'].strftime('%Y-%m-%d'),
                'event': ne['description'],
                'prompt': ne['investigation_prompt']
            })

    return {
        'metric_name': metric_name,
        'narrative_summary': narrative_summary,
        'story_arc': arc,
        'chapters': chapters,
        'chapter_summary': chapter_summary,
        'turning_points': turning_points,
        'notable_events': notable_events[:5],
        'velocity': velocity,
        'period_comparison': period_comparison,
        'investigate': investigate[:5],
        'data_points': len(series),
        'lower_is_better': lower_is_better
    }


def print_metric_story(story: Dict):
    """Pretty print the story of a metric."""
    print(f"\n{'='*70}")
    print(f"  THE STORY OF {story['metric_name'].upper()}")
    print(f"{'='*70}")
    print()
    print(f"  SUMMARY:")
    print(f"  {story['narrative_summary']}")
    print()

    if story.get('chapter_summary'):
        print(f"  CHAPTERS:")
        for ch in story['chapter_summary']:
            print(f"    • {ch}")
        print()

    print(f"  RECENT TREND:")
    print(f"    {story['period_comparison']['story']}")
    print(f"    Velocity: {story['velocity']['interpretation']}")
    print()

    if story.get('investigate'):
        print(f"  KEY EVENTS TO INVESTIGATE:")
        for item in story['investigate']:
            print(f"    📍 {item['date']}: {item['event']}")
            print(f"       → {item['prompt']}")
        print()

    arc = story['story_arc']
    print(f"  JOURNEY SUMMARY:")
    print(f"    Start:   {arc['beginning']['description']}")
    print(f"    Current: {arc['current']['description']}")
    print(f"    Change:  {arc['journey']['total_change']:+.1f} ({arc['journey']['total_change_pct']:+.1f}%)")
    print()
```

### STEP 1: Initial Data Collection

> **IMPORTANT: API Pagination**
> The Data Service API caps results at **1000 records per request**. Large datasets may have 7,000+ metrics records and 20,000+ edit records. Always use the pagination helper below for complete data.

**1.1 Get Dataset Name**
```bash
curl -s "http://localhost:1702/api/v1/db/dataset/query" | python3 -m json.tool | head -30
```

Extract `name` field from first dataset.

**1.2 Fetch Dataset Metrics (Current) - WITH PAGINATION**

First, check how many records exist:
```bash
curl -s "http://localhost:1702/api/v1/db/datasetMetrics/query?datasetName={DATASET}&isCurrent=true&size=1" | python3 -c "import sys,json; d=json.load(sys.stdin); print(f\"Total records: {d['data']['titlePage']['totalElements']}\")"
```

For small datasets (<1000 records), use:
```bash
curl -s "http://localhost:1702/api/v1/db/datasetMetrics/query?datasetName={DATASET}&isCurrent=true&sort=commitDateTime.asc&size=1000" -o /tmp/metrics.json
```

For large datasets (>1000 records), use this pagination script:
```python
import json
import subprocess

DATASET = "YOUR_DATASET_NAME"  # Replace with actual dataset name
all_records = []
page = 0
page_size = 1000

while True:
    cmd = f'curl -s "http://localhost:1702/api/v1/db/datasetMetrics/query?datasetName={DATASET}&isCurrent=true&sort=commitDateTime.asc&size={page_size}&page={page}"'
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    data = json.loads(result.stdout)
    records = data["data"]["titlePage"]["content"]
    all_records.extend(records)
    print(f"Page {page}: fetched {len(records)} records, total: {len(all_records)}")

    if data["data"]["titlePage"]["last"]:
        break
    page += 1

print(f"\nTotal fetched: {len(all_records)}")

# Save in standard format for downstream analysis
with open("/tmp/metrics.json", "w") as f:
    json.dump({"data": {"titlePage": {"content": all_records}}}, f)
```

**1.3 Fetch All Edits (6 months) - WITH PAGINATION**

Calculate 6 months ago date:
```python
from datetime import datetime, timedelta
six_months = (datetime.now() - timedelta(days=180)).strftime('%Y-%m-%dT%H:%M:%SZ')
print(f"Date filter: {six_months}")
```

For large edit datasets, use pagination:
```python
import json
import subprocess
from datetime import datetime, timedelta

DATASET = "YOUR_DATASET_NAME"  # Replace with actual dataset name
date_filter = (datetime.now() - timedelta(days=180)).strftime('%Y-%m-%dT%H:%M:%SZ')

all_edits = []
page = 0
page_size = 1000

while True:
    cmd = f'curl -s "http://localhost:1702/api/v1/db/datasetMetrics/edit/query?datasetName={DATASET}&isCurrent=true&commitDateTime=gte.{date_filter}&size={page_size}&page={page}"'
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    data = json.loads(result.stdout)
    records = data["data"]["titlePage"]["content"]
    all_edits.extend(records)
    print(f"Page {page}: fetched {len(records)} edits, total: {len(all_edits)}")

    if data["data"]["titlePage"]["last"]:
        break
    page += 1

print(f"\nTotal edits fetched: {len(all_edits)}")

with open("/tmp/edits.json", "w") as f:
    json.dump({"data": {"titlePage": {"content": all_edits}}}, f)
```

**1.4 Fetch Finding Data**
```bash
curl -s "http://localhost:1702/api/v1/db/findingData/query?size=5000" -o /tmp/findings.json
```

**1.5 Fetch Critical CVEs (for emergency check)**
```bash
curl -s "http://localhost:1702/api/v1/db/findingData/query?severity=CRITICAL&size=500" -o /tmp/critical_cves.json
```

### STEP 2: Dataset Composition Analysis

**2.1 Fetch Datasource Metrics**
```bash
curl -s "http://localhost:1702/api/v1/db/datasourceMetricsCurrent/query?size=1000" -o /tmp/dmc.json
```

**2.2 Analyze Composition by Type**

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
    by_type[ds_type]['packages'] += r.get('packages', 0)
    by_type[ds_type]['findings'] += r.get('totalFindings', 0)
    by_type[ds_type]['critical'] += r.get('criticalFindings', 0)
    by_type[ds_type]['high'] += r.get('highFindings', 0)

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

**2.3 Interpret Composition**

Look for disproportionate relationships:

| Pattern | Meaning |
|---------|---------|
| Type has X% datasources but Y% packages (Y >> X) | Heavy dependency footprint per project (e.g., npm) |
| Type has X% packages but Y% findings (Y >> X) | Higher vulnerability density (riskier ecosystem) |
| Type has X% findings but Y% critical+high (Y >> X) | More severe vulnerabilities in that ecosystem |

**Red Flags:**
- One type dominates findings despite small datasource count → concentrated risk
- High finding % with low package % → vulnerable ecosystem, prioritize remediation
- Ecosystem with disproportionate critical+high → urgent attention needed

**Example Insight:** "npm is only 1.6% of datasources but 17% of packages - each npm project brings 10x more dependencies. pypi has 6x the vulnerability density of the dataset average."

### STEP 3: PES Analysis - The Patch Effectiveness Story

PES (Patch Efficacy Score) tells the story of whether your patching efforts are actually improving security. This isn't about "what's the average?" - it's about "how has patch effectiveness evolved, and what does that tell us?"

**3.1 Tell the PES Story**

```python
# Using the temporal utilities from above
metrics = parse_metrics('/tmp/metrics.json')
pes_series = extract_time_series(metrics, 'patchEfficacyScore')

# Get the full story - PES is special: HIGHER is better
pes_story = tell_metric_story(pes_series, "Patch Efficacy Score (PES)", lower_is_better=False)
print_metric_story(pes_story)
```

**3.2 PES Story Interpretation**

The PES story answers these questions:

| Question | Where to Look |
|----------|---------------|
| "Are patches getting more effective over time?" | `story_arc.journey.narrative` and `velocity.velocity_story` |
| "When did patch effectiveness change?" | `turning_points` - look for peaks and troughs |
| "What's the recent trend vs historical?" | `period_comparison` and `velocity.recent_vs_historical` |
| "Were there any dramatic shifts?" | `notable_events` - spikes or drops in PES |
| "What phases has PES gone through?" | `chapters` - periods of improvement, decline, stability |

**3.3 Key PES Story Patterns**

| Pattern | What It Means | Investigation |
|---------|---------------|---------------|
| Chapter of DECLINE followed by GROWTH | Something changed - policy? tooling? team? | Check what happened at the inflection point |
| Recent velocity reversal (was improving, now degrading) | Recent changes are hurting effectiveness | Review recent patch decisions |
| Spike in PES on specific date | A particularly effective patch batch | What made it effective? Replicate it |
| Consistent STABILITY near zero | Patches neither helping nor hurting | Patches may not be targeting real issues |
| VOLATILE chapters | Inconsistent patching approach | Need standardization |

**3.4 Example PES Story Output**

```
======================================================================
  THE STORY OF PATCH EFFICACY SCORE (PES)
======================================================================

  SUMMARY:
  Patch Efficacy Score (PES) started at -0.5 on 2024-01-15 and increased
  to 1.2 over 365 days (+340.0%). This is good. Recent improvement -
  was degrading or stable, now improving.

  CHAPTERS:
    • Chapter 1 (Jan 2024 - Apr 2024): DECLINE phase, -0.5 → -1.8
    • Chapter 2 (Apr 2024 - Aug 2024): STABILITY phase, -1.8 → -1.5
    • Chapter 3 (Aug 2024 - Jan 2025): GROWTH phase, -1.5 → 1.2

  RECENT TREND:
    Last 30 days averaged 1.1 vs 0.3 prior (+266.7%)
    Velocity: Currently changing at +0.8/month vs historical +0.15/month

  KEY EVENTS TO INVESTIGATE:
    📍 2024-08-12: Bottomed at -1.8, then began recovering
       → What intervention worked? Can we replicate it?
    📍 2024-11-03: Sudden increase of 0.9 (+150.0%)
       → What happened on 2024-11-03? Check commits, releases, team changes.

  JOURNEY SUMMARY:
    Start:   Started at -0.5 on 2024-01-15
    Current: Now at 1.2 as of 2025-01-15
    Change:  +1.7 (+340.0%)
```

**3.5 Grading (Informed by Story)**

The grade considers both current state AND trajectory:

| Grade | Criteria |
|-------|----------|
| A | Current PES > 1 AND improving or stable |
| B | Current PES 0-1 AND improving, OR PES > 1 but declining |
| C | Current PES -1 to 0, OR positive but rapidly declining |
| D | Current PES -1 to -2, OR near-zero with no improvement trend |
| F | Current PES < -2, OR any negative PES with worsening trend |

### STEP 4: RPS Analysis - The Version Fragmentation Story

RPS (Redundant Package Score) tells the story of version consolidation. Are you converging on standard versions, or fragmenting into chaos?

**4.1 Tell the RPS Story**

```python
rps_series = extract_time_series(metrics, 'rpsScore')
rps_story = tell_metric_story(rps_series, "Redundant Package Score (RPS)", lower_is_better=True)
print_metric_story(rps_story)
```

**4.2 RPS Story Interpretation**

| Question | Where to Look |
|----------|---------------|
| "Is version fragmentation getting better or worse?" | `story_arc.journey.narrative` |
| "When did fragmentation patterns change?" | `turning_points` and `chapters` |
| "Are consolidation efforts working?" | `velocity` - is recent velocity negative (improving)? |
| "Any sudden fragmentation events?" | `notable_events` - spikes indicate bulk additions of new versions |

**4.3 Key RPS Story Patterns**

| Pattern | What It Means | Action |
|---------|---------------|--------|
| Steady DECLINE over time | Consolidation efforts working | Continue approach |
| GROWTH chapters | Periods of uncontrolled additions | Investigate what drove new versions |
| Spike events | Bulk imports or major version bumps | Check if intentional |
| VOLATILE with no trend | No consolidation strategy | Implement version pinning policy |
| Recent velocity reversal (was declining, now growing) | Consolidation efforts stalled | Re-engage standardization |

**4.4 Grading (Informed by Story)**

| Grade | Criteria |
|-------|----------|
| A | RPS < 5 AND declining or stable |
| B | RPS 5-15 AND declining, OR RPS < 5 but growing slowly |
| C | RPS 5-15 stable, OR 15-25 declining |
| D | RPS 15-25 stable or growing, OR any RPS with accelerating growth |
| F | RPS > 25, OR any RPS with rapid growth trend |

### STEP 5: Findings Analysis - The Vulnerability Story

This is often the most important story - how has your vulnerability exposure evolved over time?

**5.1 Tell the Findings Story**

```python
findings_series = extract_time_series(metrics, 'totalFindings')
findings_story = tell_metric_story(findings_series, "Total Findings", lower_is_better=True)
print_metric_story(findings_story)

# Also analyze critical findings separately - they have their own story
critical_series = extract_time_series(metrics, 'criticalFindings')
critical_story = tell_metric_story(critical_series, "Critical Findings", lower_is_better=True)
print_metric_story(critical_story)
```

**5.2 Discovery vs Remediation: The Two Forces**

Findings change due to two opposing forces:
1. **Discovery** - new CVEs found in existing packages (increases findings)
2. **Remediation** - fixing/removing vulnerable packages (decreases findings)

To understand the story, you need to see both:

```python
def analyze_discovery_vs_remediation(metrics):
    """
    Separate the discovery and remediation components of the findings story.
    """
    results = []

    for i in range(1, len(metrics)):
        prev = metrics[i-1]
        curr = metrics[i]

        prev_date = datetime.fromisoformat(prev['commitDateTime'].replace('Z', '+00:00'))
        curr_date = datetime.fromisoformat(curr['commitDateTime'].replace('Z', '+00:00'))

        findings_change = curr['totalFindings'] - prev['totalFindings']
        packages_change = curr['packages'] - prev['packages']

        # Estimate: if packages decreased and findings decreased, likely remediation
        # If packages stable/grew and findings increased, likely discovery
        if findings_change < 0:
            remediation_estimate = abs(findings_change)
            discovery_estimate = 0
        else:
            # Some findings increase could be discovery, some could be from new packages
            if packages_change > 0:
                # Rough heuristic: attribute some to new packages
                discovery_estimate = findings_change
                remediation_estimate = 0
            else:
                discovery_estimate = findings_change
                remediation_estimate = 0

        results.append({
            'date': curr_date,
            'findings_change': findings_change,
            'packages_change': packages_change,
            'discovery_estimate': discovery_estimate,
            'remediation_estimate': remediation_estimate
        })

    # Summarize
    total_discovery = sum(r['discovery_estimate'] for r in results)
    total_remediation = sum(r['remediation_estimate'] for r in results)

    print(f"\nDISCOVERY vs REMEDIATION STORY:")
    print(f"  Estimated new discoveries: +{total_discovery}")
    print(f"  Estimated remediations: -{total_remediation}")
    print(f"  Net effect: {total_discovery - total_remediation:+d}")

    if total_remediation > total_discovery:
        print(f"  → Remediation outpacing discovery (GOOD)")
    elif total_discovery > total_remediation * 1.5:
        print(f"  → Discovery significantly outpacing remediation (CONCERNING)")
    else:
        print(f"  → Roughly balanced (STABLE)")

    return results

analyze_discovery_vs_remediation(metrics)
```

**5.3 Severity Trends - Each Severity Has Its Own Story**

```python
# Track how severity mix has changed over time
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

    # Story of severity mix
    first = metrics[0]
    last = metrics[-1]

    first_crit_high_pct = ((first.get('criticalFindings', 0) + first.get('highFindings', 0)) /
                          first.get('totalFindings', 1) * 100)
    last_crit_high_pct = ((last.get('criticalFindings', 0) + last.get('highFindings', 0)) /
                         last.get('totalFindings', 1) * 100)

    print(f"\nSEVERITY MIX STORY:")
    print(f"  Critical+High started at {first_crit_high_pct:.1f}% of findings")
    print(f"  Critical+High now at {last_crit_high_pct:.1f}% of findings")

    if last_crit_high_pct < first_crit_high_pct - 5:
        print(f"  → Severity mix improving - high-severity findings being prioritized")
    elif last_crit_high_pct > first_crit_high_pct + 5:
        print(f"  → Severity mix worsening - high-severity findings accumulating")
    else:
        print(f"  → Severity mix stable")

analyze_severity_story(metrics)
```

**5.4 Key Findings Story Patterns**

| Pattern | What It Means | Action |
|---------|---------------|--------|
| Steady DECLINE | Active remediation working | Continue and celebrate |
| GROWTH despite patches | Discovery outpacing remediation | Increase remediation velocity |
| Spike followed by decline | Major discovery event, then cleanup | Good response pattern |
| Critical findings GROWTH while total stable | Prioritizing wrong things | Shift focus to severity |
| VOLATILE with no trend | Reactive, not proactive | Need systematic approach |

**5.5 Grading (Informed by Story)**

| Grade | Criteria |
|-------|----------|
| A | Findings declining AND critical findings declining faster |
| B | Findings stable or slowly declining |
| C | Findings slowly growing but critical stable/declining |
| D | Findings growing AND critical growing |
| F | Rapid findings growth OR critical findings exploding |

### STEP 6: Backlog Analysis - The Remediation Velocity Story

The backlog tells you not just how many vulnerabilities exist, but how long they've been sitting there. This is the story of **remediation velocity** - are you closing findings, or are they aging in place?

**6.1 Tell the Backlog Story**

```python
# Calculate total backlog over time
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

**6.2 The Aging Story - How Findings Move Through Buckets**

The real story is in how findings flow through the aging buckets:

```python
def analyze_backlog_flow(metrics):
    """
    Analyze how findings flow through backlog aging buckets.
    This tells us about remediation velocity.
    """
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

    # Tell the flow story
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
    print(f"  Total backlog: {first_total:.0f} → {last_total:.0f} ({last_total - first_total:+.0f})")
    print(f"  90+ day percentage: {first_90_pct:.1f}% → {last_90_pct:.1f}%")

    if last_total < first_total and last_90_pct < first_90_pct:
        print(f"  → EXCELLENT: Backlog shrinking AND aging improving")
    elif last_total < first_total:
        print(f"  → GOOD: Backlog shrinking (but watch the 90+ bucket)")
    elif last_90_pct > first_90_pct + 10:
        print(f"  → CONCERNING: Findings aging in place - remediation stalled")
    elif last_total > first_total * 1.5:
        print(f"  → CRITICAL: Backlog growing significantly")
    else:
        print(f"  → STABLE: Holding steady")

analyze_backlog_flow(metrics)
```

**6.3 Key Backlog Story Patterns**

| Pattern | What It Means | Action |
|---------|---------------|--------|
| 90+ bucket growing while others stable | Findings entering but not leaving | Remediation stalled - need to clear old items |
| All buckets shrinking | Active remediation across the board | Continue approach |
| 30-60 bucket spikes, others stable | New findings discovered | Normal - watch if they flow to 90+ |
| 90+ percentage increasing over time | Systemic remediation failure | Emergency intervention needed |
| Volatile backlog with no trend | Reactive approach | Need systematic remediation process |

**6.4 Grading (Informed by Story)**

| Grade | Criteria |
|-------|----------|
| A | Backlog declining AND 90+ percentage < 30% |
| B | Backlog stable AND 90+ percentage < 50% |
| C | Backlog stable OR slowly growing, 90+ percentage 50-70% |
| D | Backlog growing AND 90+ percentage > 70% |
| F | Rapid backlog growth OR 90+ percentage > 85% with no improvement trend |

### STEP 7: Edit Type Analysis - The Patching Behavior Story

Edits tell the story of **how** you're patching, not just **that** you're patching. Are you making meaningful changes, or churning without impact?

**7.1 Tell the Edit Behavior Story**

```python
from collections import Counter, defaultdict

with open('/tmp/edits.json') as f:
    data = json.load(f)
edits = data['data']['titlePage']['content']

# Sort edits by date
edits.sort(key=lambda x: x.get('commitDateTime', ''))

def analyze_edit_story(edits):
    """
    Analyze the story of patching behavior over time.
    """
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

    # Trend in same-version percentage
    if len(months) >= 2:
        first_months = months[:len(months)//2]
        last_months = months[len(months)//2:]

        first_same_pct = sum(by_month[m]['same'] for m in first_months) / sum(by_month[m]['total'] for m in first_months) * 100
        last_same_pct = sum(by_month[m]['same'] for m in last_months) / sum(by_month[m]['total'] for m in last_months) * 100

        print(f"\n  EFFICIENCY TREND:")
        print(f"    First half same%: {first_same_pct:.1f}%")
        print(f"    Second half same%: {last_same_pct:.1f}%")

        if last_same_pct < first_same_pct - 5:
            print(f"    → IMPROVING: More meaningful patches over time")
        elif last_same_pct > first_same_pct + 5:
            print(f"    → DEGRADING: More churn, less impact over time")
        else:
            print(f"    → STABLE: Consistent patching behavior")

    # Growth story
    if create_total > delete_total * 1.5:
        print(f"\n  GROWTH STORY: Adding packages faster than removing ({create_total} vs {delete_total})")
        print(f"    → Attack surface expanding")
    elif delete_total > create_total:
        print(f"\n  GROWTH STORY: Removing more than adding ({delete_total} vs {create_total})")
        print(f"    → Attack surface contracting (good)")
    else:
        print(f"\n  GROWTH STORY: Balanced adds/removes")
        print(f"    → Controlled growth")

analyze_edit_story(edits)
```

**7.2 Key Edit Story Patterns**

| Pattern | What It Means | Action |
|---------|---------------|--------|
| Same% declining over time | Learning - patches getting more targeted | Continue improvement |
| Same% increasing over time | Regression - more churn, less impact | Review patch selection criteria |
| CREATE >> DELETE | Uncontrolled growth | Implement removal discipline |
| DELETE >> CREATE | Active pruning | Good - may indicate cleanup initiative |
| High volume + high same% | Busy work | Stop and reassess what's being patched |
| Low volume + high different% | Selective, meaningful patches | Ideal state |

**7.3 Grading (Informed by Story)**

| Grade | Criteria |
|-------|----------|
| A | Same% < 30% AND declining trend AND CREATE ≈ DELETE |
| B | Same% 30-50% OR Same% < 30% but CREATE > DELETE |
| C | Same% 50-66% with stable trend |
| D | Same% > 66% OR Same% increasing significantly |
| F | Same% > 80% OR CREATE >> DELETE with high volume |

### STEP 8: Package Family Remediation Analysis

> **Prerequisites: This step requires direct database access**
>
> The queries in this step run against PostgreSQL directly. If you don't have database access, use the **REST API Alternative** below.
>
> **Database Connection:**
> ```bash
> # Default local connection
> psql -h localhost -p 5432 -U patchfox -d patchfox
>
> # Or via docker
> docker exec -it patchfox-postgres psql -U patchfox -d patchfox
> ```

#### REST API Alternative (No DB Access Required)

If direct database access is unavailable, you can approximate this analysis using the REST API:

```python
import json
from collections import defaultdict

# Load the metrics we already have
with open('/tmp/metrics.json') as f:
    data = json.load(f)

# Get the latest metrics record
metrics = data['data']['titlePage']['content']
latest = max(metrics, key=lambda x: x['commitDateTime'])

# RPS indicates version fragmentation - a proxy for consolidation opportunity
rps = latest.get('rpsScore', 0)
packages = latest.get('packages', 0)
packages_with_findings = latest.get('packagesWithFindings', 0)
total_findings = latest.get('totalFindings', 0)

print("PACKAGE FAMILY REMEDIATION (API APPROXIMATION)")
print("=" * 50)
print(f"RPS Score: {rps:.1f}")
print(f"  - RPS > 15 indicates significant version fragmentation")
print(f"  - Version consolidation can reduce both RPS and findings")
print(f"\nTotal packages: {packages:,}")
print(f"Packages with findings: {packages_with_findings:,} ({packages_with_findings/packages*100:.1f}%)")
print(f"Total findings: {total_findings:,}")

# Estimate consolidation opportunity based on RPS
# Higher RPS = more versions = more consolidation potential
if rps > 20:
    print(f"\n>>> HIGH CONSOLIDATION OPPORTUNITY")
    print(f"    RPS of {rps:.1f} suggests many packages exist in multiple versions.")
    print(f"    Estimate: 40-60% of findings may be eliminable via version standardization.")
elif rps > 10:
    print(f"\n>>> MODERATE CONSOLIDATION OPPORTUNITY")
    print(f"    RPS of {rps:.1f} suggests some version fragmentation.")
    print(f"    Estimate: 20-40% of findings may be eliminable via version standardization.")
else:
    print(f"\n>>> LIMITED CONSOLIDATION OPPORTUNITY")
    print(f"    RPS of {rps:.1f} suggests good version consolidation already.")
    print(f"    Focus on upgrading rather than consolidating.")

print("\n⚠️  For precise family-level analysis, run the SQL queries with direct DB access.")
```

---

**8.1 Calculate Aggregate Remediation Potential** (Requires DB Access)

First, get the headline number - how many CVE instances could be eliminated by standardizing on package versions you already use elsewhere.

Run directly against the database:
```sql
-- Aggregate: How many findings could be eliminated via version consolidation?
WITH the_latest_metric AS (
    SELECT package_indexes
    FROM public.dataset_metrics
    WHERE is_current = true
    ORDER BY commit_date_time DESC
    LIMIT 1
),
latest_packages AS (
    SELECT p.id, p.name, p.namespace, p.version,
           COALESCE(p.namespace, '') || '/' || p.name AS family_key
    FROM public.package p, the_latest_metric m
    WHERE p.id = ANY(m.package_indexes)
),
package_families AS (
    SELECT family_key, COUNT(*) AS family_size
    FROM latest_packages
    GROUP BY family_key
    HAVING COUNT(*) > 1
),
packages_in_multi_member_families AS (
    SELECT lp.*
    FROM latest_packages lp
    JOIN package_families pf ON lp.family_key = pf.family_key
),
findings_with_patched_versions AS (
    SELECT pf.finding_id, pmf.family_key
    FROM public.package_finding pf
    JOIN packages_in_multi_member_families pmf ON pf.package_id = pmf.id
    JOIN package_families fam ON pmf.family_key = fam.family_key
    GROUP BY pf.finding_id, pmf.family_key, fam.family_size
    HAVING COUNT(DISTINCT pf.package_id) < fam.family_size
)
SELECT
    COUNT(pf.finding_id) AS total_finding_instances_in_families,
    COUNT(fpv.finding_id) AS instances_with_patched_versions_in_family,
    ROUND(COUNT(fpv.finding_id)::numeric / NULLIF(COUNT(pf.finding_id), 0) * 100, 1) AS pct_eliminable
FROM public.package_finding pf
JOIN packages_in_multi_member_families pmf ON pf.package_id = pmf.id
LEFT JOIN findings_with_patched_versions fpv
    ON pf.finding_id = fpv.finding_id
    AND pmf.family_key = fpv.family_key;
```

**Interpret Results:**
- `total_finding_instances_in_families`: CVEs in packages with version fragmentation
- `instances_with_patched_versions_in_family`: CVEs eliminable via consolidation
- `pct_eliminable`: Percentage of findings that are "low-hanging fruit"

**Example:** "847 CVE instances exist in fragmented packages. 612 (72%) could be eliminated by standardizing on versions already in use elsewhere."

**8.2 Identify Specific Consolidation Opportunities**

Now get the detailed breakdown - which specific package families offer the best opportunities.

Run directly against the database:
```sql
-- Find package families with remediation opportunities
WITH the_latest_metric AS (
    SELECT package_indexes
    FROM public.dataset_metrics
    WHERE is_current = true
    ORDER BY commit_date_time DESC
    LIMIT 1
),
latest_packages AS (
    SELECT p.id, p.name, p.namespace, p.version,
           COALESCE(p.namespace, '') || '/' || p.name AS family_key
    FROM public.package p
    INNER JOIN the_latest_metric m ON p.id = ANY(m.package_indexes)
),
package_families AS (
    SELECT family_key, COUNT(*) AS family_size
    FROM latest_packages
    GROUP BY family_key
    HAVING COUNT(*) > 1
),
packages_in_multi_member_families AS (
    SELECT lp.*, pf.family_size
    FROM latest_packages lp
    JOIN package_families pf ON lp.family_key = pf.family_key
),
packages_with_any_findings AS (
    SELECT DISTINCT package_id
    FROM public.package_finding
    WHERE package_id IN (SELECT id FROM packages_in_multi_member_families)
)
SELECT
    pmf.family_key,
    COUNT(pf.finding_id) AS total_finding_instances,
    MAX(pmf.family_size) AS total_family_members,
    COUNT(DISTINCT paf.package_id) AS family_members_with_findings,
    (MAX(pmf.family_size) - COUNT(DISTINCT paf.package_id)) AS family_members_without_findings
FROM packages_in_multi_member_families pmf
LEFT JOIN public.package_finding pf ON pmf.id = pf.package_id
LEFT JOIN packages_with_any_findings paf ON pmf.id = paf.package_id
GROUP BY pmf.family_key
HAVING COUNT(DISTINCT paf.package_id) > 0  -- Only families with at least one vulnerable version
   AND (MAX(pmf.family_size) - COUNT(DISTINCT paf.package_id)) > 0  -- And at least one clean version
ORDER BY total_finding_instances DESC
LIMIT 20;
```

**8.3 Interpret Results**

| Column | Meaning |
|--------|---------|
| `family_key` | Package family (namespace/name) |
| `total_finding_instances` | Total CVEs across all versions |
| `total_family_members` | Number of versions in dataset |
| `family_members_with_findings` | Versions that have vulnerabilities |
| `family_members_without_findings` | Versions that are clean |

**High-Value Targets:**
- Families with high `total_finding_instances` AND `family_members_without_findings > 0`
- These can be consolidated to clean versions, eliminating CVEs AND reducing RPS

**Example Output:**
```
family_key                                    | findings | members | vuln | clean
----------------------------------------------+----------+---------+------+------
com.fasterxml.jackson.core/jackson-databind   |       47 |       5 |    3 |     2
org.apache.commons/commons-compress           |       12 |       4 |    2 |     2
io.netty/netty-codec-http                     |        8 |       3 |    2 |     1
```

**Interpretation:** jackson-databind has 5 versions in the dataset, 3 with vulnerabilities (47 total findings), and 2 clean versions. Consolidating to one of the clean versions eliminates 47 CVEs and reduces RPS by 4.

**8.4 Grade Remediation Opportunity**

- `pct_eliminable` > 50%: Excellent - majority of fragmented-package CVEs are low-hanging fruit
- `pct_eliminable` 25-50%: Good - significant consolidation opportunity
- `pct_eliminable` < 25%: Limited - most CVEs require actual upgrades, not just consolidation

**Detailed breakdown grading:**
- Many high-value targets (>5 families with 10+ findings): Excellent opportunity, prioritize consolidation
- Some targets (2-5 families): Good opportunity, include in remediation plan
- Few targets (<2 families): Limited quick wins, focus on other strategies

**8.5 Generate Actionable Patch Recommendations**

This step transforms the analysis into specific, actionable patch recommendations suitable for JIRA tickets or automated PRs.

**8.5.1 Identify Emergency CVEs**

First, check for "drop everything" vulnerabilities - CRITICAL severity CVEs that need immediate attention.

```bash
curl -s "http://localhost:1702/api/v1/db/findingData/query?severity=CRITICAL&size=500" -o /tmp/critical_cves.json
```

```python
import json

with open('/tmp/critical_cves.json') as f:
    data = json.load(f)

critical = data['data']['titlePage']['content']

# Known catastrophic CVEs (log4shell, etc.)
EMERGENCY_CVES = {
    'CVE-2021-44228': 'Log4Shell - Remote Code Execution',
    'CVE-2021-45046': 'Log4Shell followup - RCE',
    'CVE-2021-45105': 'Log4Shell followup - DoS',
    'CVE-2022-22965': 'Spring4Shell - RCE',
    'CVE-2021-26855': 'ProxyLogon - RCE',
    'CVE-2023-44487': 'HTTP/2 Rapid Reset - DoS',
}

print("=== EMERGENCY CVE CHECK ===\n")
emergencies = []
for cve in critical:
    identifier = cve.get('identifier', '')
    if identifier in EMERGENCY_CVES:
        emergencies.append({
            'cve': identifier,
            'description': EMERGENCY_CVES[identifier],
            'detail': cve.get('description', '')
        })

if emergencies:
    print("🚨 CRITICAL: EMERGENCY CVEs DETECTED - IMMEDIATE ACTION REQUIRED 🚨\n")
    for e in emergencies:
        print(f"  {e['cve']}: {e['description']}")
        print(f"    {e['detail']}\n")
else:
    print("✓ No known emergency CVEs (log4shell, spring4shell, etc.) detected\n")

print(f"Total CRITICAL severity CVEs: {len(critical)}")
for cve in critical[:10]:
    print(f"  {cve.get('identifier')}: {cve.get('description', '')[:60]}...")
```

**8.5.2 Map Packages to Datasources**

Identify which datasources contain vulnerable packages.

```bash
curl -s "http://localhost:1702/api/v1/db/datasourceMetricsCurrent/query?size=1000" -o /tmp/dmc.json
```

```python
import json
import re

with open('/tmp/dmc.json') as f:
    dmc_data = json.load(f)

datasources = dmc_data['data']['titlePage']['content']

# Build lookup: package type -> datasources
ds_by_type = {}
for ds in datasources:
    purl = ds.get('purl', '')
    ds_type = purl.split('@')[-1] if '@' in purl else 'unknown'
    ds_name = purl.split('/')[2].split('%3A')[0] if '//' not in purl and len(purl.split('/')) > 2 else purl

    if ds_type not in ds_by_type:
        ds_by_type[ds_type] = []
    ds_by_type[ds_type].append({
        'name': ds_name,
        'purl': purl,
        'packages': ds.get('packages', 0),
        'findings': ds.get('totalFindings', 0),
        'critical': ds.get('criticalFindings', 0)
    })

print("Datasources by package type:")
for pkg_type, ds_list in sorted(ds_by_type.items()):
    print(f"  {pkg_type}: {len(ds_list)} datasources")
```

**8.5.3 Generate Patch Tickets**

Combine family analysis with datasource mapping to produce actionable recommendations.

```python
# Assuming you have the family analysis results from 8.2
# For each high-value family, generate a ticket-ready recommendation

def assess_breaking_risk(from_version, to_version, pkg_name=None, pkg_type=None):
    """Assess breaking change risk based on semver AND changelog analysis"""
    import urllib.request
    import json as json_lib

    risk_factors = []
    changelog_notes = []

    # 1. Semver analysis
    try:
        from_parts = re.split(r'[.\-]', from_version)[:3]
        to_parts = re.split(r'[.\-]', to_version)[:3]

        from_major = int(re.sub(r'[^\d]', '', from_parts[0])) if from_parts else 0
        to_major = int(re.sub(r'[^\d]', '', to_parts[0])) if to_parts else 0

        from_minor = int(re.sub(r'[^\d]', '', from_parts[1])) if len(from_parts) > 1 else 0
        to_minor = int(re.sub(r'[^\d]', '', to_parts[1])) if len(to_parts) > 1 else 0

        if to_major > from_major:
            semver_risk = "HIGH"
            risk_factors.append("Major version bump")
        elif to_minor > from_minor:
            semver_risk = "MEDIUM"
            risk_factors.append("Minor version bump")
        else:
            semver_risk = "LOW"
            risk_factors.append("Patch version bump")
    except:
        semver_risk = "UNKNOWN"
        risk_factors.append("Could not parse version numbers")

    # 2. Changelog analysis (where available)
    if pkg_name and pkg_type:
        try:
            changelog_text = ""

            # NPM: Fetch from registry
            if pkg_type == 'npm':
                url = f"https://registry.npmjs.org/{pkg_name}/{to_version}"
                with urllib.request.urlopen(url, timeout=5) as resp:
                    data = json_lib.loads(resp.read())
                    # Check for deprecation warnings
                    if data.get('deprecated'):
                        risk_factors.append("⚠️ VERSION DEPRECATED")
                        changelog_notes.append(f"Deprecation: {data['deprecated']}")

            # PyPI: Fetch release info
            elif pkg_type == 'pypi':
                url = f"https://pypi.org/pypi/{pkg_name}/{to_version}/json"
                with urllib.request.urlopen(url, timeout=5) as resp:
                    data = json_lib.loads(resp.read())
                    # Check classifiers for stability
                    classifiers = data.get('info', {}).get('classifiers', [])
                    for c in classifiers:
                        if 'Development Status' in c:
                            changelog_notes.append(f"Status: {c}")
                        if 'Alpha' in c or 'Beta' in c:
                            risk_factors.append("Pre-release version")

            # Maven: Check for release notes in description
            elif pkg_type == 'maven':
                # Maven Central search API
                group_id, artifact_id = pkg_name.rsplit('/', 1) if '/' in pkg_name else ('', pkg_name)
                group_id = group_id.replace('/', '.')
                url = f"https://search.maven.org/solrsearch/select?q=g:{group_id}+AND+a:{artifact_id}+AND+v:{to_version}&rows=1&wt=json"
                with urllib.request.urlopen(url, timeout=5) as resp:
                    data = json_lib.loads(resp.read())
                    docs = data.get('response', {}).get('docs', [])
                    if docs:
                        # Check timestamp - very old versions may have compatibility issues
                        timestamp = docs[0].get('timestamp', 0)
                        if timestamp > 0:
                            from datetime import datetime
                            release_date = datetime.fromtimestamp(timestamp/1000)
                            age_years = (datetime.now() - release_date).days / 365
                            if age_years > 3:
                                changelog_notes.append(f"Release is {age_years:.1f} years old")

        except Exception as e:
            changelog_notes.append(f"(Changelog fetch failed: {str(e)[:50]})")

    # 3. Keyword analysis in version string
    breaking_keywords = ['alpha', 'beta', 'rc', 'snapshot', 'pre', 'dev']
    if any(kw in to_version.lower() for kw in breaking_keywords):
        risk_factors.append("Pre-release/unstable version indicator")
        if semver_risk == "LOW":
            semver_risk = "MEDIUM"

    # Compile final assessment
    risk_reason = "; ".join(risk_factors)
    if changelog_notes:
        risk_reason += "\n    Changelog: " + "; ".join(changelog_notes)

    return semver_risk, risk_reason


def fetch_changelog_summary(pkg_name, pkg_type, from_version, to_version):
    """Attempt to fetch and summarize changelog between versions"""
    import json
    import urllib.request

    summary = []

    try:
        if pkg_type == 'npm':
            # GitHub releases are often linked in npm
            url = f"https://registry.npmjs.org/{pkg_name}"
            with urllib.request.urlopen(url, timeout=5) as resp:
                data = json.loads(resp.read())
                repo = data.get('repository', {})
                if isinstance(repo, dict) and 'url' in repo:
                    repo_url = repo['url']
                    if 'github.com' in repo_url:
                        summary.append(f"GitHub: {repo_url.replace('git+', '').replace('.git', '')}/releases")

        elif pkg_type == 'pypi':
            url = f"https://pypi.org/pypi/{pkg_name}/json"
            with urllib.request.urlopen(url, timeout=5) as resp:
                data = json.loads(resp.read())
                project_urls = data.get('info', {}).get('project_urls', {})
                for key in ['Changelog', 'Release Notes', 'History', 'Changes']:
                    if key in project_urls:
                        summary.append(f"{key}: {project_urls[key]}")
                        break

    except:
        pass

    return summary or ["No changelog URL found - manual review recommended"]

# Template for each recommendation
TICKET_TEMPLATE = """
================================================================================
PATCH RECOMMENDATION: {family_key}
================================================================================
PRIORITY: {priority}
RISK LEVEL: {risk_level}
RISK REASON: {risk_reason}

ACTION: Update {family_key}
  FROM: {vulnerable_versions}
  TO:   {target_version}

IMPACT:
  - CVEs eliminated: {cve_count}
  - Critical/High CVEs: {critical_high}
  - RPS reduction: {rps_reduction}

AFFECTED DATASOURCES ({ds_type}):
{datasource_list}

CVEs RESOLVED:
{cve_list}

CHANGELOG / RELEASE NOTES:
{changelog_links}

NOTES:
{notes}
================================================================================
"""

# Example output generation (customize based on your 8.2 query results)
def generate_ticket(family_key, vulnerable_versions, target_version, cve_count,
                    critical_high, pkg_type, cve_details):

    # Extract package name from family_key (namespace/name format)
    pkg_name = family_key.split('/')[-1] if '/' in family_key else family_key

    # Assess breaking risk with changelog analysis
    risk_level, risk_reason = assess_breaking_risk(
        vulnerable_versions[0], target_version,
        pkg_name=pkg_name, pkg_type=pkg_type
    )

    # Fetch changelog links for manual review
    changelog_links = fetch_changelog_summary(pkg_name, pkg_type, vulnerable_versions[0], target_version)

    # Set priority based on CVE severity
    if critical_high > 0:
        priority = "P1 - CRITICAL" if 'CRITICAL' in str(cve_details) else "P2 - HIGH"
    else:
        priority = "P3 - MEDIUM"

    # Get affected datasources
    affected_ds = ds_by_type.get(pkg_type, [])
    ds_list = "\n".join([f"  - {ds['name']} ({ds['findings']} findings)"
                         for ds in affected_ds[:10]])
    if len(affected_ds) > 10:
        ds_list += f"\n  ... and {len(affected_ds) - 10} more"

    return TICKET_TEMPLATE.format(
        family_key=family_key,
        priority=priority,
        risk_level=risk_level,
        risk_reason=risk_reason,
        vulnerable_versions=", ".join(vulnerable_versions),
        target_version=target_version,
        cve_count=cve_count,
        critical_high=critical_high,
        rps_reduction=len(vulnerable_versions),
        ds_type=pkg_type,
        datasource_list=ds_list or "  (Unable to determine specific datasources)",
        cve_list=cve_details[:500] + "..." if len(cve_details) > 500 else cve_details,
        changelog_links="\n".join(f"  - {link}" for link in changelog_links),
        notes="- Review changelog for BREAKING CHANGES before upgrading\n- Run full test suite after upgrade\n- Consider staged rollout for major version bumps"
    )

# Example usage with mock data - replace with actual 8.2 results:
# print(generate_ticket(
#     family_key="com.fasterxml.jackson.core/jackson-databind",
#     vulnerable_versions=["2.9.8", "2.9.10", "2.10.0"],
#     target_version="2.12.3",
#     cve_count=47,
#     critical_high=12,
#     pkg_type="maven",
#     cve_details="CVE-2020-36518, CVE-2020-25649, CVE-2019-14540..."
# ))
```

**8.5.4 Breaking Change Risk Matrix**

| Risk Level | Version Change | Action |
|------------|----------------|--------|
| LOW | Patch bump (1.2.3 → 1.2.4) | Safe to auto-merge |
| MEDIUM | Minor bump (1.2.x → 1.3.x) | Review changelog, run tests |
| HIGH | Major bump (1.x → 2.x) | Manual review required, expect API changes |
| UNKNOWN | Non-semver or unparseable | Manual investigation required |

**8.5.5 Output Formats**

For JIRA integration, export recommendations as CSV:
```python
import csv

# Write to CSV for JIRA import
with open('/tmp/patch_recommendations.csv', 'w', newline='') as f:
    writer = csv.writer(f)
    writer.writerow(['Summary', 'Priority', 'Component', 'Description', 'Labels'])
    # Add rows from your generated tickets
```

For automated PR creation, export as JSON:
```python
# Write to JSON for automation
recommendations = []
# ... populate from analysis ...
with open('/tmp/patch_recommendations.json', 'w') as f:
    json.dump(recommendations, f, indent=2)
```

### STEP 9: CVE Age Analysis - The Technical Debt Story

CVE age tells the story of **technical debt** - how long have known vulnerabilities been sitting unaddressed? This is where you find the skeletons in the closet.

**9.1 Tell the CVE Age Story**

```python
from datetime import datetime

with open('/tmp/findings.json') as f:
    data = json.load(f)
findings = data['data']['titlePage']['content']

def analyze_cve_age_story(findings):
    """
    Tell the story of how long CVEs have been present.
    """
    ages = []
    now = datetime.now()

    for finding in findings:
        published = finding.get('publishedAt')
        if published:
            pub_dt = datetime.fromisoformat(published.replace('Z', '+00:00')).replace(tzinfo=None)
            age_days = (now - pub_dt).days
            ages.append({
                'cve': finding.get('identifier'),
                'severity': finding.get('severity'),
                'age_days': age_days,
                'age_years': age_days / 365,
                'published': pub_dt
            })

    if not ages:
        print("No CVEs with published dates found")
        return

    # Sort by age
    ages.sort(key=lambda x: x['age_days'], reverse=True)

    # Distribution
    buckets = {
        '<6 months': [a for a in ages if a['age_days'] < 180],
        '6-12 months': [a for a in ages if 180 <= a['age_days'] < 365],
        '1-2 years': [a for a in ages if 365 <= a['age_days'] < 730],
        '2-3 years': [a for a in ages if 730 <= a['age_days'] < 1095],
        '3+ years': [a for a in ages if a['age_days'] >= 1095]
    }

    print("CVE AGE STORY:")
    print("=" * 60)
    print(f"\nTotal CVEs analyzed: {len(ages)}")
    print(f"Oldest CVE: {ages[0]['cve']} - {ages[0]['age_years']:.1f} years old ({ages[0]['severity']})")
    print(f"Newest CVE: {ages[-1]['cve']} - {ages[-1]['age_days']} days old ({ages[-1]['severity']})")
    print(f"Average age: {sum(a['age_days'] for a in ages) / len(ages) / 365:.1f} years")

    print(f"\nAGE DISTRIBUTION:")
    print(f"{'Bucket':<15} {'Count':>8} {'%':>8} {'Critical':>10} {'High':>8}")
    print("-" * 55)

    for bucket_name, bucket_cves in buckets.items():
        count = len(bucket_cves)
        pct = count / len(ages) * 100 if ages else 0
        critical = sum(1 for c in bucket_cves if c['severity'] == 'CRITICAL')
        high = sum(1 for c in bucket_cves if c['severity'] == 'HIGH')
        print(f"{bucket_name:<15} {count:>8} {pct:>7.1f}% {critical:>10} {high:>8}")

    # The story
    old_cves = len(buckets['2-3 years']) + len(buckets['3+ years'])
    old_pct = old_cves / len(ages) * 100 if ages else 0

    critical_old = sum(1 for a in ages if a['age_days'] > 365 and a['severity'] == 'CRITICAL')

    print(f"\nTHE TECHNICAL DEBT STORY:")
    if old_pct > 50:
        print(f"  ⚠️ SEVERE: {old_pct:.0f}% of CVEs are over 2 years old")
        print(f"     This indicates systemic neglect of security updates")
    elif old_pct > 25:
        print(f"  ⚠️ CONCERNING: {old_pct:.0f}% of CVEs are over 2 years old")
        print(f"     Technical debt is accumulating")
    else:
        print(f"  ✓ Manageable: Only {old_pct:.0f}% of CVEs are over 2 years old")

    if critical_old > 0:
        print(f"\n  🚨 CRITICAL FINDING: {critical_old} CRITICAL CVEs are over 1 year old")
        print(f"     These should have been emergency priorities")
        oldest_critical = [a for a in ages if a['severity'] == 'CRITICAL']
        if oldest_critical:
            print(f"     Oldest CRITICAL: {oldest_critical[0]['cve']} ({oldest_critical[0]['age_years']:.1f} years)")

    # Ancient CVEs worth calling out
    ancient = [a for a in ages if a['age_days'] > 1095]  # 3+ years
    if ancient:
        print(f"\n  ANCIENT CVEs (3+ years):")
        for a in ancient[:5]:
            print(f"    • {a['cve']} ({a['severity']}): {a['age_years']:.1f} years")
        if len(ancient) > 5:
            print(f"    ... and {len(ancient) - 5} more")

    return ages

ages = analyze_cve_age_story(findings)
```

**9.2 The Age Story Patterns**

| Pattern | What It Means | Action |
|---------|---------------|--------|
| Majority < 1 year | Active scanning, normal discovery | Good - maintain velocity |
| Growing 2+ year bucket | CVEs aging out, not being fixed | Need dedicated cleanup sprint |
| CRITICAL CVEs > 1 year | Failed triage process | Emergency remediation |
| Even distribution | Steady state without progress | Need remediation velocity increase |
| Spike in recent CVEs | Major discovery event | Normal - ensure they don't age |

**9.3 Grading (Informed by Story)**

| Grade | Criteria |
|-------|----------|
| A | No CVEs > 2 years AND no CRITICAL > 1 year |
| B | < 10% CVEs > 2 years AND no CRITICAL > 1 year |
| C | < 25% CVEs > 2 years OR a few old CRITICAL being worked |
| D | > 25% CVEs > 2 years OR CRITICAL > 2 years |
| F | > 50% CVEs > 2 years OR multiple ancient CRITICAL |

### STEP 10: Package Growth Analysis - The Attack Surface Story

Package count tells the story of your **attack surface** - is it expanding or contracting? Growth isn't inherently bad, but uncontrolled growth with stagnant remediation is dangerous.

**10.1 Tell the Package Growth Story**

```python
packages_series = extract_time_series(metrics, 'packages')
packages_story = tell_metric_story(packages_series, "Package Count", lower_is_better=True)
print_metric_story(packages_story)
```

**10.2 The Growth vs Security Story**

The real insight comes from correlating package growth with findings:

```python
def analyze_growth_vs_security(metrics):
    """
    Tell the story of how package growth relates to security outcomes.
    """
    first = metrics[0]
    last = metrics[-1]

    # Package growth
    pkg_start = first.get('packages', 0)
    pkg_end = last.get('packages', 0)
    pkg_change = pkg_end - pkg_start
    pkg_change_pct = (pkg_change / pkg_start * 100) if pkg_start else 0

    # Findings growth
    find_start = first.get('totalFindings', 0)
    find_end = last.get('totalFindings', 0)
    find_change = find_end - find_start
    find_change_pct = (find_change / find_start * 100) if find_start else 0

    # Vulnerable percentage
    vuln_pct_start = (find_start / pkg_start * 100) if pkg_start else 0
    vuln_pct_end = (find_end / pkg_end * 100) if pkg_end else 0

    print("\nGROWTH vs SECURITY STORY:")
    print("=" * 60)
    print(f"\n  PACKAGES:")
    print(f"    Start: {pkg_start:,}")
    print(f"    End: {pkg_end:,}")
    print(f"    Change: {pkg_change:+,} ({pkg_change_pct:+.1f}%)")

    print(f"\n  FINDINGS:")
    print(f"    Start: {find_start:,}")
    print(f"    End: {find_end:,}")
    print(f"    Change: {find_change:+,} ({find_change_pct:+.1f}%)")

    print(f"\n  VULNERABILITY DENSITY:")
    print(f"    Start: {vuln_pct_start:.2f}% of packages vulnerable")
    print(f"    End: {vuln_pct_end:.2f}% of packages vulnerable")
    print(f"    Change: {vuln_pct_end - vuln_pct_start:+.2f}pp")

    # The story
    print(f"\n  THE STORY:")
    if pkg_change_pct > 0 and find_change_pct < 0:
        print(f"  ✓ EXCELLENT: Growing package base ({pkg_change_pct:+.1f}%) with")
        print(f"    DECLINING vulnerabilities ({find_change_pct:+.1f}%)")
        print(f"    → Security improving faster than growth")
    elif pkg_change_pct > 0 and find_change_pct > 0 and find_change_pct < pkg_change_pct:
        print(f"  ○ GOOD: Packages grew {pkg_change_pct:+.1f}%, findings grew {find_change_pct:+.1f}%")
        print(f"    → Keeping pace, but watch the trend")
    elif pkg_change_pct > 0 and find_change_pct > pkg_change_pct:
        print(f"  ⚠️ CONCERNING: Findings growing FASTER than packages")
        print(f"    Packages: {pkg_change_pct:+.1f}%, Findings: {find_change_pct:+.1f}%")
        print(f"    → Security degrading despite (or because of) growth")
    elif pkg_change_pct < 0 and find_change_pct < 0:
        print(f"  ✓ GOOD: Controlled contraction - reducing surface AND vulnerabilities")
    elif pkg_change_pct < 0 and find_change_pct > 0:
        print(f"  🚨 CRITICAL: Packages declining but findings increasing")
        print(f"    → Removing packages but not the vulnerable ones")
    else:
        print(f"  ○ STABLE: Minimal change in both dimensions")

    # Calculate velocity correlation
    print(f"\n  VELOCITY COMPARISON:")
    days = (datetime.fromisoformat(last['commitDateTime'].replace('Z', '')).replace(tzinfo=None) -
            datetime.fromisoformat(first['commitDateTime'].replace('Z', '')).replace(tzinfo=None)).days or 1
    pkg_per_month = pkg_change / days * 30
    find_per_month = find_change / days * 30
    print(f"    Package velocity: {pkg_per_month:+.1f}/month")
    print(f"    Findings velocity: {find_per_month:+.1f}/month")

    return {
        'pkg_change_pct': pkg_change_pct,
        'find_change_pct': find_change_pct,
        'vuln_density_change': vuln_pct_end - vuln_pct_start
    }

analyze_growth_vs_security(metrics)
```

**10.3 Key Growth Story Patterns**

| Pattern | What It Means | Action |
|---------|---------------|--------|
| Packages ↑, Findings ↓ | Security improving as you grow | Ideal - maintain |
| Packages ↑, Findings ↑ (slower) | Keeping pace | Good - improve remediation |
| Packages ↑, Findings ↑ (faster) | Losing ground | Halt growth, focus on security |
| Packages ↓, Findings ↓ | Controlled reduction | Good cleanup effort |
| Packages ↓, Findings ↑ | Removing wrong things | Review what's being cut |

**10.4 Grading (Informed by Story)**

| Grade | Criteria |
|-------|----------|
| A | Findings declining regardless of package growth |
| B | Findings growing slower than packages (improving density) |
| C | Findings and packages growing at similar rates |
| D | Findings growing faster than packages |
| F | Findings exploding OR packages declining while findings grow |

### STEP 11: Synthesize the Complete Story

Now we bring all the individual stories together into a coherent narrative.

**11.1 The Overall Narrative**

```python
def synthesize_story(metrics, edits, findings):
    """
    Bring all stories together into a coherent narrative.
    """
    # Gather all story elements
    pes_series = extract_time_series(metrics, 'patchEfficacyScore')
    rps_series = extract_time_series(metrics, 'rpsScore')
    findings_series = extract_time_series(metrics, 'totalFindings')
    packages_series = extract_time_series(metrics, 'packages')

    pes_story = tell_metric_story(pes_series, "PES", lower_is_better=False)
    rps_story = tell_metric_story(rps_series, "RPS", lower_is_better=True)
    findings_story = tell_metric_story(findings_series, "Findings", lower_is_better=True)
    packages_story = tell_metric_story(packages_series, "Packages", lower_is_better=True)

    print("\n" + "=" * 70)
    print("  THE COMPLETE SECURITY STORY")
    print("=" * 70)

    # Chapter 1: Where we started
    print("\n📖 CHAPTER 1: THE BEGINNING")
    print("-" * 40)
    first = metrics[0]
    first_date = first['commitDateTime'][:10]
    print(f"   On {first_date}, this dataset had:")
    print(f"   • {first.get('packages', 0):,} packages")
    print(f"   • {first.get('totalFindings', 0):,} findings ({first.get('criticalFindings', 0)} critical)")
    print(f"   • PES of {first.get('patchEfficacyScore', 0):.2f}")
    print(f"   • RPS of {first.get('rpsScore', 0):.1f}")

    # Chapter 2: The journey
    print("\n📖 CHAPTER 2: THE JOURNEY")
    print("-" * 40)

    # Describe each metric's journey
    for story in [pes_story, findings_story, rps_story]:
        name = story['metric_name']
        arc = story['story_arc']
        print(f"\n   {name}:")
        print(f"   {arc['journey']['narrative']} over {arc['journey']['total_days']} days")
        if story.get('chapter_summary'):
            for ch in story['chapter_summary'][:2]:
                print(f"     • {ch}")

    # Chapter 3: Key events
    print("\n📖 CHAPTER 3: THE TURNING POINTS")
    print("-" * 40)

    all_events = []
    for story in [pes_story, findings_story, rps_story]:
        for event in story.get('investigate', []):
            all_events.append({
                'metric': story['metric_name'],
                'date': event['date'],
                'event': event['event'],
                'prompt': event['prompt']
            })

    # Sort by date
    all_events.sort(key=lambda x: x['date'])

    if all_events:
        for event in all_events[:5]:
            print(f"   📍 {event['date']} ({event['metric']})")
            print(f"      {event['event']}")
            print(f"      → {event['prompt']}")
    else:
        print("   No significant turning points detected")

    # Chapter 4: Where we are now
    print("\n📖 CHAPTER 4: THE CURRENT STATE")
    print("-" * 40)
    last = metrics[-1]
    last_date = last['commitDateTime'][:10]
    print(f"   As of {last_date}:")
    print(f"   • {last.get('packages', 0):,} packages")
    print(f"   • {last.get('totalFindings', 0):,} findings ({last.get('criticalFindings', 0)} critical)")
    print(f"   • PES of {last.get('patchEfficacyScore', 0):.2f}")
    print(f"   • RPS of {last.get('rpsScore', 0):.1f}")

    # Net changes
    print(f"\n   THE BOTTOM LINE:")
    pkg_change = last.get('packages', 0) - first.get('packages', 0)
    find_change = last.get('totalFindings', 0) - first.get('totalFindings', 0)
    pes_change = last.get('patchEfficacyScore', 0) - first.get('patchEfficacyScore', 0)
    rps_change = last.get('rpsScore', 0) - first.get('rpsScore', 0)

    print(f"   • Packages: {pkg_change:+,}")
    print(f"   • Findings: {find_change:+,}")
    print(f"   • PES: {pes_change:+.2f}")
    print(f"   • RPS: {rps_change:+.1f}")

    # Chapter 5: The verdict
    print("\n📖 CHAPTER 5: THE VERDICT")
    print("-" * 40)

    # Count positives and negatives
    positives = []
    negatives = []

    if pes_change > 0:
        positives.append("Patch effectiveness improving")
    elif pes_change < -0.5:
        negatives.append("Patch effectiveness declining")

    if find_change < 0:
        positives.append("Vulnerabilities decreasing")
    elif find_change > 0:
        negatives.append("Vulnerabilities increasing")

    if rps_change < 0:
        positives.append("Version fragmentation improving")
    elif rps_change > 2:
        negatives.append("Version fragmentation worsening")

    if pes_story['velocity'].get('velocity_story', '').startswith('Recent improvement'):
        positives.append("Recent momentum is positive")
    elif 'reversal' in pes_story['velocity'].get('velocity_story', '').lower():
        negatives.append("Recent momentum has stalled")

    print("   WHAT'S WORKING:")
    for p in positives or ["Nothing stands out as improving"]:
        print(f"   ✓ {p}")

    print("\n   WHAT NEEDS ATTENTION:")
    for n in negatives or ["No critical issues identified"]:
        print(f"   ⚠️ {n}")

    # Overall assessment
    if len(positives) >= 3 and len(negatives) == 0:
        verdict = "EXCELLENT - Strong positive trajectory"
    elif len(positives) > len(negatives):
        verdict = "GOOD - More wins than losses"
    elif len(positives) == len(negatives):
        verdict = "MIXED - Progress in some areas, regression in others"
    elif len(negatives) > len(positives):
        verdict = "CONCERNING - More losses than wins"
    else:
        verdict = "CRITICAL - Significant intervention needed"

    print(f"\n   OVERALL: {verdict}")

    return {
        'stories': {'pes': pes_story, 'rps': rps_story, 'findings': findings_story, 'packages': packages_story},
        'positives': positives,
        'negatives': negatives,
        'verdict': verdict
    }

synthesis = synthesize_story(metrics, edits, findings)
```

**11.2 Root Cause Analysis Through the Story Lens**

Common story patterns and what they reveal:

**Pattern 1: "The Busy Without Impact Story"**
- High patch activity + negative PES + stable/growing findings
- Story: "We're patching a lot but not the right things"
- Root cause: Patches targeting non-vulnerable packages
- Fix: Vulnerability-driven patch prioritization

**Pattern 2: "The Outrunning Story"**
- Package growth > remediation velocity
- Story: "We're adding packages faster than we can secure them"
- Root cause: No growth controls, reactive security
- Fix: Slow growth, proactive security reviews

**Pattern 3: "The Fragmentation Story"**
- RPS increasing + PES negative
- Story: "We're creating version chaos without benefit"
- Root cause: Decentralized patching without coordination
- Fix: Version pinning, coordinated upgrade campaigns

**Pattern 4: "The Neglect Story"**
- High % of CVEs > 2 years old + stable findings
- Story: "Vulnerabilities are discovered but never fixed"
- Root cause: No remediation accountability
- Fix: SLA enforcement, backlog grooming

**Pattern 5: "The Recovery Story"**
- Past chapters of decline + recent improvement
- Story: "Things got bad, but we turned it around"
- Root cause: Policy change or new tooling
- Success: Document and replicate what worked

### STEP 12: Generate Report - Telling the Complete Story

**12.1 Report Generation Script**

This final step ties all analysis together and produces both terminal highlights and a full markdown report.

```python
from datetime import date, datetime
import json

def generate_report(dataset_name, metrics, edits, findings, composition, family_analysis):
    """
    Generate terminal summary and full markdown report.

    Args:
        dataset_name: Name of the dataset
        metrics: List of datasetMetrics records (sorted by commitDateTime)
        edits: List of edit records
        findings: List of findingData records
        composition: Dict from Step 2 composition analysis {type: {count, packages, findings, critical, high}}
        family_analysis: List of dicts from Step 8 [{family_key, findings, clean_versions, ...}]
    """

    # === CALCULATE ALL METRICS ===

    # PES
    avg_pes = sum(m['patchEfficacyScore'] for m in metrics) / len(metrics)
    latest_pes = metrics[-1]['patchEfficacyScore']
    positive_pes_pct = sum(1 for m in metrics if m['patchEfficacyScore'] > 0) / len(metrics) * 100

    # RPS
    rps_start = metrics[0]['rpsScore']
    rps_end = metrics[-1]['rpsScore']
    rps_trend = "Decreasing" if rps_end < rps_start else "Increasing" if rps_end > rps_start else "Stable"

    # Findings
    findings_start = metrics[0]['totalFindings']
    findings_end = metrics[-1]['totalFindings']
    findings_change = findings_end - findings_start
    findings_trend = "Decreasing" if findings_change < 0 else "Increasing" if findings_change > 0 else "Stable"

    # Backlog
    latest = metrics[-1]
    backlog_30_60 = latest.get('findingsInBacklogBetweenThirtyAndSixtyDays', 0)
    backlog_60_90 = latest.get('findingsInBacklogBetweenSixtyAndNinetyDays', 0)
    backlog_90_plus = latest.get('findingsInBacklogOverNinetyDays', 0)
    total_backlog = backlog_30_60 + backlog_60_90 + backlog_90_plus
    pct_90_plus = (backlog_90_plus / total_backlog * 100) if total_backlog > 0 else 0

    # Edit efficiency
    same_edits = sum(1 for e in edits if e.get('sameEdit', False))
    same_edit_pct = (same_edits / len(edits) * 100) if edits else 0

    # Severity breakdown
    critical = latest.get('criticalFindings', 0)
    high = latest.get('highFindings', 0)
    medium = latest.get('mediumFindings', 0)
    low = latest.get('lowFindings', 0)

    # === CALCULATE GRADES ===

    def grade_pes(avg):
        if avg > 1: return 'A'
        if avg > 0: return 'B'
        if avg > -1: return 'C'
        if avg > -2: return 'D'
        return 'F'

    def grade_findings(trend, change_pct):
        if trend == "Decreasing": return 'A'
        if trend == "Stable": return 'C'
        if change_pct < 20: return 'C'
        if change_pct < 50: return 'D'
        return 'F'

    def grade_backlog(pct_90):
        if pct_90 < 30: return 'A'
        if pct_90 < 50: return 'C'
        if pct_90 < 70: return 'D'
        return 'F'

    def grade_rps(trend):
        if trend == "Decreasing": return 'A'
        if trend == "Stable": return 'C'
        return 'F'

    def grade_edits(same_pct):
        if same_pct < 30: return 'A'
        if same_pct < 50: return 'C'
        if same_pct < 66: return 'D'
        return 'F'

    findings_change_pct = abs(findings_change / findings_start * 100) if findings_start > 0 else 0

    grades = {
        'pes': grade_pes(avg_pes),
        'findings': grade_findings(findings_trend, findings_change_pct),
        'backlog': grade_backlog(pct_90_plus),
        'rps': grade_rps(rps_trend),
        'edits': grade_edits(same_edit_pct)
    }

    # Weighted overall grade
    grade_values = {'A': 4, 'B': 3, 'C': 2, 'D': 1, 'F': 0}
    weighted_score = (
        grade_values[grades['pes']] * 0.30 +
        grade_values[grades['findings']] * 0.25 +
        grade_values[grades['backlog']] * 0.20 +
        grade_values[grades['rps']] * 0.15 +
        grade_values[grades['edits']] * 0.10
    )

    if weighted_score >= 3.5: overall_grade = 'A'
    elif weighted_score >= 2.5: overall_grade = 'B'
    elif weighted_score >= 1.5: overall_grade = 'C'
    elif weighted_score >= 0.5: overall_grade = 'D'
    else: overall_grade = 'F'

    # === IDENTIFY CRITICAL ISSUES ===

    critical_issues = []
    if grades['pes'] in ['D', 'F']:
        critical_issues.append(f"PES is negative ({avg_pes:.2f}) - patches making things worse")
    if grades['backlog'] == 'F':
        critical_issues.append(f"Backlog critical: {pct_90_plus:.0f}% of findings are 90+ days old")
    if critical > 0:
        critical_issues.append(f"{critical} CRITICAL severity CVEs present")
    if grades['findings'] == 'F':
        critical_issues.append(f"Findings increased {findings_change_pct:.0f}% - accumulating debt")

    # === TOP RECOMMENDATIONS ===

    recommendations = []
    if family_analysis:
        top_family = max(family_analysis, key=lambda x: x.get('total_finding_instances', 0))
        if top_family.get('total_finding_instances', 0) > 10:
            recommendations.append(f"Consolidate {top_family['family_key']} - eliminates {top_family['total_finding_instances']} CVE instances")
    if grades['backlog'] in ['D', 'F']:
        recommendations.append("Prioritize remediation of 90+ day backlog items")
    if grades['pes'] in ['D', 'F']:
        recommendations.append("Target patches at vulnerable packages, not just version bumps")
    if rps_trend == "Increasing":
        recommendations.append("Implement version pinning to reduce fragmentation")

    # === TERMINAL OUTPUT ===

    print()
    print("=" * 70)
    print(f"  PATCHFOX SECURITY ANALYSIS: {dataset_name}")
    print(f"  Generated: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
    print("=" * 70)
    print()
    print(f"  OVERALL GRADE: {overall_grade}")
    print()
    print(f"  PES: {avg_pes:.2f} (Grade: {grades['pes']})  |  RPS: {rps_end:.1f} ({rps_trend}, Grade: {grades['rps']})")
    print(f"  Findings: {findings_end} ({findings_trend} {abs(findings_change):+d}, Grade: {grades['findings']})")
    print(f"  Backlog: {total_backlog:.0f} total, {pct_90_plus:.0f}% in 90+d (Grade: {grades['backlog']})")
    print(f"  Edit Efficiency: {100-same_edit_pct:.0f}% different-version (Grade: {grades['edits']})")
    print()

    if critical_issues:
        print("  CRITICAL ISSUES:")
        for issue in critical_issues:
            print(f"    ⚠ {issue}")
        print()

    if recommendations:
        print("  TOP RECOMMENDATIONS:")
        for i, rec in enumerate(recommendations[:5], 1):
            print(f"    {i}. {rec}")
        print()

    print("=" * 70)

    # === MARKDOWN REPORT ===

    report_date = date.today().isoformat()
    report_path = f"/tmp/{dataset_name}_analysis_{report_date}.md"

    # Build composition table
    comp_table = "| Type | Datasources | Packages | Findings | Critical | High |\n"
    comp_table += "|------|-------------|----------|----------|----------|------|\n"
    for pkg_type, stats in sorted(composition.items(), key=lambda x: x[1]['findings'], reverse=True):
        comp_table += f"| {pkg_type} | {stats['count']} | {stats['packages']} | {stats['findings']} | {stats['critical']} | {stats['high']} |\n"

    # Build family remediation table
    family_table = ""
    if family_analysis:
        family_table = "| Package Family | CVE Instances | Versions | Vulnerable | Clean |\n"
        family_table += "|----------------|---------------|----------|------------|-------|\n"
        for fam in sorted(family_analysis, key=lambda x: x.get('total_finding_instances', 0), reverse=True)[:15]:
            family_table += f"| {fam.get('family_key', 'N/A')} | {fam.get('total_finding_instances', 0)} | {fam.get('total_family_members', 0)} | {fam.get('family_members_with_findings', 0)} | {fam.get('family_members_without_findings', 0)} |\n"

    report_content = f"""# PatchFox Security Analysis Report

**Dataset:** {dataset_name}
**Generated:** {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}
**Analysis Period:** {metrics[0]['commitDateTime'][:10]} to {metrics[-1]['commitDateTime'][:10]}

---

## Executive Summary

**Overall Grade: {overall_grade}**

This dataset has a Patch Efficacy Score (PES) of **{avg_pes:.2f}**, indicating {"effective patching that improves security posture" if avg_pes > 0 else "patches that are not effectively improving security"}.
The vulnerability backlog shows **{pct_90_plus:.0f}%** of findings aged 90+ days, {"indicating active remediation" if pct_90_plus < 30 else "indicating remediation lag"}.
{"Key concerns include: " + "; ".join(critical_issues[:3]) + "." if critical_issues else "No critical issues identified."}

---

## Part 1: Dataset Composition

{comp_table}

**Analysis Period:** {len(metrics)} snapshots analyzed

---

## Part 2: Patch Efficacy (PES)

| Metric | Value | Grade |
|--------|-------|-------|
| Average PES | {avg_pes:.2f} | {grades['pes']} |
| Latest PES | {latest_pes:.2f} | - |
| Positive PES % | {positive_pes_pct:.1f}% | - |
| Average Impact | {sum(m['patchImpact'] for m in metrics) / len(metrics):.2f} | - |
| Average Effort | {sum(m['patchEffort'] for m in metrics) / len(metrics):.2f} | - |

**Interpretation:** {"Patches are effectively reducing vulnerabilities and complexity." if avg_pes > 1 else "Patches are having neutral to negative security impact." if avg_pes < 0 else "Patches are having modest positive impact."}

---

## Part 3: Findings & Backlog

### Current Severity Breakdown

| Severity | Count |
|----------|-------|
| CRITICAL | {critical} |
| HIGH | {high} |
| MEDIUM | {medium} |
| LOW | {low} |
| **TOTAL** | **{findings_end}** |

### Findings Trend

- **Start:** {findings_start} findings
- **End:** {findings_end} findings
- **Change:** {findings_change:+d} ({findings_change_pct:.1f}%)
- **Trend:** {findings_trend}
- **Grade:** {grades['findings']}

### Backlog Aging

| Bucket | Count | Percentage |
|--------|-------|------------|
| 30-60 days | {backlog_30_60:.0f} | {(backlog_30_60/total_backlog*100) if total_backlog > 0 else 0:.1f}% |
| 60-90 days | {backlog_60_90:.0f} | {(backlog_60_90/total_backlog*100) if total_backlog > 0 else 0:.1f}% |
| 90+ days | {backlog_90_plus:.0f} | {pct_90_plus:.1f}% |
| **Total** | **{total_backlog:.0f}** | 100% |

**Grade:** {grades['backlog']}

---

## Part 4: Version Fragmentation (RPS)

| Metric | Value |
|--------|-------|
| Starting RPS | {rps_start:.1f} |
| Ending RPS | {rps_end:.1f} |
| Change | {rps_end - rps_start:+.1f} |
| Trend | {rps_trend} |
| Grade | {grades['rps']} |

**Interpretation:** {"Version consolidation is improving - fewer duplicate package versions." if rps_trend == "Decreasing" else "Version fragmentation is increasing - more duplicate package versions across the codebase."}

---

## Part 5: Edit Analysis

| Metric | Value |
|--------|-------|
| Total Edits | {len(edits):,} |
| Same-Version Edits | {same_edits:,} ({same_edit_pct:.1f}%) |
| Different-Version Edits | {len(edits) - same_edits:,} ({100-same_edit_pct:.1f}%) |
| Grade | {grades['edits']} |

---

## Part 6: Package Family Remediation Opportunities

{family_table if family_table else "*No multi-version package families with remediation opportunities identified.*"}

{"**Top Opportunity:** " + family_analysis[0]['family_key'] + f" - consolidating to a clean version could eliminate {family_analysis[0].get('total_finding_instances', 0)} CVE instances." if family_analysis and family_analysis[0].get('total_finding_instances', 0) > 0 else ""}

---

## Part 7: Grading Summary

| Category | Weight | Grade | Score |
|----------|--------|-------|-------|
| PES | 30% | {grades['pes']} | {grade_values[grades['pes']]} |
| Findings Trend | 25% | {grades['findings']} | {grade_values[grades['findings']]} |
| Backlog | 20% | {grades['backlog']} | {grade_values[grades['backlog']]} |
| RPS Trend | 15% | {grades['rps']} | {grade_values[grades['rps']]} |
| Edit Efficiency | 10% | {grades['edits']} | {grade_values[grades['edits']]} |
| **Overall** | 100% | **{overall_grade}** | **{weighted_score:.2f}** |

---

## Part 8: Recommendations

### Critical (0-30 days)

{chr(10).join(f"- {issue}" for issue in critical_issues) if critical_issues else "- No critical issues requiring immediate action"}

### High Priority (30-90 days)

{chr(10).join(f"- {rec}" for rec in recommendations) if recommendations else "- Continue current practices"}

### Medium Priority (90-180 days)

- Review and update dependency policies
- Implement automated vulnerability scanning in CI/CD
- Establish version pinning standards

---

## Conclusion

{"This dataset demonstrates **effective security practices** with positive patch efficacy and controlled vulnerability growth." if overall_grade in ['A', 'B'] else "This dataset shows **areas for improvement** in patch targeting and remediation velocity." if overall_grade == 'C' else "This dataset has **significant security concerns** that require immediate attention and process changes."}

**Bottom Line:** Grade **{overall_grade}** - {
    "Excellent security posture, maintain current practices" if overall_grade == 'A' else
    "Good security posture with minor improvements needed" if overall_grade == 'B' else
    "Acceptable but needs focused improvement" if overall_grade == 'C' else
    "Poor security posture, significant changes required" if overall_grade == 'D' else
    "Critical security failure, emergency intervention needed"
}

---

*Report generated by PatchFox Dataset Analysis Runbook v1.0*
"""

    with open(report_path, 'w') as f:
        f.write(report_content)

    print(f"  Full report written to: {report_path}")
    print()

    return report_path, overall_grade, grades
```

**12.2 Usage Example**

```python
# After completing Steps 1-11, call generate_report with collected data:

# Fetch all required data
metrics = fetch_metrics()  # From Step 1
metrics.sort(key=lambda x: x['commitDateTime'])

edits = fetch_edits()  # From Step 1
findings = fetch_findings()  # From Step 1

# Build composition dict from Step 2
composition = {}  # {type: {count, packages, findings, critical, high}}
# ... populate from datasourceMetricsCurrent analysis

# Build family analysis from Step 8
family_analysis = []  # [{family_key, total_finding_instances, ...}]
# ... populate from SQL query results

# Generate report
report_path, grade, grades = generate_report(
    dataset_name="MyDataset",
    metrics=metrics,
    edits=edits,
    findings=findings,
    composition=composition,
    family_analysis=family_analysis
)

print(f"Analysis complete. Grade: {grade}")
```

**12.3 Sample Terminal Output**

```
======================================================================
  PATCHFOX SECURITY ANALYSIS: nationalsecurityagency
  Generated: 2026-01-19 10:30:45
======================================================================

  OVERALL GRADE: C

  PES: -0.42 (Grade: C)  |  RPS: 12.3 (Increasing, Grade: F)
  Findings: 111 (Increasing +43, Grade: D)
  Backlog: 89 total, 72% in 90+d (Grade: D)
  Edit Efficiency: 34% different-version (Grade: D)

  CRITICAL ISSUES:
    ⚠ Backlog critical: 72% of findings are 90+ days old
    ⚠ 3 CRITICAL severity CVEs present
    ⚠ Findings increased 63% - accumulating debt

  TOP RECOMMENDATIONS:
    1. Consolidate com.fasterxml.jackson.core/jackson-databind - eliminates 47 CVE instances
    2. Prioritize remediation of 90+ day backlog items
    3. Target patches at vulnerable packages, not just version bumps
    4. Implement version pinning to reduce fragmentation

======================================================================
  Full report written to: /tmp/nationalsecurityagency_analysis_2026-01-19.md
```

---

## 6. Common Pitfalls

### ⚠️ CRITICAL GOTCHA #0: Package and Finding Tables Contain Historical Data

**THIS IS THE #1 MISTAKE. DO NOT QUERY THE `package` OR `finding` TABLES DIRECTLY FOR CURRENT STATE ANALYSIS.**

The `package` and `finding` tables are append-only and contain **every package and finding PatchFox has ever seen**, not just what's currently in the dataset.

| Table/Query | What It Contains | Result |
|-------------|------------------|--------|
| `SELECT * FROM package` | **ALL packages EVER seen** (historical) | **WRONG - massively inflated** |
| `SELECT * FROM finding` | **ALL findings EVER seen** (historical) | **WRONG - massively inflated** |
| `dataset_metrics.package_indexes` | Only packages in **CURRENT** state | **CORRECT** |
| `dataset_metrics.finding_indexes` | Only findings in **CURRENT** state | **CORRECT** |

**Example:** A dataset with 3,000 current packages might have 90,000+ historical package records. Querying the package table directly will show 136,006 instances when there are only 251 datasources.

**The ONLY Correct Way:**
```sql
-- ✅ CORRECT: Join with package_indexes for current state
SELECT p.name, COUNT(DISTINCT p.version) as versions
FROM public.dataset_metrics dm,
     unnest(dm.package_indexes) as pkg_id
JOIN public.package p ON p.id = pkg_id
WHERE dm.is_current = true
GROUP BY p.name;

-- ✅ CORRECT: Join with finding_indexes for current state
SELECT COUNT(DISTINCT f.id) as current_findings
FROM public.dataset_metrics dm,
     unnest(dm.finding_indexes) as finding_id
JOIN public.finding f ON f.id = finding_id
WHERE dm.is_current = true;
```

**Checklist:**
- [ ] Am I joining with `dataset_metrics.package_indexes` or `finding_indexes`?
- [ ] Am I filtering `WHERE dm.is_current = true`?
- [ ] Do my numbers make sense? (instances should be ≤ datasource_count)

### ⚠️ CRITICAL GOTCHA #0.5: Package Metrics Are Computed Against Present-Day State

**ALL package downlevel/stale/versions-behind metrics are computed with respect to PRESENT DAY knowledge, not historical state.**

The `package` table stores current metadata:
- `number_versions_behind_head` - how many versions behind **NOW**
- `most_recent_version_published_at` - when the latest version was published (could be yesterday)
- Stale calculations use `NOW()` - interval, not commit datetime

**Impact on Historical Analysis:**
- A commit from 2019 will show packages as "downlevel" or "stale" based on 2026 package index data
- A package that was current in 2019 may show as "5 versions behind" today
- Stale metrics reflect "how old is the latest version today" not "was it stale in 2019"

**This means:**
- ✅ Metrics are accurate for **current/recent** commits
- ⚠️ Metrics are **anachronistic** for **historical** commits (showing present-day state, not historical state)
- ❌ You cannot determine "was this package downlevel at the time of commit" from these metrics alone

**Why:** The package table doesn't store historical version metadata - only current state. True historical analysis would require a time-series package version database.

### ❌ PITFALL #1: Misunderstanding the `sameEdit` Flag

**WRONG:**
"66% of edits are 'same version' re-evaluations with NO version change and ZERO security potential"

**CORRECT:**
"66% of edits have `sameEdit=true`, meaning these changes were made 2+ times in the previous 90 days (coordinated/repeated changes)"

**Key:** The `sameEdit` flag does NOT mean "no version change occurred." It means "this same version change was applied 2+ times recently" (e.g., bulk monorepo upgrades). These edits DO change versions and CAN improve security.

**Example:**
```
Before: datawave-core@7.27.1-SNAPSHOT
After: datawave-core@7.23.6
sameEdit: true (change made 3x in 90 days)
sameEditCount: 3
```

**Why This Matters:** Misinterpreting `sameEdit` leads to wrongly concluding that 66% of patches are useless. In reality, they're coordinated releases that may or may not target security issues.

### ❌ PITFALL #2: Misunderstanding CVE Timing (Two Scenarios)

**WRONG:**
"Findings appeared 574 days after CVE publication - they're slow to detect vulnerabilities"

**CORRECT:**
"There are TWO types of CVE timing scenarios you must distinguish:"

**Scenario A: Old CVEs (Remediation Lag)**
- CVE published: 2021-04-23 (CVE-2021-26291, CRITICAL)
- Package added: Sometime between 2021-2025 (unknown)
- Most recent commit: Nov 18, 2025
- PatchFox scan: Jan 18, 2026
- **Interpretation:** Package with 4.7-year-old CVE still present = **remediation failure**

**Scenario B: Recent CVEs (Normal Lifecycle)**
- Package added: July 2025 (urllib3 2.3.0, safe at the time)
- Most recent commit: Nov 18, 2025 (urllib3 still present)
- CVE published: Jan 7, 2026 (CVE-2026-21441 discovered AFTER package was added)
- PatchFox scan: Jan 18, 2026
- **Interpretation:** New vulnerability in existing package = **normal lifecycle, needs remediation**

**Key:**
- `publishedAt` = when NVD published the CVE (public knowledge date)
- `reportedAt` = when PatchFox detected it in their packages
- Compare `publishedAt` to `commitDateTime` (when package was added) NOT to `reportedAt`
- If CVE published BEFORE commit: remediation lag (bad)
- If CVE published AFTER most recent commit: ongoing discovery (normal, but needs response)

### ❌ PITFALL #3: Confusing Activity with Outcomes

**WRONG:**
"High patch count = good security"

**CORRECT:**
"High patch count + negative PES + growing findings = busy work without security impact"

**Key:** Measure outcomes (PES, findings trend, RPS), not activity (patch count). You can make 20,000 patches and still have worsening security.

### ❌ PITFALL #4: Ignoring Package Growth

**WRONG:**
"Vulnerable % decreased from 2.5% to 1.5%, security improved"

**CORRECT:**
"Vulnerable % decreased, but package count doubled. Absolute vulnerable packages increased 36 → 45 (+25%)"

**Key:** Track both percentage AND absolute counts. Fast package growth can hide security degradation. A lower percentage with more total vulnerabilities is NOT improvement.

### ❌ PITFALL #5: Misinterpreting Negative PES

**WRONG:**
"Negative PES means patches introduce more CVEs"

**CORRECT:**
"Negative PES means patches INCREASE complexity (RPS, downlevel) without reducing vulnerabilities"

**Key:** PES = Patch Impact / Patch Effort. Negative PES means impact is negative (making metrics worse) even though effort is positive (patches are being made). This happens when:
- RPS increases (version fragmentation)
- Findings grow
- Downlevel packages increase
- Stale packages accumulate

### ❌ PITFALL #6: Not Checking Backlog Aging

**WRONG:**
"90 findings is acceptable for a large codebase"

**CORRECT:**
"90 findings with 82 (91%) aged 90+ days means NO remediation velocity - findings accumulate but never close"

**Key:** Backlog aging shows whether vulnerabilities are being addressed or accumulating. If 90%+ are in the 90+ day bucket and the number is growing, there's no remediation happening.

### ❌ PITFALL #7: Assuming Low Vuln % = Good Security

**WRONG:**
"1.39% vulnerable packages is excellent, top-tier security"

**CORRECT:**
"1.39% vulnerable WITH 22 CVEs over 3 years old means packages aren't being upgraded, not that security is excellent"

**Key:** Low vulnerable % combined with ancient CVEs is a RED FLAG. It means:
- Old dependencies not being upgraded
- Possibly infrequent scanning
- Remediation lag, not security success

Look at CVE AGE DISTRIBUTION, not just vulnerable package percentage.

### ❌ PITFALL #8: Not Understanding the Vulnerability Lifecycle

**WRONG:**
"Findings grew from 68 → 111, they must be introducing vulnerable code"

**CORRECT:**
"Findings grew from 68 → 111 because (a) new CVEs are discovered in existing packages over time, AND (b) old CVEs aren't being remediated"

**Key:** Even if you FREEZE all code changes, findings will continue to grow as security researchers discover new vulnerabilities in existing dependencies. This is normal. What matters is:
- **Remediation velocity:** Are old CVEs being fixed?
- **CVE age distribution:** Are most CVEs recent (normal) or ancient (remediation failure)?

**Example:**
- Package `urllib3 2.3.0` added July 2025 (safe)
- November 2025: No code changes
- January 2026: New CVE discovered affecting urllib3 2.3.0
- Finding count increases WITHOUT any new code

This is NOT a failure to avoid vulnerable code - it's the reality of dependency management.

---

## 7. Interpretation Guidelines

### 7.1 PES Interpretation Matrix

| PES Range | Impact | Effort | Meaning | Action |
|-----------|--------|--------|---------|--------|
| +5 to +10 | High positive | Low-Medium | Excellent efficiency | Replicate approach |
| +1 to +5 | Positive | Medium | Good progress | Continue strategy |
| -1 to +1 | Near zero | Low-High | Treading water | Reassess priorities |
| -1 to -5 | Negative | Medium-High | Making worse | Major change needed |
| < -5 | High negative | High | Severe degradation | Emergency intervention |

### 7.2 Findings Trend Interpretation

| Change | Package Growth | Meaning | Grade |
|--------|----------------|---------|-------|
| Decreasing | Any | Active remediation | A |
| Stable | <10% | Keeping pace | B |
| Stable | >30% | Falling behind | D |
| Increasing | <10% | New CVEs detected | C |
| Increasing | >30% | Accumulating debt | F |

### 7.3 Edit Type Interpretation

| Same Version % | Different % | CREATE vs DELETE | Assessment |
|----------------|-------------|------------------|------------|
| <30% | >70% | DELETE > CREATE | Excellent |
| 30-50% | 50-70% | Balanced | Good |
| 50-70% | 30-50% | CREATE > DELETE | Poor |
| >70% | <30% | CREATE >> DELETE | Failing |

### 7.4 Backlog Interpretation

| 90+ Day % | Trend | Meaning | Action |
|-----------|-------|---------|--------|
| <30% | Decreasing | Active resolution | Maintain |
| 30-50% | Stable | Treading water | Increase capacity |
| 50-70% | Increasing | Falling behind | Prioritize remediation |
| >70% | Increasing | Critical failure | Emergency response |

---

## 8. Report Structure Template

### Executive Summary (1 paragraph)

State:
1. Overall grade (A-F)
2. Key finding (PES score)
3. Main problem (e.g., "66% of patches ineffective")
4. Bottom line verdict

### Part 1: Patch Activity

- Total edit count
- Edit type breakdown (UPDATE, CREATE, DELETE)
- Same vs different version breakdown
- Temporal distribution
- Most updated packages

### Part 2: Patch Efficacy

- PES calculation and trend
- RPS trend
- Why positive or negative
- Grade and distribution

### Part 3: Findings & Backlog

- Findings trajectory
- Severity breakdown
- Backlog aging analysis
- CVE age analysis
- Trend interpretation

### Part 4: Root Cause Analysis

- Why patches don't reduce findings
- Why RPS increases/decreases
- Why backlog accumulates/resolves
- Systemic issues identified

### Part 5: Detailed Metrics

- RPS deep dive
- Downlevel/stale analysis
- Package growth correlation
- Edit type patterns

### Part 6: Grading

- Individual component grades
- Overall grade with rationale
- Comparison to industry standards

### Part 7: Recommendations

**Critical (0-30 days):**
- Emergency fixes
- Stop doing harmful things

**High Priority (30-90 days):**
- Structural changes
- New processes

**Medium Priority (90-180 days):**
- Optimization
- Tooling improvements

### Part 8: Conclusion

- What's working
- What's failing
- Bottom line assessment
- Grade justification

---

## 9. Example Analysis Script

Here's a complete Python script that performs the core analysis:

```python
#!/usr/bin/env python3
import json
import requests
from datetime import datetime, timedelta
from collections import Counter

DATA_SERVICE_URL = "http://localhost:1702"
DATASET_NAME = "YOUR_DATASET_NAME"

def fetch_metrics():
    url = f"{DATA_SERVICE_URL}/api/v1/db/datasetMetrics/query"
    params = {'datasetName': DATASET_NAME, 'isCurrent': 'true'}
    response = requests.get(url, params=params, timeout=30)
    response.raise_for_status()
    data = response.json()
    return data['data']['titlePage']['content']

def fetch_edits():
    six_months_ago = (datetime.now() - timedelta(days=180)).strftime('%Y-%m-%dT%H:%M:%SZ')
    url = f"{DATA_SERVICE_URL}/api/v1/db/datasetMetrics/edit/query"
    params = {
        'datasetName': DATASET_NAME,
        'isCurrent': 'true',
        'commitDateTime': f'gte.{six_months_ago}',
        'size': 25000
    }
    response = requests.get(url, params=params, timeout=60)
    response.raise_for_status()
    data = response.json()
    return data['data']['titlePage']['content']

def fetch_findings():
    url = f"{DATA_SERVICE_URL}/api/v1/db/findingData/query"
    params = {'size': 500}
    response = requests.get(url, params=params, timeout=30)
    response.raise_for_status()
    data = response.json()
    return data['data']['titlePage']['content']

def analyze_pes(metrics):
    metrics.sort(key=lambda x: x['commitDateTime'])

    avg_pes = sum(m['patchEfficacyScore'] for m in metrics) / len(metrics)
    avg_impact = sum(m['patchImpact'] for m in metrics) / len(metrics)
    avg_effort = sum(m['patchEffort'] for m in metrics) / len(metrics)

    positive = sum(1 for m in metrics if m['patchEfficacyScore'] > 0)
    negative = sum(1 for m in metrics if m['patchEfficacyScore'] < 0)

    print("=== PES ANALYSIS ===")
    print(f"Average PES: {avg_pes:.2f}")
    print(f"Average Impact: {avg_impact:.2f}")
    print(f"Average Effort: {avg_effort:.2f}")
    print(f"Positive PES: {positive}/{len(metrics)} ({positive/len(metrics)*100:.1f}%)")
    print(f"Negative PES: {negative}/{len(metrics)} ({negative/len(metrics)*100:.1f}%)")

def analyze_edits(edits):
    edit_types = Counter(e['editType'] for e in edits)
    same_edits = sum(1 for e in edits if e['sameEdit'])
    different_edits = len(edits) - same_edits

    print("\n=== EDIT ANALYSIS ===")
    print(f"Total Edits: {len(edits):,}")
    for et, count in edit_types.most_common():
        print(f"  {et}: {count:,} ({count/len(edits)*100:.1f}%)")
    print(f"Same version: {same_edits:,} ({same_edits/len(edits)*100:.1f}%)")
    print(f"Different version: {different_edits:,} ({different_edits/len(edits)*100:.1f}%)")

def analyze_findings_trend(metrics):
    metrics.sort(key=lambda x: x['commitDateTime'])

    print("\n=== FINDINGS TREND ===")
    for m in metrics:
        print(f"{m['commitDateTime'][:10]}: {m['totalFindings']} findings, {m['packages']} packages")

    change = metrics[-1]['totalFindings'] - metrics[0]['totalFindings']
    print(f"\nChange: {change:+d} findings")

def main():
    print(f"Analyzing dataset: {DATASET_NAME}\n")

    metrics = fetch_metrics()
    edits = fetch_edits()
    findings = fetch_findings()

    analyze_pes(metrics)
    analyze_edits(edits)
    analyze_findings_trend(metrics)

    print("\n=== Analysis complete ===")

if __name__ == "__main__":
    main()
```

---

## 10. Checklist

Before finalizing your report, verify:

- [ ] Fetched all required data (metrics, edits, findings)
- [ ] Calculated PES average and distribution
- [ ] Analyzed RPS trend (increasing/decreasing)
- [ ] Analyzed findings trend (increasing/decreasing)
- [ ] Calculated edit type breakdown (same vs different %)
- [ ] Analyzed backlog aging (% in 90+ days)
- [ ] Analyzed CVE ages (oldest vulnerabilities)
- [ ] Tracked package growth correlation
- [ ] Identified root causes (why negative/positive trends)
- [ ] Provided specific recommendations with timelines
- [ ] Assigned component grades and overall grade
- [ ] Verified interpretations against common pitfalls
- [ ] Included data sources and analysis date

---

## Appendix: Quick Reference

### API Endpoints Quick Copy

```bash
# Dataset name
curl -s "http://localhost:1702/api/v1/db/dataset/query"

# Metrics
curl -s "http://localhost:1702/api/v1/db/datasetMetrics/query?datasetName={NAME}&isCurrent=true"

# Edits
curl -s "http://localhost:1702/api/v1/db/datasetMetrics/edit/query?datasetName={NAME}&isCurrent=true&size=25000"

# Findings
curl -s "http://localhost:1702/api/v1/db/findingData/query?size=500"
```

### Grading Scale

| Grade | PES Range | Findings | Backlog | RPS |
|-------|-----------|----------|---------|-----|
| A | >+1 | Decreasing | <30% in 90+d | Decreasing |
| B | 0 to +1 | Stable | 30-50% in 90+d | Stable |
| C | -1 to 0 | Slight increase | 50-70% in 90+d | Slight increase |
| D | -2 to -1 | Increasing | >70% in 90+d | Increasing |
| F | <-2 | Rapidly increasing | >90% in 90+d | Rapidly increasing |

### Key Metrics Interpretation

- **PES < 0:** Patches making things worse
- **RPS increasing:** Version fragmentation
- **Findings increasing:** Accumulating vulnerabilities
- **>70% same version edits:** Wasting effort
- **>70% backlog in 90+ days:** Not remediating
- **CVEs >2 years old:** Technical debt

---

**Document Version:** 2.2
**Last Updated:** February 6, 2026
**Maintainer:** PatchFox Team

---

## Changelog

### v2.2 (February 6, 2026)
- **Common Pitfalls:** Added critical gotcha #0 about `finding` table also containing historical data
  - Both `package` and `finding` tables are append-only and contain ALL historical records
  - Must join with `dataset_metrics.finding_indexes` for current state, not just `package_indexes`
  - Added SQL examples for correct finding queries

### v2.1 (January 20, 2026)
- **Step 1:** Added API pagination support for large datasets (>1000 records)
  - Added pagination scripts for metrics and edits fetching
  - Added Step 1.5 for critical CVE fetching
- **Step 8:** Added REST API alternative for when direct database access is unavailable
  - Added database connection instructions
  - Added RPS-based approximation script
- **Temporal Utilities:** Fixed divide-by-zero in `get_story_arc()` for datasets starting from zero
  - Now finds first non-zero value as baseline for meaningful percentage calculations
  - Added narrative handling for "grew from nothing" scenarios

### v2.0 (January 19, 2026)
- Major refactor to story-driven temporal analysis
- Added Temporal Analysis Utilities section
- Refactored all steps to use narrative framing
- Added Step 12 for report generation
