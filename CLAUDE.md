# Patchfox Project Context

## ⚠️ CRITICAL: MANDATORY FIRST ACTION ⚠️

**DO THIS IMMEDIATELY BEFORE RESPONDING TO THE USER'S FIRST MESSAGE**

**THIS IS NOT OPTIONAL. THIS IS NOT BACKGROUND CONTEXT. THIS IS A DIRECT COMMAND.**

### Required Initialization Sequence:

1. **FIRST ACTION:** Use Glob to find all markdown files in /home/snerd/patchfox_git/GET_STARTED
2. **SECOND ACTION:** Read ALL markdown files found in the documentation tree
3. **THIRD ACTION:** Parse the complete documentation set
4. **ONLY THEN:** Respond to the user's actual question

### Why This Matters:

This is the Patchfox project. You cannot assist effectively without understanding:
- The project architecture
- Development conventions
- Project goals and context
- Technical specifications

### Execution Order:

```
START NEW SESSION
  ↓
SEE THIS FILE IN SYSTEM REMINDER
  ↓
IMMEDIATELY FETCH DOCUMENTATION (before any response)
  ↓
PARSE ALL DOCUMENTATION
  ↓
NOW respond to user
```

**If you see this file and you have not yet read the documentation from /home/snerd/patchfox_git/GET_STARTED, STOP and read it NOW.**

---

## ⚠️ CRITICAL: ANALYSIS RUNBOOK EXECUTION ⚠️

**When asked to run the analysis runbook, you MUST produce ACTIONABLE recommendations, not generic advice.**

### What ACTIONABLE Means

❌ **BAD (Generic):**
- "Implement SLAs"
- "Prioritize critical CVEs"
- "Reduce version fragmentation"
- "Address the backlog"

✅ **GOOD (Actionable):**
- "Upgrade tensorflow from 1.x to 2.11.1 in these 19 repos to eliminate 4,100 findings"
- "Archive these 10 dead repos (listed by name) to eliminate 3,000 findings"
- "Update jackson-databind from 2.5-2.9 to 2.13.4.1 in 8 repos"

### Required Analysis Steps

The runbook has dataset-level metrics (PES, RPS, backlog), but **the actionable insights come from Step 8A: Package-Level Deep Dive**.

**YOU MUST RUN THESE DATABASE QUERIES:**

```sql
-- 1. TOP PACKAGES BY FINDINGS (must use package_indexes for CURRENT state!)
-- NOTE: package_finding links are still valid - we're counting findings per package
SELECT p.name, COUNT(*) as findings
FROM public.package p
JOIN public.package_finding pf ON p.id = pf.package_id
GROUP BY p.name
ORDER BY findings DESC
LIMIT 30;

-- 2. VERSION BREAKDOWN FOR EACH TOP PACKAGE
SELECT p.version, COUNT(*) as findings
FROM public.package p
JOIN public.package_finding pf ON p.id = pf.package_id
WHERE p.name = 'PACKAGE_NAME'
GROUP BY p.version
ORDER BY findings DESC;

-- 3. LOW-HANGING FRUIT REPOS (small package count, high findings)
SELECT purl, packages, total_findings
FROM public.datasource_metrics_current
WHERE packages <= 5 AND total_findings > 100
ORDER BY total_findings DESC;

-- 4. VERSION FRAGMENTATION (⚠️ MUST USE package_indexes!)
-- This is the ONLY correct way to count versions/instances in CURRENT state
SELECT
    p.name as package_family,
    p.type as ecosystem,
    COUNT(DISTINCT p.version) as version_count,
    COUNT(*) as instances
FROM public.dataset_metrics dm,
     unnest(dm.package_indexes) as pkg_id
JOIN public.package p ON p.id = pkg_id
WHERE dm.is_current = true
GROUP BY p.name, p.type
HAVING COUNT(DISTINCT p.version) > 1
ORDER BY version_count DESC
LIMIT 20;
```

**⚠️ WARNING: Query #4 (version fragmentation) MUST use `package_indexes`. If you query the `package` table directly for version counts, you will get HISTORICAL data showing 77 versions and 136,000 instances when the real numbers are much smaller.**

