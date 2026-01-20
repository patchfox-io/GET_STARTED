# Temporal Analysis Utilities

These Python utilities extract the story from your time series data.

## Overview

The temporal analysis utilities provide functions to:
- Extract narrative arcs from time series data
- Identify chapters (distinct phases)
- Find turning points and notable events
- Analyze velocity of change
- Compare periods

## Core Functions

| Function | Purpose |
|----------|---------|
| `parse_metrics()` | Load and sort metrics chronologically |
| `extract_time_series()` | Extract a single metric as time series |
| `get_story_arc()` | Extract beginning, middle, current state |
| `identify_chapters()` | Break into distinct phases |
| `find_turning_points()` | Find trend direction changes |
| `find_notable_events()` | Find anomalies |
| `analyze_velocity()` | Calculate rate of change |
| `compare_periods()` | Compare current vs previous period |
| `tell_metric_story()` | Generate complete narrative |
| `print_metric_story()` | Pretty print the story |

---

## Complete Code

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
                f"{ch['character'].upper()} phase, {ch['start_value']:.1f} -> {ch['end_value']:.1f}"
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
            print(f"    * {ch}")
        print()

    print(f"  RECENT TREND:")
    print(f"    {story['period_comparison']['story']}")
    print(f"    Velocity: {story['velocity']['interpretation']}")
    print()

    if story.get('investigate'):
        print(f"  KEY EVENTS TO INVESTIGATE:")
        for item in story['investigate']:
            print(f"    {item['date']}: {item['event']}")
            print(f"       -> {item['prompt']}")
        print()

    arc = story['story_arc']
    print(f"  JOURNEY SUMMARY:")
    print(f"    Start:   {arc['beginning']['description']}")
    print(f"    Current: {arc['current']['description']}")
    print(f"    Change:  {arc['journey']['total_change']:+.1f} ({arc['journey']['total_change_pct']:+.1f}%)")
    print()
```

---

## Usage Example

```python
# Load metrics
metrics = parse_metrics('/tmp/metrics.json')

# Extract time series for a specific metric
pes_series = extract_time_series(metrics, 'patchEfficacyScore')

# Tell the complete story
pes_story = tell_metric_story(pes_series, "Patch Efficacy Score (PES)", lower_is_better=False)

# Print formatted output
print_metric_story(pes_story)
```
