# PatchFox Analyze-Service Performance Telemetry Runbook

This runbook explains how to measure and predict analyze-service processing performance based on dataset characteristics.

## Reference System Specifications

**This analysis was performed on:**
- **CPU**: Intel Xeon E3-1505M v6 @ 3.00GHz (4 cores, 8 threads, max 4.0GHz)
- **Memory**: 32GB RAM
- **Disk**: NVMe SSD (934GB)
- **OS**: Ubuntu 22.04.5 LTS (native Linux, not virtualized)
- **Docker**: Running natively on Linux (not Docker Desktop)

**Performance will vary on different hardware.** The formula coefficients are specific to this system. On different hardware (especially virtualized environments like Docker Desktop on macOS), you'll need to recalculate the formula using the same methodology.

## Overview

Analyze-service processing time scales with three primary factors:
1. **Packages** - Total unique packages in the dataset (from `dataset_metrics.packages`)
2. **Findings** - Total security findings in the dataset (from `dataset_metrics.total_findings`)
3. **Datasources** - Number of unique datasources that have contributed to the dataset up to that point (count of distinct `datasource_metrics.purl` where `commit_date_time <= current_record_commit_date_time`)

The relationship is:
```
time = 0.000149521×packages - 0.001413×findings + 0.000000420×findings² + 0.001833×datasources + 0.32
```

Or simplified:
```
time = packages/6688 + findings²/2379423 - findings/708 + datasources/546 + 0.32
```

### Why This Formula?

The **findings² term** (quadratic) dominates at scale because of hidden algorithmic complexity:

**The backlog calculation** (in `TabulateService.java`) iterates through each current finding and then searches across all datasources that contain that specific CVE to find when the package+CVE combination first appeared. This is **O(findings × datasources_per_CVE)** complexity.

As the dataset grows:
- More findings accumulate
- More datasources contribute to the dataset
- Each CVE appears in more datasources over time
- The product of (findings × datasources_per_CVE) grows quadratically

The regression captures this as a findings² term because datasource count correlates with dataset growth (more commits processed = more datasources = more CVEs spread across datasources).

**At 200k packages, 20k findings, 3000 datasources (on this system):**
- Packages contribute: 29.9s (linear scaling)
- Findings² contribute: 168.1s (quadratic - the bottleneck)
- Datasources contribute: 5.5s (linear scaling)
- **Total: ~176 seconds per record**

## Prerequisites

- Running PatchFox pipeline with analyze-service
- JMX enabled on analyze-service (port 9010)
- jmxterm.jar downloaded to /tmp
- Access to postgres database

## Step 1: Collect Sample Data

### 1.1 Get Processing Times from Logs

Extract processing duration for every 50th record:

```bash
docker logs docker-compose-analyze-service-1 2>&1 | \
  grep "record processing duration" | \
  awk '{print NR, $NF}' | \
  sed 's/PT//;s/S//' | \
  awk 'NR % 50 == 1 {print $1, $2}'
```

**Example output:**
```
1 0.343646916
51 0.552064349
101 0.567907802
151 1.401803771
201 2.092872916
...
801 12.361370275
851 14.133382784
901 16.128702133
```

### 1.2 Get Commit Datetimes for Sample Records

Extract commit datetimes for the same records:

```bash
docker logs docker-compose-analyze-service-1 2>&1 | \
  grep "for commit datetime.*totalPacakgeTypeCount" | \
  awk 'NR % 50 == 1 {print NR, $0}' | \
  awk '{print $1, $11}'
```

**Example output:**
```
1 2016-05-13T15:31:13Z
51 2018-02-17T00:02:30Z
101 2018-05-04T14:24:18Z
...
```

### 1.3 Query Dataset Metrics

Get packages, findings, and datasource counts for each sample:

```sql
SELECT 
  dm.commit_date_time::date as date,
  dm.packages,
  dm.total_findings,
  (SELECT COUNT(DISTINCT dsm.purl) 
   FROM datasource_metrics dsm 
   WHERE dsm.commit_date_time <= dm.commit_date_time) as datasource_count
FROM dataset_metrics dm
WHERE dm.commit_date_time IN (
  '2016-05-13T15:31:13Z',
  '2018-02-17T00:02:30Z',
  '2018-05-04T14:24:18Z',
  '2018-10-11T07:42:39.004Z',
  '2019-02-04T18:16:01Z',
  '2019-05-30T16:03:30.006Z',
  '2019-07-11T19:19:04.015Z',
  '2019-07-22T17:01:38.018Z',
  '2019-08-14T14:47:20Z',
  '2019-09-08T12:32:11Z',
  '2019-10-16T17:49:10Z',
  '2019-12-18T20:57:26Z',
  '2020-02-25T21:33:06Z',
  '2020-05-05T14:12:59Z',
  '2020-06-24T20:46:46.008Z',
  '2020-07-17T20:02:44Z',
  '2020-08-21T07:09:17.009Z',
  '2020-10-06T09:08:38.015Z',
  '2020-11-17T06:20Z'
)
ORDER BY dm.commit_date_time;
```

