# Step 8A: Package-Level Deep Dive (CRITICAL)

**Purpose:** Identify the specific packages causing the most findings and provide actionable upgrade paths.

**This step produces the ACTIONABLE recommendations. Do not skip it.**

---

## Prerequisites

- PostgreSQL access: `docker exec -it docker-compose-postgres-1 psql -U mr_data -d mrs_db`
- Completed Steps 1-8

---

## ⚠️ CRITICAL: Current Dataset vs Historical Data ⚠️

**The `package` table contains ALL packages ever seen by PatchFox (historical), not just packages in the current dataset state.**

| Table/Field | Contains | Use For |
|-------------|----------|---------|
| `package` table (all rows) | ALL historical packages (e.g., 90,000+) | Finding-to-package joins |
| `dataset_metrics.package_indexes` | Only CURRENT packages (e.g., 3,172) | Current state analysis |

### The Mistake to Avoid

```sql
-- ❌ WRONG: Queries ALL historical packages
SELECT p.name, COUNT(DISTINCT p.version) as versions
FROM public.package p
GROUP BY p.name;
-- This might show 1,400+ "versions" that are actually historical package IDs!

-- ✅ CORRECT: Queries only packages in CURRENT dataset
SELECT p.name, COUNT(DISTINCT p.version) as versions
FROM public.dataset_metrics dm,
     unnest(dm.package_indexes) as pkg_id
JOIN public.package p ON p.id = pkg_id
WHERE dm.is_current = true
GROUP BY p.name;
-- This shows actual version fragmentation in the current state
```

**If you report version counts without joining to `package_indexes`, your numbers will be wildly inflated.**

---

## Why This Step Matters

The dataset-level metrics tell you "you have 13,000 findings." This step tells you:
- **Which 5 packages** cause 45% of those findings
- **Which versions** are vulnerable vs clean
- **Exactly what to upgrade to** and the expected impact

---

## 8A.1 Identify Top Finding Contributors

```sql
-- THE MOST IMPORTANT QUERY: Which packages cause the most findings?
-- CRITICAL: Must use datasource.package_indexes to get CURRENT packages only
WITH current_packages AS (
    SELECT 
        unnest(d.package_indexes) as pkg_id,
        d.purl as datasource
    FROM public.datasource d
),
package_findings AS (
    SELECT 
        p.name,
        p.version,
        p.id as package_id,
        COUNT(DISTINCT pf.finding_id) as unique_findings,
        COUNT(*) as finding_instances
    FROM current_packages cp
    JOIN public.package p ON p.id = cp.pkg_id
    LEFT JOIN public.package_finding pf ON pf.package_id = p.id
    GROUP BY p.name, p.version, p.id
)
SELECT 
    name as package,
    COUNT(DISTINCT package_id) as versions,
    SUM(unique_findings) as total_unique_findings,
    SUM(finding_instances) as total_instances
FROM package_findings
WHERE unique_findings > 0
GROUP BY name
ORDER BY total_unique_findings DESC
LIMIT 30;
```

**Expected output:** A list showing packages like `tensorflow: 1654 unique findings across 5 versions`

**Action:** Note the top 10 packages - these are your remediation targets.

**Key fields:**
- `total_unique_findings`: Distinct CVEs (what matters for remediation)
- `total_instances`: How many times those CVEs appear (indicates duplication)
- `versions`: Number of package versions in use

---

## 8A.2 Version Analysis for Top Packages

For each top package, run:

```sql
-- Get version breakdown for a specific package
-- CRITICAL: Must use datasource.package_indexes to get CURRENT packages only
WITH current_packages AS (
    SELECT unnest(d.package_indexes) as pkg_id
    FROM public.datasource d
)
SELECT 
    p.version,
    COUNT(DISTINCT pf.finding_id) as unique_findings,
    COUNT(*) as finding_instances
FROM current_packages cp
JOIN public.package p ON p.id = cp.pkg_id
LEFT JOIN public.package_finding pf ON pf.package_id = p.id
WHERE p.name = 'PACKAGE_NAME_HERE' AND pf.finding_id IS NOT NULL
GROUP BY p.version
ORDER BY unique_findings DESC;
```

