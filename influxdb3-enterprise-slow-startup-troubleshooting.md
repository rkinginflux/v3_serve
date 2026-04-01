# InfluxDB 3 Enterprise - Slow Startup Time Troubleshooting Guide

## Overview

InfluxDB 3 Enterprise startup time can be affected by several factors, primarily related to loading Write-Ahead Log (WAL) snapshots and catalog synchronization. This guide explains the causes of slow startup times and provides solutions to improve performance.

---

## Primary Cause: WAL Snapshot Loading

The biggest factor affecting startup time is **loading snapshots on startup**.

### How Snapshots Accumulate

Without checkpointing, the server must load individual snapshots on startup. The number of snapshots is determined by:

```
Number of Snapshots = gen1-lookback-duration / gen1-duration
```

With default settings:
- `gen1-lookback-duration`: 1 month (~30 days = 43,200 minutes)
- `gen1-duration`: 10 minutes
- **Result: Up to ~4,320 snapshots to load on startup**

### Impact on Startup Time

| Scenario | Approximate Snapshots | Startup Impact |
|----------|----------------------|----------------|
| Default settings (1 month lookback, 10min gen1) | ~4,320 | Slow |
| 24h lookback, 10min gen1 | ~144 | Moderate |
| 1 month lookback with checkpointing | 1-2 per month | Fast |

---

## Configuration Options Affecting Startup Time

### Snapshot and WAL Settings

| Option | Environment Variable | Default | Impact on Startup |
|--------|---------------------|---------|-------------------|
| `--checkpoint-interval` | `INFLUXDB3_CHECKPOINT_INTERVAL` | Disabled | **Most important** - enables checkpoint consolidation |
| `--gen1-lookback-duration` | `INFLUXDB3_GEN1_LOOKBACK_DURATION` | `24h` | Longer duration = more snapshots to load |
| `--gen1-duration` | `INFLUXDB3_GEN1_DURATION` | `10m` | Shorter duration = more snapshots per time period |
| `--wal-snapshot-size` | `INFLUXDB3_WAL_SNAPSHOT_SIZE` | `600` | Affects snapshot frequency |
| `--wal-replay-concurrency-limit` | `INFLUXDB3_WAL_REPLAY_CONCURRENCY_LIMIT` | `max(num_cpus, 10)` | Lower value = slower replay |
| `--wal-replay-fail-on-error` | `INFLUXDB3_WAL_REPLAY_FAIL_ON_ERROR` | `false` | If true, stops on first error |
| `--snapshotted-wal-files-to-keep` | `INFLUXDB3_NUM_WAL_FILES_TO_KEEP` | `300` | More files = more to process |

### Memory and Resource Settings

| Option | Environment Variable | Default | Impact on Startup |
|--------|---------------------|---------|-------------------|
| `--exec-mem-pool-bytes` | `INFLUXDB3_EXEC_MEM_POOL_BYTES` | `20%` | Insufficient memory slows loading |
| `--num-cores` | `INFLUXDB3_ENTERPRISE_NUM_CORES` | All available | Fewer cores = slower concurrent operations |
| `--datafusion-num-threads` | `INFLUXDB3_DATAFUSION_NUM_THREADS` | Auto | Affects parallel processing |

### Cluster Settings

| Option | Environment Variable | Default | Impact on Startup |
|--------|---------------------|---------|-------------------|
| `--catalog-sync-interval` | `INFLUXDB3_ENTERPRISE_CATALOG_SYNC_INTERVAL` | `10s` | Affects initial catalog sync |
| `--wait-for-running-ingestor` | `INFLUXDB3_ENTERPRISE_WAIT_FOR_RUNNING_INGESTOR` | `10s` | Adds wait time in cluster mode |

### Caching Settings

| Option | Environment Variable | Default | Impact on Startup |
|--------|---------------------|---------|-------------------|
| `--preemptive-cache-age` | `INFLUXDB3_PREEMPTIVE_CACHE_AGE` | `3d` | Cache warming adds startup time |
| `--parquet-mem-cache-size` | `INFLUXDB3_PARQUET_MEM_CACHE_SIZE` | `20%` | Larger cache = more to initialize |

---

## Solutions to Improve Startup Time

### Solution 1: Enable Checkpointing (Highly Recommended)

Checkpointing is the **most effective way** to reduce startup time. It consolidates snapshot metadata into periodic checkpoints.

```bash
influxdb3 serve \
  --checkpoint-interval 1h \
  --node-id=node-01 \
  --cluster-id=prod-cluster \
  --object-store=file \
  --data-dir=/var/lib/influxdb3
```