Run via docker:
```bash
docker exec docker-compose-postgres-1 psql -U mr_data mrs_db -c "SELECT ..."
```

## Step 2: Calculate Regression Coefficients

Use Python with numpy to calculate the optimal formula:

```python
import numpy as np

# Data format: (packages, findings, datasources, actual_time_seconds)
data = [
    (5, 8, 1, 0.34),
    (1781, 280, 50, 0.55),
    (13513, 848, 150, 1.40),
    (19004, 1159, 200, 2.09),
    (23553, 2453, 350, 4.26),
    (25548, 3030, 494, 4.83),
    (30859, 4029, 592, 6.69),
    (34340, 4311, 692, 7.99),
    (43100, 5077, 792, 12.36),
    (50786, 5607, 842, 14.13),
    (54003, 5693, 892, 16.13)
]

# Build feature matrix: packages, findings, findings², datasources, constant
X = np.array([[p, f, f**2, d, 1] for p, f, d, _ in data])
y = np.array([t for _, _, _, t in data])

# Solve using least squares
coefficients = np.linalg.lstsq(X, y, rcond=None)[0]
a, b, c, d, e = coefficients

print(f"Optimal Formula:")
print(f"time = {a:.9f}×packages + {b:.6f}×findings + {c:.9f}×findings² + {d:.6f}×datasources + {e:.2f}")
print(f"\nSimplified:")
print(f"time = packages/{1/a:.0f} + findings²/{1/c:.0f} - findings/{abs(1/b):.0f} + datasources/{1/d:.0f} + {e:.2f}")

# Test prediction
test_packages = 200000
test_findings = 20000
test_datasources = 3000
predicted = a * test_packages + b * test_findings + c * (test_findings**2) + d * test_datasources + e
print(f"\nAt {test_packages:,} packages, {test_findings:,} findings, {test_datasources:,} datasources:")
print(f"Predicted time: {predicted:.1f} seconds ({predicted/60:.1f} minutes)")

# Show fit quality
print("\n| Packages | Findings | DS | Actual | Projected | Δ |")
print("|----------|----------|-----|--------|-----------|---|")
for p, f, ds, actual in data:
    proj = a * p + b * f + c * (f**2) + d * ds + e
    delta = proj - actual
    print(f"| {p:6,d}   | {f:5,d}    | {ds:3d} | {actual:6.2f} | {proj:9.2f} | {delta:+6.2f} |")

# Calculate R²
y_pred = np.array([a * p + b * f + c * (f**2) + d * ds + e for p, f, ds, _ in data])
ss_res = np.sum((y - y_pred) ** 2)
ss_tot = np.sum((y - np.mean(y)) ** 2)
r2 = 1 - (ss_res / ss_tot)
print(f"\nR² score: {r2:.4f} (1.0 = perfect fit)")
```

## Step 3: Monitor Heap Usage

### 3.1 Download jmxterm (if not already installed)

```bash
wget https://github.com/jiaqi/jmxterm/releases/download/v1.0.4/jmxterm-1.0.4-uber.jar -O /tmp/jmxterm.jar
```

### 3.2 Check Current Heap Usage

```bash
echo "get -b java.lang:type=Memory HeapMemoryUsage" | \
  java -jar /tmp/jmxterm.jar -l localhost:9010 -n 2>/dev/null
```

**Example output:**
```
HeapMemoryUsage = { 
  committed = 4180000000;
  init = 524288000;
  max = 12884901888;
  used = 2600000000;
};
```

**Key metrics:**
- **used**: Actual memory in use (objects in heap)
- **committed**: Memory reserved from OS
- **max**: Hard limit (-Xmx setting)

### 3.3 Monitor Heap Over Time