**What you're looking for:**
- Old versions with HIGH findings (upgrade FROM these)
- New versions with LOW findings (upgrade TO these)

**Example output:**
```
tensorflow 1.12:    399 findings  ← UPGRADE FROM
tensorflow 1.15.0:  397 findings  ← UPGRADE FROM
tensorflow 2.7.0:   187 findings  ← UPGRADE TO (or newer)
```
tensorflow 2.3.1:   388 findings  ← UPGRADE FROM
tensorflow 2.11.1:    1 finding   ← UPGRADE TO
```

---

## 8A.3 Version Group Impact Analysis

Group versions to quantify upgrade impact:

```sql
-- Example for TensorFlow - adapt for other packages
SELECT
    CASE
        WHEN version LIKE '1.%' THEN 'v1.x (OLD)'
        WHEN version LIKE '2.0%' OR version LIKE '2.1%' OR version LIKE '2.2%' OR version LIKE '2.3%' THEN 'v2.0-2.3'
        WHEN version LIKE '2.4%' OR version LIKE '2.5%' OR version LIKE '2.6%' OR version LIKE '2.7%' THEN 'v2.4-2.7'
        WHEN version LIKE '2.8%' OR version LIKE '2.9%' OR version LIKE '2.10%' OR version LIKE '2.11%' THEN 'v2.8+ (CURRENT)'
        ELSE 'Other'
    END as version_group,
    COUNT(DISTINCT id) as repo_count,
    SUM(finding_count) as total_findings
FROM (
    SELECT p.id, p.version, COUNT(*) as finding_count
    FROM public.package p
    JOIN public.package_finding pf ON p.id = pf.package_id
    WHERE p.name = 'tensorflow'
    GROUP BY p.id, p.version
) sub
GROUP BY version_group
ORDER BY total_findings DESC;
```

**This tells you:** "7 repos on TensorFlow 1.x = 2,776 findings. Upgrading saves ~2,600 findings."

---

## 8A.4 Identify Low-Hanging Fruit Repos

Find repos with tiny package counts but massive findings:

```sql
-- Repos that are easy to clean up
SELECT
    SPLIT_PART(SPLIT_PART(d.purl, '/', 3), '@', 1) as repo_name,
    d.packages,
    d.total_findings,
    d.critical_findings,
    ROUND(d.total_findings::numeric / NULLIF(d.packages, 0), 1) as findings_per_pkg
FROM public.datasource_metrics_current d
WHERE d.packages > 0
  AND d.packages <= 5
  AND d.total_findings > 100
ORDER BY d.total_findings DESC
LIMIT 20;
```

**These are your quick wins:** Repos with 1-5 packages but 200+ findings. Often dead/demo projects that can be archived.

---

## 8A.5 Ecosystem Risk Analysis

```sql
-- Which ecosystem is causing the most pain?
SELECT
    SPLIT_PART(d.purl, '@', 2) as ecosystem,
    COUNT(*) as repo_count,
    SUM(d.packages) as total_packages,
    SUM(d.total_findings) as total_findings,
    ROUND(SUM(d.total_findings)::numeric / NULLIF(SUM(d.packages), 0), 2) as findings_per_package
FROM public.datasource_metrics_current d
WHERE d.packages > 0
GROUP BY ecosystem
ORDER BY total_findings DESC;
```

**Look for:** Ecosystems with high `findings_per_package` ratio - these are disproportionately risky.

---

## 8A.6 Version Fragmentation Analysis (RPS Deep Dive)

Analyze which packages have multiple versions in the **current** dataset state:

```sql
-- ✅ CORRECT: Version fragmentation in CURRENT dataset only
-- Must join with package_indexes from dataset_metrics!
SELECT
    p.name as package_family,
    p.type as ecosystem,
    COUNT(DISTINCT p.version) as version_count,
    COUNT(*) as instances,
    array_agg(DISTINCT p.version ORDER BY p.version) as versions
