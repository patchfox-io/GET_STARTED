# PatchFox Dataset Security Analysis Runbook

**Version:** 2.2
**Last Updated:** January 20, 2026

---

## ⚠️ STOP: READ THIS BEFORE DOING ANYTHING ⚠️

### Known Bug: `totalFindings` Undercounts (Often by 1.5-2x)

There is a **critical granularity bug** ([GitHub Issue #9](https://github.com/patchfox-io/GET_STARTED/issues/9)):

| Metric | How It Counts | Result |
|--------|---------------|--------|
| `totalFindings` | Deduplicates across datasources | **UNDERCOUNTS** |
| Backlog totals | Counts per-datasource | **CORRECT** |

**The backlog count is correct** because each repo needs individual remediation.

### Your FIRST Query Must Be:

```sql
SELECT
    total_findings as REPORTED,
    (COALESCE(findings_in_backlog_between_thirty_and_sixty_days, 0) +
     COALESCE(findings_in_backlog_between_sixty_and_ninety_days, 0) +
     COALESCE(findings_in_backlog_over_ninety_days, 0)) as BACKLOG_TOTAL,
    GREATEST(total_findings,
        COALESCE(findings_in_backlog_between_thirty_and_sixty_days, 0) +
        COALESCE(findings_in_backlog_between_sixty_and_ninety_days, 0) +
        COALESCE(findings_in_backlog_over_ninety_days, 0)
    ) as TRUE_COUNT
FROM public.dataset_metrics
WHERE is_current = true
ORDER BY commit_date_time DESC LIMIT 1;
```

### The Rule

```
TRUE_FINDINGS = MAX(totalFindings, backlog_30 + backlog_60 + backlog_90)
```

**If you skip this validation, your entire analysis will be wrong.**

See [Common Pitfalls #13](./reference/common_pitfalls.md#13-backlog-exceeds-total-findings-granularity-mismatch) for details.

---

## Purpose

This runbook provides a systematic methodology for analyzing the security posture and patching effectiveness of any PatchFox dataset. Follow the step-by-step guides to produce a comprehensive, story-driven assessment.

## The PatchFox Advantage

PatchFox doesn't just show you where you are - it shows you **how you got here**. Every metric is a time series that tells a story: the beginning state, the journey, the inflection points, and the current chapter.

When you see 100 findings, the question isn't "is 100 good or bad?" - it's:
- How did we get to 100?
- What happened along the way?
- What does that journey tell us about what's working?

This runbook treats your data as a **narrative** - identifying chapters, turning points, and the organizational behaviors that drove change.

---

## Prerequisites

### Required Access
- Data Service API: `http://localhost:1702`
- Orchestrate Service: `http://localhost:1707` (for peristalsis control)
- PostgreSQL (optional, for Step 8 detailed analysis)

### Required Knowledge
- [Key Metrics & Definitions](./reference/metrics_definitions.md)
- [Temporal Analysis Utilities](./reference/temporal_utilities.md)

### Tools Needed
- `curl` for API queries
- `python3` for data analysis
- `jq` or Python for JSON processing

---

## Analysis Steps

| Step | Name | Description | Doc |
|------|------|-------------|-----|
| 1 | Data Collection | Fetch all required data with pagination | [01_data_collection.md](./01_data_collection.md) |
| 2 | Composition Analysis | Analyze dataset by ecosystem type | [02_composition_analysis.md](./02_composition_analysis.md) |
| 3 | PES Analysis | Patch Efficacy Score story | [03_pes_analysis.md](./03_pes_analysis.md) |
| 4 | RPS Analysis | Version fragmentation story | [04_rps_analysis.md](./04_rps_analysis.md) |
| 5 | Findings Analysis | Vulnerability story | [05_findings_analysis.md](./05_findings_analysis.md) |
| 6 | Backlog Analysis | Remediation velocity story | [06_backlog_analysis.md](./06_backlog_analysis.md) |
| 7 | Edit Analysis | Patching behavior story | [07_edit_analysis.md](./07_edit_analysis.md) |
| 8 | Package Family Analysis | Consolidation opportunities | [08_package_family_analysis.md](./08_package_family_analysis.md) |
| **8A** | **Package Deep Dive** | **CRITICAL: Specific upgrade paths** | [08a_package_deep_dive.md](./08a_package_deep_dive.md) |
| 9 | CVE Age Analysis | Technical debt story | [09_cve_age_analysis.md](./09_cve_age_analysis.md) |
| **9A** | **Shadow Findings Analysis** | **Zero-day exposure window** | [09a_shadow_findings_analysis.md](./09a_shadow_findings_analysis.md) |
| 10 | Package Growth | Attack surface story | [10_package_growth_analysis.md](./10_package_growth_analysis.md) |
| 11 | Synthesis | Combine all stories | [11_synthesis.md](./11_synthesis.md) |
| 12 | Report Generation | Generate final report | [12_report_generation.md](./12_report_generation.md) |

---

## Reference Documentation

| Document | Description |
|----------|-------------|
| [Metrics Definitions](./reference/metrics_definitions.md) | PES, RPS, Findings, Backlog definitions |
| [Temporal Utilities](./reference/temporal_utilities.md) | Python functions for story extraction |
| [Grading Criteria](./reference/grading_criteria.md) | How to grade each metric |
| [Common Pitfalls](./reference/common_pitfalls.md) | Mistakes to avoid |
| [Interpretation Guidelines](./reference/interpretation_guidelines.md) | How to read the data |

---

## Quick Start

```bash
# 1. Get dataset name
curl -s "http://localhost:1702/api/v1/db/dataset/query" | python3 -m json.tool | head -30

# 2. Set your dataset
export DATASET="your_dataset_name"

# 3. Follow Step 1 to collect data
# 4. Run through Steps 2-11
# 5. Generate report with Step 12
```

---

## Output

A comprehensive report with:
- Overall security grade (A-F)
- Patch efficacy assessment
- Findings and backlog trends
- Root cause analysis
- Prioritized recommendations

---

## Red Flags (Stop and Investigate)

- **PES < 0:** Patches making things worse
- **RPS increasing:** Version fragmentation growing
- **Findings increasing:** Accumulating vulnerabilities
- **>70% same version edits:** Wasted effort
- **>70% backlog in 90+ days:** Not remediating
- **CVEs >2 years old:** Technical debt

---

## Changelog

See [CHANGELOG.md](./CHANGELOG.md) for version history.