```bash
for i in {1..20}; do
  heap=$(echo "get -b java.lang:type=Memory HeapMemoryUsage" | \
    java -jar /tmp/jmxterm.jar -l localhost:9010 -n 2>/dev/null | \
    grep -E "used|committed")
  used=$(echo "$heap" | grep "used" | awk '{print $3}' | tr -d ';')
  committed=$(echo "$heap" | grep "committed" | awk '{print $3}' | tr -d ';')
  echo "$(date +%H:%M:%S) - Used: $((used/1024/1024))MB, Committed: $((committed/1024/1024))MB"
  sleep 2
done
```

### 3.4 Correlate Heap Spikes with Cache Operations

Check for Redis cache save/load events:

```bash
docker logs docker-compose-analyze-service-1 2>&1 | \
  grep -E "Saving caches to Redis|Loaded.*cache from Redis" | \
  tail -10
```

**Pattern:** Heap spikes (committed expanding to 4-8GB) typically occur during Redis cache serialization/deserialization between page processing.

## Step 4: Analyze Performance Bottlenecks

### 4.1 Check Step-by-Step Timing

```bash
docker logs docker-compose-analyze-service-1 2>&1 | \
  grep -E "duration:|Processing [0-9]+ (packages|current package-level findings)" | \
  tail -50
```

**Key steps to monitor:**
- `tally backlog findings duration` - Backlog calculation time
- `Processing X packages for dataset metrics` - Package index tabulation time
- `record processing duration` - Total time per record

### 4.2 Identify Dominant Step

At scale (50k+ packages, 5k+ findings):
- **Backlog calculation**: ~1.5-2.0 seconds (optimized with inverted indexes)
- **Package processing**: ~8-12 seconds (batch size 10,000)
- **Other operations**: ~0.5-1.0 seconds

## Example Performance Data

From IBM dataset processing (February 2026):

| Record | Packages | Findings | Datasources | Actual Time (s) | Projected Time (s) | Δ |
|--------|----------|----------|-------------|-----------------|-------------------|---|
| 1      | 5        | 8        | 1           | 0.34            | 0.31              | -0.03 |
| 51     | 1,781    | 280      | 50          | 0.55            | 0.31              | -0.24 |
| 151    | 13,513   | 848      | 150         | 1.40            | 1.72              | +0.32 |
| 201    | 19,004   | 1,159    | 200         | 2.09            | 2.45              | +0.36 |
| 351    | 23,553   | 2,453    | 350         | 4.26            | 3.54              | -0.72 |
| 501    | 25,548   | 3,030    | 494         | 4.83            | 4.62              | -0.21 |
| 601    | 30,859   | 4,029    | 592         | 6.69            | 7.15              | +0.46 |
| 701    | 34,340   | 4,311    | 692         | 7.99            | 8.44              | +0.45 |
| 801    | 43,100   | 5,077    | 792         | 12.36           | 11.87             | -0.49 |
| 851    | 50,786   | 5,607    | 842         | 14.13           | 14.74             | +0.61 |
| 901    | 54,003   | 5,693    | 892         | 16.13           | 15.60             | -0.53 |

**R² = 0.9932** (excellent fit)

## Performance Optimizations Applied

### Version 0.0.86 - Backlog Calculation Optimization
**Change:** Replaced O(findings × commits × datasources) nested loops with inverted index approach.

**Implementation:** Build three indexes upfront:
- `Map<CVE, Map<Datasource, FirstSeen>>` - CVEs by datasource
- `Map<Package, Map<Datasource, FirstSeen>>` - Packages by datasource  
- `Map<CVE, FirstSeen>` - HISTORICAL CVEs

**Impact:** Reduced backlog calculation from 20+ seconds to ~1.5-2.0 seconds at 2,500 findings.

### Version 0.0.87 - Batch Size Increase
**Change:** Increased package processing batch size from 500 to 2,500.

**Configuration:** `patchfox.tabulate.package-index-batch-size=2500` in application.properties

**Impact:** Reduced stored procedure calls from 15 to 3 per 7,500 packages, saving ~2 seconds per record.

### Version 0.0.88 - Further Batch Size Optimization
**Change:** Increased batch size to 10,000.

**Configuration:** `patchfox.tabulate.package-index-batch-size=10000`

**Impact:** Minimal additional improvement (~0.2s) - diminishing returns beyond 10k batch size.

**Total Improvement:** 20 seconds → 3.8 seconds per record at the reference point (81% reduction)

## Projections

### At 200k Packages, 20k Findings, 3000 Datasources

**Predicted processing time: 175.6 seconds (2.9 minutes) per record**