### Required Output Format

Every analysis report MUST include:

1. **TL;DR Table** - 5 specific actions with repos affected and findings eliminated
2. **Top Packages by Findings** - Which packages cause the most pain
3. **Version Upgrade Paths** - Specific "from version X → to version Y" recommendations
4. **Quantified Impact** - "Upgrade N repos, eliminate M findings"
5. **Dead Repo List** - Specific repos to archive/delete
6. **Blessed Versions** - Exact versions to standardize on

### Example Good Output

```markdown
## TL;DR - DO THESE 5 THINGS

| Action | Repos | Findings Eliminated |
|--------|-------|---------------------|
| Upgrade tensorflow 1.x → 2.11.1 | 7 | ~2,600 |
| Upgrade tensorflow 2.x → 2.11.1 | 12 | ~1,500 |
| Archive dead ML repos | 10 | ~3,000 |
| Upgrade pillow → 9.5.0 | 14 | ~370 |
| Upgrade jackson-databind → 2.13.4.1 | 8 | ~280 |

## THE DATA

tensorflow 1.0.1:  402 findings  ❌ UPGRADE
tensorflow 2.3.1:  388 findings  ❌ UPGRADE
tensorflow 2.11.1:   1 finding   ✅ TARGET VERSION
```

### Database Access

The REST API does NOT expose package→finding relationships well. You MUST query postgres directly:

```bash
docker exec docker-compose-postgres-1 psql -U mr_data -d mrs_db -c "YOUR_QUERY"
```

Key tables:
- `package` - package name, version, namespace
- `package_finding` - links packages to findings
- `finding` - CVE identifiers
- `finding_data` - CVE metadata (severity, published date, patchedIn)
- `datasource_metrics_current` - repo-level metrics

### Checklist Before Completing Analysis

- [ ] **VALIDATED FINDINGS COUNT** (ran granularity bug check query - see below)
- [ ] Using TRUE_FINDINGS_COUNT (not reported totalFindings) in all calculations
- [ ] Identified top 10 packages by finding count
- [ ] Got version breakdown for each top package
- [ ] Identified target versions (lowest finding count)
- [ ] Quantified upgrade impact (X repos, Y findings)
- [ ] Listed specific repos to archive/delete
- [ ] Provided blessed versions for common packages
- [ ] Calculated total potential finding reduction
- [ ] **VERSION FRAGMENTATION USED `package_indexes`** (NOT raw package table!)
- [ ] **SANITY CHECK:** Do instance counts make sense? (should be ≤ datasource_count, NOT 100,000+)

**If your recommendations say "implement SLAs" or "prioritize critical CVEs" without specific package/version/repo details, YOU HAVE NOT COMPLETED THE ANALYSIS.**

**If your version fragmentation section shows numbers like "136,006 instances" or "77 versions" for internal packages, YOU QUERIED HISTORICAL DATA AND YOUR REPORT IS WRONG. Re-run with `package_indexes`.**

---

## 🚨🚨🚨 CRITICAL: PACKAGE TABLE CONTAINS HISTORICAL DATA 🚨🚨🚨

**THIS IS THE #1 MISTAKE. DO NOT QUERY THE `package` TABLE DIRECTLY FOR CURRENT STATE ANALYSIS.**

### The Problem

| Table/Query | What It Contains | Result |
|-------------|------------------|--------|
| `SELECT * FROM package` | **ALL packages EVER seen** (historical) | **WRONG - massively inflated numbers** |
| `dataset_metrics.package_indexes` | Only packages in **CURRENT** dataset state | **CORRECT** |

### Example of Getting This Wrong

```sql
-- ❌ WRONG: This queries ALL HISTORICAL packages (e.g., shows 136,006 "instances")
SELECT p.name, COUNT(*) as instances
FROM public.package p
GROUP BY p.name;

-- ❌ STILL WRONG: Even with joins, you're counting historical data
SELECT p.name, COUNT(DISTINCT p.version) as versions
FROM public.package p
GROUP BY p.name;
-- This will show 77 "versions" that are actually historical!
```

