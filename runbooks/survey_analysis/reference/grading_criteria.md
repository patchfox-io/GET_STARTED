# Grading Criteria

This document consolidates all grading criteria used in the PatchFox Dataset Security Analysis.

---

## Overall Grade

Calculated from the average of all individual metric grades (GPA):

| GPA | Overall Grade |
|-----|---------------|
| >= 3.5 | A |
| 2.5 - 3.49 | B |
| 1.5 - 2.49 | C |
| 0.5 - 1.49 | D |
| < 0.5 | F |

**Grade Point Values:** A=4, B=3, C=2, D=1, F=0

---

## PES (Patch Efficacy Score)

Considers both current state AND trajectory:

| Grade | Criteria |
|-------|----------|
| A | Current PES > 1 AND improving or stable |
| B | Current PES 0-1 AND improving, OR PES > 1 but declining |
| C | Current PES -1 to 0, OR positive but rapidly declining |
| D | Current PES -1 to -2, OR near-zero with no improvement trend |
| F | Current PES < -2, OR any negative PES with worsening trend |

---

## RPS (Redundant Package Score)

| Grade | Criteria |
|-------|----------|
| A | RPS < 5 AND declining or stable |
| B | RPS 5-15 AND declining, OR RPS < 5 but growing slowly |
| C | RPS 5-15 stable, OR 15-25 declining |
| D | RPS 15-25 stable or growing, OR any RPS with accelerating growth |
| F | RPS > 25, OR any RPS with rapid growth trend |

---

## Findings

| Grade | Criteria |
|-------|----------|
| A | Findings declining AND critical findings declining faster |
| B | Findings stable or slowly declining |
| C | Findings slowly growing but critical stable/declining |
| D | Findings growing AND critical growing |
| F | Rapid findings growth OR critical findings exploding |

---

## Backlog

Based on percentage of backlog in 90+ day bucket:

| Grade | Criteria |
|-------|----------|
| A | 90+ % < 30% AND declining |
| B | 90+ % 30-50% OR declining from higher |
| C | 90+ % 50-70% stable |
| D | 90+ % 50-70% growing |
| F | 90+ % > 70% |

---

## Edit Behavior

Based on same-version edit percentage:

| Grade | Criteria |
|-------|----------|
| A | Same% < 30% AND declining trend AND CREATE ~ DELETE |
| B | Same% 30-50% OR Same% < 30% but CREATE > DELETE |
| C | Same% 50-66% with stable trend |
| D | Same% > 66% OR Same% increasing significantly |
| F | Same% > 80% OR CREATE >> DELETE with high volume |

---

## CVE Age

Based on oldest CRITICAL CVE age:

| Grade | Criteria |
|-------|----------|
| A | Oldest critical < 90 days |
| B | Oldest critical 90-180 days |
| C | Oldest critical 180-365 days |
| D | Oldest critical 1-2 years |
| F | Oldest critical > 2 years |

---

## Attack Surface

Based on vulnerable and downlevel package percentages:

| Grade | Vulnerable % | Downlevel % |
|-------|--------------|-------------|
| A | < 5% | < 20% |
| B | < 10% | < 40% |
| C | < 15% | < 60% |
| D | < 25% | any |
| F | >= 25% | any |

---

## Package Family Remediation

Based on percentage eliminable via consolidation:

| pct_eliminable | Assessment |
|----------------|------------|
| > 50% | Excellent opportunity |
| 25-50% | Good opportunity |
| < 25% | Limited quick wins |
