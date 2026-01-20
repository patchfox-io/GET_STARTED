# Step 1: Initial Data Collection

**Purpose:** Fetch all required data with proper pagination support.

---

## Prerequisites

- Data Service API running at `http://localhost:1702`
- `curl` and `python3` installed

---

## Important: API Pagination

The Data Service API caps results at **1000 records per request**. Large datasets may have:
- 7,000+ metrics records
- 20,000+ edit records

Always use the pagination helper below for complete data.

---

## 1.1 Get Dataset Name

```bash
curl -s "http://localhost:1702/api/v1/db/dataset/query" | python3 -m json.tool | head -30
```

Extract the `name` field from the first dataset.

```bash
export DATASET="your_dataset_name"
```

---

## 1.2 Fetch Dataset Metrics (Current) - WITH PAGINATION

**First, check how many records exist:**

```bash
curl -s "http://localhost:1702/api/v1/db/datasetMetrics/query?datasetName=${DATASET}&isCurrent=true&size=1" | \
  python3 -c "import sys,json; d=json.load(sys.stdin); print(f\"Total records: {d['data']['titlePage']['totalElements']}\")"
```

**For small datasets (<1000 records):**

```bash
curl -s "http://localhost:1702/api/v1/db/datasetMetrics/query?datasetName=${DATASET}&isCurrent=true&sort=commitDateTime.asc&size=1000" -o /tmp/metrics.json
```

**For large datasets (>1000 records), use this pagination script:**

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

---

## 1.3 Fetch All Edits (6 months) - WITH PAGINATION

**Calculate 6 months ago date:**

```python
from datetime import datetime, timedelta
six_months = (datetime.now() - timedelta(days=180)).strftime('%Y-%m-%dT%H:%M:%SZ')
print(f"Date filter: {six_months}")
```

**For large edit datasets, use pagination:**

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

---

## 1.4 Fetch Finding Data

```bash
curl -s "http://localhost:1702/api/v1/db/findingData/query?size=5000" -o /tmp/findings.json
```

---

## 1.5 Fetch Critical CVEs (for emergency check)

```bash
curl -s "http://localhost:1702/api/v1/db/findingData/query?severity=CRITICAL&size=500" -o /tmp/critical_cves.json
```

---

## 1.6 Fetch Datasource Metrics

```bash
curl -s "http://localhost:1702/api/v1/db/datasourceMetricsCurrent/query?size=1000" -o /tmp/dmc.json
```

---

## 1.7 MANDATORY: Validate Findings Count (Granularity Bug Check)

> **⚠️ DO NOT SKIP THIS STEP ⚠️**
>
> There is a known bug where `totalFindings` undercounts by 1.5-2x because it deduplicates across datasources. The backlog metrics count correctly (per-datasource).
>
> **If you skip this step, your entire analysis will use wrong numbers.**

Run this query to get the TRUE findings count:

```sql
docker exec docker-compose-postgres-1 psql -U mr_data -d mrs_db -c "
SELECT
    total_findings as reported_findings,
    (COALESCE(findings_in_backlog_between_thirty_and_sixty_days, 0) +
     COALESCE(findings_in_backlog_between_sixty_and_ninety_days, 0) +
     COALESCE(findings_in_backlog_over_ninety_days, 0)) as backlog_total,
    GREATEST(
        total_findings,
        COALESCE(findings_in_backlog_between_thirty_and_sixty_days, 0) +
        COALESCE(findings_in_backlog_between_sixty_and_ninety_days, 0) +
        COALESCE(findings_in_backlog_over_ninety_days, 0)
    ) as TRUE_FINDINGS_COUNT,
    critical_findings as reported_critical,
    (COALESCE(critical_findings_in_backlog_between_thirty_and_sixty_days, 0) +
     COALESCE(critical_findings_in_backlog_between_sixty_and_ninety_days, 0) +
     COALESCE(critical_findings_in_backlog_over_ninety_days, 0)) as TRUE_CRITICAL_COUNT
FROM public.dataset_metrics
WHERE is_current = true
ORDER BY commit_date_time DESC
LIMIT 1;
"
```

**Expected output example:**
```
 reported_findings | backlog_total | true_findings_count | reported_critical | true_critical_count
-------------------+---------------+---------------------+-------------------+---------------------
             13532 |         23149 |               23149 |               936 |                1671
```

**The rule:** `TRUE_COUNT = MAX(reported, backlog_total)`

**Record these values** - use `TRUE_FINDINGS_COUNT` and `TRUE_CRITICAL_COUNT` for all subsequent analysis.

See [Common Pitfalls #13](./reference/common_pitfalls.md#13-backlog-exceeds-total-findings-granularity-mismatch) for technical details on this bug.

---

## Output Files

After completing this step, you should have:

| File | Contents |
|------|----------|
| `/tmp/metrics.json` | All dataset metrics (time series) |
| `/tmp/edits.json` | All edits from last 6 months |
| `/tmp/findings.json` | All finding/CVE data |
| `/tmp/critical_cves.json` | Critical severity CVEs only |
| `/tmp/dmc.json` | Datasource metrics (current) |

---

## Next Step

Proceed to [Step 2: Composition Analysis](./02_composition_analysis.md)