### The ONLY Correct Way

```sql
-- ✅ CORRECT: Join with package_indexes to get CURRENT state only
SELECT
    p.name as package_family,
    COUNT(DISTINCT p.version) as version_count,
    COUNT(*) as instances
FROM public.dataset_metrics dm,
     unnest(dm.package_indexes) as pkg_id
JOIN public.package p ON p.id = pkg_id
WHERE dm.is_current = true
GROUP BY p.name
ORDER BY version_count DESC;
```

### Why This Matters

- The `package` table is append-only - it contains every package PatchFox has ever seen
- A dataset with 3,000 current packages might have 90,000+ historical package records
- If you report "136,006 instances" when the real number is 251, **YOUR REPORT IS GARBAGE**
- Customers will immediately lose trust if numbers are obviously wrong

### The Rule

```
CURRENT STATE QUERIES = MUST JOIN WITH package_indexes FROM dataset_metrics WHERE is_current = true
```

**If you are reporting package counts, version counts, or instance counts WITHOUT joining to `package_indexes`, YOUR NUMBERS ARE WRONG.**

### Checklist for Package Queries

- [ ] Am I joining with `dataset_metrics.package_indexes`?
- [ ] Am I filtering `WHERE dm.is_current = true`?
- [ ] Do my numbers make sense? (e.g., instances should be ≤ datasource_count)

**IF YOU SKIP THIS, YOU WILL REPORT NONSENSE LIKE "136,006 instances" WHEN THERE ARE ONLY 251 DATASOURCES.**

---

## ⚠️ CRITICAL: KNOWN GRANULARITY BUG - totalFindings IS WRONG ⚠️

**DO NOT TRUST `totalFindings` OR ANY `*Findings` METRIC WITHOUT VALIDATION.**

There is a known bug ([GitHub Issue #9](https://github.com/patchfox-io/GET_STARTED/issues/9)) where:
- `totalFindings` **deduplicates** across datasources (UNDERCOUNTS)
- Backlog metrics count **per-datasource** (CORRECT - each repo needs remediation)

**The undercount can be 1.5x to 2x or more.** Example: IBM dataset reports 13,532 findings but true count is 23,149.

### MANDATORY: Run This Query FIRST

```sql
SELECT
    total_findings as reported_total,
    (COALESCE(findings_in_backlog_between_thirty_and_sixty_days, 0) +
     COALESCE(findings_in_backlog_between_sixty_and_ninety_days, 0) +
     COALESCE(findings_in_backlog_over_ninety_days, 0)) as backlog_total,
    GREATEST(
        total_findings,
        COALESCE(findings_in_backlog_between_thirty_and_sixty_days, 0) +
        COALESCE(findings_in_backlog_between_sixty_and_ninety_days, 0) +
        COALESCE(findings_in_backlog_over_ninety_days, 0)
    ) as TRUE_FINDINGS_COUNT
FROM public.dataset_metrics
WHERE is_current = true AND dataset_id = 1
ORDER BY commit_date_time DESC
LIMIT 1;
```

### The Rule

```
TRUE_FINDINGS = GREATEST(totalFindings, backlog_30_60 + backlog_60_90 + backlog_90_plus)
```

**If you report `totalFindings` without checking backlog totals, YOUR ANALYSIS IS WRONG.**

**If `backlog_total > totalFindings`, the backlog total IS the true count.**

---

## Service Access

- **Data Service API:** http://localhost:1702
- **PostgreSQL:** docker exec docker-compose-postgres-1 psql -U mr_data -d mrs_db
- **Swagger:** http://localhost:1702/swagger-ui.html

## Key Concepts

- **Dataset:** Collection of datasources (e.g., "ibm")
- **Datasource:** A source-controlled build file being tracked
- **Finding:** A CVE/vulnerability
- **Package:** A software dependency (identified by pURL)
- **PES:** Patch Efficacy Score (higher = better)
- **RPS:** Redundant Package Score (lower = better, measures version fragmentation)