FROM public.dataset_metrics dm,
     unnest(dm.package_indexes) as pkg_id
JOIN public.package p ON p.id = pkg_id
WHERE dm.is_current = true
GROUP BY p.name, p.type
HAVING COUNT(DISTINCT p.version) > 1
ORDER BY version_count DESC
LIMIT 20;
```

**Expected output:**
```
 package_family      | ecosystem | version_count | instances | versions
---------------------+-----------+---------------+-----------+------------------
 datawave-query-core | maven     |            77 |    136006 | {7.14.1,7.23.10,...}
 spring-core         | maven     |             3 |      1847 | {5.3.21,5.3.39,6.1.0}
```

**Interpretation:**
- `version_count`: Number of DISTINCT versions of this package in current state
- `instances`: How many times packages with this name appear across all datasources
- High `version_count` = consolidation opportunity (contributes to RPS score)

**DO NOT** run this query without the `package_indexes` join - you'll get inflated historical counts!

---

## 8A.7 Build the Upgrade Matrix

For each top package, document:

| Package | Vulnerable Versions | Target Version | Repos Affected | Findings Eliminated |
|---------|--------------------| ---------------|----------------|---------------------|
| tensorflow | 1.x, 2.0-2.7 | 2.11.1 | 19 | ~4,100 |
| pillow | 5.x-8.x | 9.5.0 | 14 | ~370 |
| jackson-databind | 2.5-2.9 | 2.13.4.1 | 8 | ~280 |

---

## 8A.8 Identify Dead Repos for Archival

```sql
-- Repos with high findings but likely abandoned
SELECT
    SPLIT_PART(SPLIT_PART(d.purl, '/', 3), '@', 1) as repo_name,
    SPLIT_PART(d.purl, '@', 2) as ecosystem,
    d.packages,
    d.total_findings,
    d.commit_date_time as last_commit
FROM public.datasource_metrics_current d
WHERE d.total_findings > 100
  AND d.packages < 10
ORDER BY d.total_findings DESC
LIMIT 20;
```

**Recommendation:** Review these repos. If last commit > 1 year ago, consider archiving.

---

## Output

This step should produce:

1. **Top 10 packages by findings** with version breakdown
2. **Specific upgrade paths** (from version X to version Y)
3. **Quantified impact** (upgrade N repos, eliminate M findings)
4. **Quick win repos** to archive/delete
5. **Ecosystem risk ranking**

---

## Example Actionable Output

```
ACTION 1: UPGRADE TENSORFLOW
- 7 repos on 1.x (2,776 findings) → upgrade to 2.11.1
- 5 repos on 2.4-2.7 (948 findings) → upgrade to 2.11.1
- Expected result: Eliminate ~4,100 findings

ACTION 2: ARCHIVE DEAD REPOS
- RuDaS__experiments: 1 package, 402 findings → ARCHIVE
- tensorflow-hangul-recognition: 1 package, 218 findings → ARCHIVE
- Expected result: Eliminate ~3,000 findings instantly

ACTION 3: CREATE BLESSED REQUIREMENTS
- tensorflow>=2.11.1
- pillow>=9.5.0
- Propagate to all 28 high-risk pypi repos
```

---

## Red Flags

- **Single package > 20% of findings:** Version upgrade is critical
- **findings_per_package > 10:** Ecosystem needs attention
- **Repo with <5 packages but >200 findings:** Likely dead, archive it
- **Same vulnerable package in 10+ repos:** Need blessed version policy

---

## Next Step

Proceed to [Step 9: CVE Age Analysis](./09_cve_age_analysis.md)

---

*This step is CRITICAL for producing actionable recommendations. The dataset-level metrics tell you there's a problem. This step tells you exactly how to fix it.*