**How checkpointing helps:**
- Without checkpointing: Load up to ~4,320 individual snapshots
- With checkpointing: Load 1-2 checkpoints per calendar month, then only snapshots since last checkpoint
- Up to 10 checkpoints load concurrently during startup
- Server retains two checkpoints per calendar month

**Recommended checkpoint intervals:**
| Workload | Interval |
|----------|----------|
| High write volume | `30m` |
| Standard workload | `1h` |
| Low write volume | `2h` |

### Solution 2: Increase WAL Replay Concurrency

Increase the number of concurrent operations during WAL replay:

```bash
influxdb3 serve \
  --wal-replay-concurrency-limit 20 \
  ...
```

**Considerations:**
- Default: `max(num_cpus, 10)`
- Higher values speed up replay but use more memory
- Setting too high can cause OOM (out of memory) errors
- Recommended: Start with `num_cpus` and adjust based on available memory

### Solution 3: Reduce Lookback Duration

If your use case allows, reduce the lookback duration:

```bash
influxdb3 serve \
  --gen1-lookback-duration 12h \
  ...
```

**Trade-offs:**
| Duration | Snapshots (10min gen1) | Consideration |
|----------|------------------------|---------------|
| `24h` | ~144 | Good for most use cases |
| `12h` | ~72 | Faster startup, less historical data in buffer |
| `6h` | ~36 | Fastest, but limited lookback |

### Solution 4: Increase Gen1 Duration

Create fewer, larger snapshots:

```bash
influxdb3 serve \
  --gen1-duration 30m \
  ...
```

**Trade-offs:**
| Duration | Snapshots per Day | Consideration |
|----------|-------------------|---------------|
| `10m` (default) | 144 | More granular, more snapshots |
| `30m` | 48 | Balanced |
| `1h` | 24 | Fewer snapshots, less granular |

**Supported values:** `1m`, `5m`, `10m`

### Solution 5: Optimize Object Store Access

For cloud object stores, consider:

```bash
# Increase connection limit for faster parallel downloads
influxdb3 serve \
  --object-store-connection-limit 32 \
  --object-store-http2-only \
  ...
```

| Option | Default | Recommendation |
|--------|---------|----------------|
| `--object-store-connection-limit` | `16` | Increase to `32` or `64` for faster startup |
| `--object-store-http2-only` | disabled | Enable for better multiplexing |
| `--object-store-max-retries` | varies | Reduce for faster failure detection |

### Solution 6: Use Local Object Store Cache

For remote object stores, use a local cache endpoint:

```bash
influxdb3 serve \
  --object-store s3 \
  --bucket my-bucket \
  --object-store-cache-endpoint http://localhost:9000 \
  ...
```

---

## Optimized Configuration Examples

### Fast Startup Configuration (Recommended)

```bash
influxdb3 serve \
  --node-id=node-01 \
  --cluster-id=prod-cluster \
  --object-store=s3 \
  --bucket=my-bucket \
  --checkpoint-interval=1h \
  --wal-replay-concurrency-limit=16 \
  --gen1-lookback-duration=24h \
  --gen1-duration=10m \
  --object-store-connection-limit=32
```

### Environment Variable Configuration

```bash
# Checkpoint settings
export INFLUXDB3_CHECKPOINT_INTERVAL=1h

# WAL settings
export INFLUXDB3_WAL_REPLAY_CONCURRENCY_LIMIT=16
export INFLUXDB3_GEN1_LOOKBACK_DURATION=24h
export INFLUXDB3_GEN1_DURATION=10m

# Object store optimization
export INFLUXDB3_OBJECT_STORE_CONNECTION_LIMIT=32

# Start server
influxdb3 serve \
  --node-id=node-01 \
  --cluster-id=prod-cluster \
  --object-store=s3 \
  --bucket=my-bucket
```

### Kubernetes/Docker Configuration

```yaml
env:
  - name: INFLUXDB3_CHECKPOINT_INTERVAL
    value: "1h"
  - name: INFLUXDB3_WAL_REPLAY_CONCURRENCY_LIMIT
    value: "16"
  - name: INFLUXDB3_GEN1_LOOKBACK_DURATION
    value: "24h"
  - name: INFLUXDB3_OBJECT_STORE_CONNECTION_LIMIT
    value: "32"
```

---

## Other Contributing Factors

### Infrastructure Factors