**Breakdown:**
- Packages contribution: 29.9s
- Findings² contribution: 168.1s (dominant factor)
- Findings linear: -28.2s (negative coefficient)
- Datasources contribution: 5.5s
- Constant: 0.3s

### Job Completion Estimate

For 17,331 records with gradual growth to target scale:
- Average processing time: ~90-100 seconds per record
- **Total time: 457 hours = 19 days**

## Known Bottlenecks

### 1. Backlog Calculation - O(findings × datasources_per_CVE)

**Current implementation:** For each current finding, iterate through all datasources that have that CVE to find earliest package+CVE combination.

**Complexity:** As both findings and datasources grow, this becomes quadratic-like behavior.

**Evidence:** The findings² term in the regression formula captures this hidden O(n×d) complexity.

**Future optimization needed:** 
- Cache datasource lookups across findings with same CVE
- Limit datasource iteration to top N most relevant
- Pre-compute package+CVE combinations in database

### 2. Package Index Tabulation - O(packages / batch_size)

**Current implementation:** Calls stored procedure `TABULATE_PACKAGE_INDEX_DATA_BATCHED` in batches.

**Batch size:** 10,000 (configurable via `patchfox.tabulate.package-index-batch-size`)

**Constraint:** Postgres parameter limits prevent larger batch sizes.

**At 200k packages:** ~20 stored procedure calls per record, taking ~30 seconds total.

### 3. Redis Cache Serialization

**Observation:** Heap committed memory spikes to 4-8GB during cache save/load operations between pages.

**Cause:** Java serialization creates full object graphs in memory.

**Current mitigation:** Increased max heap to 12GB to provide headroom.

**Future optimization:** Consider Kryo, MessagePack, or Protobuf for more efficient serialization.

## Monitoring Commands

### Check Analyze-Service Version
```bash
docker logs docker-compose-analyze-service-1 2>&1 | grep "Starting App v"
```

### Check Current Processing Progress
```bash
docker exec docker-compose-postgres-1 psql -U mr_data mrs_db -c \
  "SELECT MAX(id) as latest_record, MAX(packages) as max_packages, MAX(total_findings) as max_findings FROM dataset_metrics;"
```

### Monitor Real-Time Processing
```bash
docker logs -f docker-compose-analyze-service-1 2>&1 | \
  grep --line-buffered "record processing duration"
```

### Check Peristalsis Status
```bash
curl http://127.0.0.1:1707/api/v1/peristalsis
```

## Troubleshooting

### Processing Time Suddenly Increases

**Check:** Has the dataset grown significantly in packages/findings/datasources?

```bash
docker exec docker-compose-postgres-1 psql -U mr_data mrs_db -c \
  "SELECT packages, total_findings FROM dataset_metrics ORDER BY id DESC LIMIT 1;"
```

### Out of Memory Errors

**Check:** Current heap usage approaching max?

```bash
echo "get -b java.lang:type=Memory HeapMemoryUsage" | \
  java -jar /tmp/jmxterm.jar -l localhost:9010 -n 2>/dev/null
```

**Solution:** Increase max heap in docker-compose.yml:
```yaml
JAVA_TOOL_OPTIONS: "-Xmx16g ..."
```

### Slow Performance on macOS

**Issue:** Docker volumes on macOS are virtualized, causing 5-10x slowdown for database-heavy workloads.

**Solution:** Run Postgres natively on macOS:
```bash
brew install postgresql@16
brew services start postgresql@16
createdb mrs_db
# Update docker-compose to point to host.docker.internal:5432
```

**Expected improvement:** 140s → 10-15s per record on M4 MacBook.

## Configuration Reference

### Analyze-Service Configuration (application.properties)

```properties
# Package processing batch size (default: 500)
patchfox.tabulate.package-index-batch-size=10000

# Redis cache page size (default: 50 records)
io.patchfox.analyze-service.max-tabulate-cache-size=50
```

### Docker Compose JVM Settings

```yaml
analyze-service:
  environment:
    JAVA_TOOL_OPTIONS: "-Xmx12g -XX:StartFlightRecording=duration=300s,filename=/tmp/profile.jfr -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=9010 ..."
```

**Key JVM flags:**
- `-Xmx12g` - Maximum heap size
- `-Dcom.sun.management.jmxremote.port=9010` - Enable JMX on port 9010

## References

- [Dataset Analysis Runbook](../dataset_analysis_runbook.md)
- [Data Service API Guide](../data_service_api.md)
- [Dataset Metrics Entity](../entities/datasetMetrics.md)
- [Datasource Metrics Entity](../entities/datasourceMetrics.md)
