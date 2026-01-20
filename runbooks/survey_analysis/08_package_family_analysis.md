# Step 8: Package Family Remediation Analysis

**Purpose:** Identify consolidation opportunities - how many CVEs could be eliminated by standardizing on versions you already use?

---

## Prerequisites

- Completed [Step 1: Data Collection](./01_data_collection.md)
- Either: Direct database access OR `/tmp/metrics.json` for approximation

---

## Database Access Required

This step's detailed analysis requires direct PostgreSQL access for precise package family queries.

**Database Connection:**
```bash
# Default local connection
psql -h localhost -p 5432 -U patchfox -d patchfox

# Or via docker
docker exec -it patchfox-postgres psql -U patchfox -d patchfox
```

---

## REST API Alternative (No DB Access Required)

If direct database access is unavailable, use this approximation:

```python
import json

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
```

---

## 8.1 Calculate Aggregate Remediation Potential (DB Required)

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

---

## 8.2 Identify Specific Consolidation Opportunities (DB Required)

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

---

## 8.3 Grading

| pct_eliminable | Grade |
|----------------|-------|
| > 50% | Excellent opportunity |
| 25-50% | Good opportunity |
| < 25% | Limited quick wins |

---

## Output

- Aggregate consolidation opportunity percentage
- Top package families for consolidation
- Estimated CVE reduction from standardization

---

## Next Step

Proceed to [Step 9: CVE Age Analysis](./09_cve_age_analysis.md)