| Factor | Impact | Mitigation |
|--------|--------|------------|
| **Object Store Latency** | Remote stores (S3, GCS, Azure) are slower than local file storage | Use local cache or file storage for dev/test |
| **Network Bandwidth** | Affects snapshot download speed | Ensure adequate bandwidth, use same-region storage |
| **Disk I/O Speed** | Slow disks delay file operations | Use SSD/NVMe storage |
| **Available Memory** | Insufficient memory causes swapping | Allocate adequate memory |
| **CPU Cores** | Fewer cores = slower concurrent operations | Use machines with more cores |

### Data Factors

| Factor | Impact | Mitigation |
|--------|--------|------------|
| **Number of Databases** | More catalog entries = longer sync | Consolidate databases if possible |
| **Number of Tables** | Affects catalog loading time | Use appropriate table design |
| **Data Volume** | More data = more snapshots | Enable checkpointing |
| **Schema Complexity** | Complex schemas take longer to load | Optimize schema design |

### Cluster Factors

| Factor | Impact | Mitigation |
|--------|--------|------------|
| **Catalog Sync** | Initial sync adds startup time | Adjust `--catalog-sync-interval` |
| **Waiting for Ingestor** | Delays query/compact node startup | Adjust `--wait-for-running-ingestor` |
| **Node Count** | More nodes = more coordination | Stagger node restarts |

---

## Monitoring Startup Performance

### Enable Debug Logging for Startup

```bash
influxdb3 serve \
  --log-filter info,influxdb3_wal=debug \
  ...
```

### Key Log Messages to Monitor

Look for these patterns in startup logs:

```
# Snapshot loading progress
Loading snapshot X of Y...

# Checkpoint loading
Loading checkpoint for month YYYY-MM...

# WAL replay progress
Replaying WAL file X...

# Catalog sync
Catalog sync complete...
```

### Targeted Log Filtering for Startup Issues

```bash
# Debug WAL operations
--log-filter info,influxdb3_wal=debug

# Debug catalog operations
--log-filter info,influxdb3_catalog=debug

# Debug enterprise-specific operations
--log-filter info,influxdb3_enterprise=debug

# Multiple components
--log-filter info,influxdb3_wal=debug,influxdb3_catalog=debug
```

---

## Troubleshooting Checklist

### Quick Diagnosis

1. **Check if checkpointing is enabled:**
   ```bash
   # Look for --checkpoint-interval in your startup command
   # If not present, this is likely the cause of slow startups
   ```

2. **Calculate expected snapshot count:**
   ```
   Snapshots = gen1-lookback-duration / gen1-duration

   Example with defaults:
   24h / 10m = 144 snapshots (reasonable)

   Example with 1 month lookback:
   30d / 10m = 4,320 snapshots (slow!)
   ```

3. **Check available resources:**
   - Memory: Ensure adequate RAM for snapshot loading
   - CPU: More cores enable faster concurrent loading
   - Network: Verify bandwidth to object store

### Step-by-Step Resolution

| Step | Action | Expected Improvement |
|------|--------|---------------------|
| 1 | Enable checkpointing (`--checkpoint-interval 1h`) | 50-90% faster startup |
| 2 | Increase WAL replay concurrency | 10-30% faster |
| 3 | Optimize object store connections | 5-20% faster |
| 4 | Adjust lookback duration (if acceptable) | Proportional to reduction |
| 5 | Use local/faster storage | Significant for remote stores |

---


## Summary

### Top Recommendations for Fast Startup

1. **Enable checkpointing** - Most impactful single change
   ```bash
   --checkpoint-interval 1h
   ```

2. **Optimize WAL replay concurrency** - Based on available memory
   ```bash
   --wal-replay-concurrency-limit 16
   ```

3. **Increase object store connections** - For cloud storage
   ```bash
   --object-store-connection-limit 32
   ```

4. **Use appropriate lookback duration** - Balance between history and startup time
   ```bash
   --gen1-lookback-duration 24h
   ```

### Configuration Priority

| Priority | Setting | Impact |
|----------|---------|--------|
| Critical | `--checkpoint-interval` | Highest impact on startup time |
| High | `--wal-replay-concurrency-limit` | Significant improvement |
| Medium | `--object-store-connection-limit` | Moderate improvement |
| Low | `--gen1-lookback-duration` | Trade-off with data availability |

---

## References

- [InfluxDB 3 Enterprise Configuration Options](https://docs.influxdata.com/influxdb3/enterprise/reference/config-options/)
- [InfluxDB 3 Enterprise Serve Command](https://docs.influxdata.com/influxdb3/enterprise/reference/cli/influxdb3/serve/)
- [Data Durability and WAL](https://docs.influxdata.com/influxdb3/enterprise/reference/internals/durability/)
- [Performance Tuning](https://docs.influxdata.com/influxdb3/enterprise/admin/performance-tuning/)
