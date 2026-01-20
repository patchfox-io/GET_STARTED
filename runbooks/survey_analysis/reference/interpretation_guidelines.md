# Interpretation Guidelines

How to read and interpret PatchFox analysis data.

---

## Reading Time Series Data

### The Story Framework

Every metric tells a story with:
1. **Beginning** - Where we started
2. **Journey** - How we got here (chapters, turning points)
3. **Current** - Where we are now
4. **Velocity** - How fast things are changing

### Key Questions for Any Metric

| Question | What to Look At |
|----------|-----------------|
| Where did we start? | `story_arc.beginning` |
| Where are we now? | `story_arc.current` |
| Is it improving or degrading? | `story_arc.journey.narrative` |
| Is change accelerating or slowing? | `velocity.velocity_story` |
| When did things change? | `turning_points`, `notable_events` |
| What distinct phases occurred? | `chapters` |

---

## Metric-Specific Guidelines

### PES (Patch Efficacy Score)

**Higher is better**

| Value | Interpretation |
|-------|----------------|
| > +5 | Excellent - patches highly effective |
| +1 to +5 | Good - positive security impact |
| -1 to +1 | Neutral - patches not moving the needle |
| -1 to -5 | Poor - patches may be hurting |
| < -5 | Failing - significant negative impact |

**Key insight:** PES near 0 for extended periods means patches aren't targeting vulnerabilities effectively.

---

### RPS (Redundant Package Score)

**Lower is better**

| Value | Interpretation |
|-------|----------------|
| 0-5 | Excellent - well-consolidated versions |
| 5-15 | Good - some fragmentation |
| 15-25 | Concerning - significant fragmentation |
| > 25 | Severe - major version chaos |

**Key insight:** High RPS means the same package exists in many versions across your codebase, increasing maintenance burden and potential for version-specific vulnerabilities.

---

### Findings

**Lower is better**

Context matters:
- **Findings up, packages up** → Growth, not necessarily bad
- **Findings up, packages stable** → New CVEs discovered (discovery)
- **Findings down, packages stable** → Remediation working
- **Findings down, packages down** → Cleanup/removal

**Severity mix matters:**
- Critical+High % increasing = wrong prioritization
- Critical declining faster than total = good prioritization

---

### Backlog

**Lower is better, especially 90+ bucket**

| 90+ % | Interpretation |
|-------|----------------|
| < 30% | Healthy - findings being addressed |
| 30-50% | Acceptable - some aging |
| 50-70% | Poor - remediation stalled |
| > 70% | Critical - findings aging in place |

**Key insight:** Backlog aging tells you about remediation velocity, not just volume.

---

### Edit Behavior

**Lower same% is better**

| Same % | Interpretation |
|--------|----------------|
| < 30% | Efficient - targeted patches |
| 30-50% | Acceptable - some churn |
| 50-70% | Concerning - significant churn |
| > 70% | Wasteful - mostly no-impact patches |

**Key insight:** Same-version edits have ZERO security benefit - they cannot fix vulnerabilities.

---

## Cross-Metric Patterns

### Pattern: High Activity, No Progress
- High edit volume
- High same% edits
- PES near zero
- Backlog stable or growing

**Interpretation:** Organization is busy but not effective. Patches aren't targeting real issues.

### Pattern: Growing Attack Surface
- Packages increasing
- CREATE >> DELETE
- RPS increasing
- Findings increasing

**Interpretation:** Uncontrolled growth. Need dependency management policy.

### Pattern: Technical Debt Accumulation
- Backlog 90+ % increasing
- CVE ages increasing
- Edit volume declining
- Same% increasing

**Interpretation:** Giving up on remediation. Urgent intervention needed.

### Pattern: Effective Security Program
- PES positive and stable
- Findings declining
- Backlog declining
- RPS declining
- CVE ages declining

**Interpretation:** Security practices working. Continue and optimize.

---

## Quick Reference Table

| Metric | Good | Concerning | Critical |
|--------|------|------------|----------|
| PES | > +1 | -1 to +1 | < -1 |
| RPS | < 10 | 10-20 | > 20 |
| Vulnerable % | < 5% | 5-15% | > 15% |
| Backlog 90+ | < 30% | 30-70% | > 70% |
| Same Edit % | < 30% | 30-70% | > 70% |
| Critical CVE Age | < 90d | 90d-1yr | > 1yr |
