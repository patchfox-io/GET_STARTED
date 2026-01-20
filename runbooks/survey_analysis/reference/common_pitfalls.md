# Common Pitfalls

Mistakes to avoid when performing dataset security analysis.

---

## Data Collection Pitfalls

### 1. Not Using Pagination
**Problem:** API returns max 1000 records per request. Missing records skews analysis.

**Solution:** Always check `totalElements` and paginate if > 1000.

### 2. Wrong Date Filter
**Problem:** Using wrong date format or timezone causes missing data.

**Solution:** Use ISO 8601 format: `2025-07-01T00:00:00Z`

### 3. Forgetting `isCurrent=true`
**Problem:** Getting historical snapshots instead of current state.

**Solution:** Always include `isCurrent=true` for current analysis.

---

## Metric Interpretation Pitfalls

### 4. Misunderstanding `sameEdit`
**Problem:** Thinking `sameEdit: true` means "no version change."

**Reality:** `sameEdit` means this same version change was applied 2+ times recently (e.g., bulk monorepo upgrades). These edits DO change versions.

**What you want:** Focus on `samePatches` vs `differentPatches` in the metrics, not `sameEdit` in edit records.

### 5. Confusing Discovery vs Remediation
**Problem:** Findings increased = "we're doing worse"

**Reality:** Findings can increase from:
- New CVEs discovered in existing packages (discovery)
- Adding new packages with existing CVEs (growth)

**Solution:** Look at packages change alongside findings change.

### 6. Ignoring Trajectory
**Problem:** Only looking at current values, not trends.

**Reality:** A dataset at PES -1 but improving is better than one at PES +1 but declining.

**Solution:** Always consider velocity and direction, not just absolute values.

---

## Analysis Pitfalls

### 7. Treating All Ecosystems Equally
**Problem:** Assuming all ecosystems have same vulnerability density.

**Reality:** pypi often has 6x higher vulnerability density than maven.

**Solution:** Analyze by ecosystem type and weight accordingly.

### 8. Not Accounting for Dataset Age
**Problem:** Comparing a 1-month dataset to a 3-year dataset.

**Reality:** Longer history = more accumulated findings, more data points.

**Solution:** Normalize for time when comparing datasets.

### 9. Ignoring Backlog Aging
**Problem:** Only tracking total backlog, not age distribution.

**Reality:** 100 findings in 30-60 day bucket is much better than 100 in 90+ day bucket.

**Solution:** Always analyze backlog by age bucket, especially 90+.

---

## Reporting Pitfalls

### 10. Grade Without Context
**Problem:** Reporting "Grade: D" without explaining why.

**Solution:** Always include the story - what drove the grade, what can improve it.

### 11. Missing Root Cause
**Problem:** Listing symptoms without explaining causes.

**Reality:** "Findings are high" is a symptom. "69% same-version patches" is a cause.

**Solution:** Connect observations to behaviors that drove them.

### 12. Unrealistic Recommendations
**Problem:** "Fix all critical CVEs" without acknowledging constraints.

**Solution:** Prioritize recommendations, acknowledge effort vs impact.

---

## Known Bugs

### 13. Backlog Exceeds Total Findings (Granularity Mismatch)

**Problem:** `backlog_90` (or other backlog buckets) exceeds `totalFindings`, which appears mathematically impossible.

**Root Cause:** There is a known bug ([GitHub Issue #9](https://github.com/patchfox-io/GET_STARTED/issues/9)) where `totalFindings` and backlog metrics count at different granularities:

- `totalFindings` counts unique (package_id, finding_id) pairs (deduplicated)
- Backlog counts (datasource, package_id, finding_id) tuples (per datasource occurrence)

When the same package (e.g., `lodash@4.17.4`) appears in multiple datasources (repos), its findings are counted once in `totalFindings` but once-per-datasource in backlog.

**Example from IBM dataset:**
- `pkg:npm/path-to-regexp@0.1.7` has 2 CVEs and appears in 273 datasources
- Contribution to `totalFindings`: **2** (deduplicated)
- Contribution to backlog: **546** (273 × 2)

**Workaround:** When `backlog_total > totalFindings`, use the **LARGER** value (backlog) as the true finding count, since per-datasource counting is the intended business logic (each repo needs separate remediation).

```python
def get_true_findings_count(metrics_record):
    """Get the true findings count, accounting for the granularity bug."""
    total = metrics_record.get('totalFindings', 0) or 0
    backlog_30 = metrics_record.get('findingsInBacklogBetweenThirtyAndSixtyDays', 0) or 0
    backlog_60 = metrics_record.get('findingsInBacklogBetweenSixtyAndNinetyDays', 0) or 0
    backlog_90 = metrics_record.get('findingsInBacklogOverNinetyDays', 0) or 0
    backlog_total = backlog_30 + backlog_60 + backlog_90

    # Use the larger value - backlog counts at correct granularity
    return max(total, backlog_total)
```

**Similarly for severity buckets:** If `criticalFindingsInBacklog* > criticalFindings`, use the backlog sum.

---

## Red Flags Checklist

Stop and investigate if you see:

| Red Flag | Possible Issue |
|----------|----------------|
| PES consistently 0 | No impact data, or patches not targeting CVEs |
| RPS > 30 | Severe fragmentation, may indicate no standardization |
| 90+ backlog > 80% | Remediation completely stalled |
| Same% > 80% | Massive churn with zero security value |
| CVEs > 5 years old | Abandoned remediation or blocked fixes |
| Findings spiking | Major discovery event or new vulnerable dependency |
| Backlog > Total Findings | Known bug - see pitfall #13, use larger value |
