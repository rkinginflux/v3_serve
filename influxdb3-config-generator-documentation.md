# InfluxDB 3 Enterprise Configuration Generator

## Overview

The `influxdb3-config-generator.sh` script generates recommended startup options for the `influxdb3 serve` command based on your system's hardware resources and desired node mode. It calculates optimal thread allocation, memory settings, and provides ready-to-use commands.

---


## Features

### Automatic Resource Detection
- **CPU Detection**: Automatically detects available CPU cores using `/proc/cpuinfo` (Linux) or `sysctl` (macOS)
- **RAM Detection**: Automatically detects total system memory
- **Override Support**: Manual specification via `--cpus` and `--ram` flags

### Intelligent Thread Allocation
The script calculates optimal thread distribution based on the selected mode:

| Thread Type | Purpose |
|-------------|---------|
| **IO Threads** | HTTP request handling, line protocol parsing, file operations |
| **DataFusion Threads** | Query execution, data snapshots, compaction operations |

### Mode-Specific Optimization
Each mode receives tailored settings:
- **Ingest**: More IO threads for concurrent writers
- **Query**: Maximum DataFusion threads, larger parquet cache
- **Compact**: DataFusion-heavy for sort/dedupe operations
- **Process**: Balanced allocation for plugin execution

### Output Formats
- Command-line arguments (ready to copy/paste)
- Environment variables (with `--env-vars` flag)
- Quick reference comparison table

### Additional Settings
The script also calculates:
- Memory pool percentage
- Parquet cache size
- WAL replay concurrency
- Object store connection limits
- Checkpoint intervals
- Mode-specific options (compaction settings, plugin directory, etc.)

---

## Supported Modes

| Mode | Flag Value | Description | IO Thread Strategy | DataFusion Strategy |
|------|------------|-------------|-------------------|---------------------|
| **All** | `all` | All capabilities enabled | Balanced (~20% of cores) | Remaining cores |
| **Ingest** | `ingest` | High-throughput data ingestion | High (~35% of cores, max 20) | Remaining cores |
| **Query** | `query` | Analytical query execution | Minimal (2-4 threads) | Maximum available |
| **Compact** | `compact` | Background compaction | Minimal (2 threads) | Maximum available |
| **Process** | `process` | Data processing with plugins | Moderate (~25% of cores) | Remaining cores |
| **Ingest+Query** | `ingest,query` | Combined workload | Moderate (~25% of cores) | Remaining cores |
| **Query+Compact** | `query,compact` | Combined workload | Minimal (2-4 threads) | Maximum available |

### Mode Selection Guidelines

| Use Case | Recommended Mode |
|----------|------------------|
| Single-node deployment | `all` |
| Dedicated write server | `ingest` |
| Dedicated read server | `query` |
| Background maintenance node | `compact` |
| Plugin/transformation server | `process` |
| Read/write hybrid | `ingest,query` |
| Read with self-compaction | `query,compact` |

---

## Command-Line Options

| Option | Description | Default |
|--------|-------------|---------|
| `--cpus N` | Number of CPU cores | Auto-detect |
| `--ram N` | RAM in GB | Auto-detect |
| `--mode MODE` | Node mode | Interactive selection |
| `--object-store TYPE` | Object store type: `file`, `s3`, `gcs`, `azure` | `file` |
| `--env-vars` | Also output environment variable configuration | Disabled |
| `--help` | Show help message | - |

---

## Output Sections

### 1. Resource Allocation Summary

Displays the calculated resource distribution:

```
System Resources:
  CPU Cores:        32
  RAM:              64GB

Thread Allocation:
  IO Threads:       11 (HTTP requests, line protocol parsing)
  DataFusion:       21 (queries, snapshots, compaction)

Memory Allocation:
  Exec Memory Pool: 60%
  Parquet Cache:    6GB

Mode: ingest
Description: Optimized for high-throughput data ingestion
```

### 2. Recommended Command

A ready-to-use `influxdb3 serve` command with all calculated options:

```bash
influxdb3 --num-io-threads=11 serve \
  --mode=ingest \
  --node-id=<YOUR_NODE_ID> \
  --cluster-id=<YOUR_CLUSTER_ID> \
  --object-store=file \
  --data-dir=/var/lib/influxdb3 \
  --datafusion-num-threads=21 \
  --exec-mem-pool-bytes=60% \
  --checkpoint-interval=1h \
  --wal-replay-concurrency-limit=20 \
  --force-snapshot-mem-threshold=70% \
  --object-store-connection-limit=32 \
  --license-email=<YOUR_EMAIL>
```

### 3. Environment Variables (Optional)

When using `--env-vars`, displays equivalent environment variable configuration:

```bash
# Thread configuration
export INFLUXDB3_NUM_IO_THREADS=11
export INFLUXDB3_DATAFUSION_NUM_THREADS=21

# Mode and identity
export INFLUXDB3_ENTERPRISE_MODE=ingest
export INFLUXDB3_NODE_IDENTIFIER_PREFIX=<YOUR_NODE_ID>
export INFLUXDB3_ENTERPRISE_CLUSTER_ID=<YOUR_CLUSTER_ID>

# Memory configuration
export INFLUXDB3_EXEC_MEM_POOL_BYTES=60%
export INFLUXDB3_FORCE_SNAPSHOT_MEM_THRESHOLD=70%

# Startup optimization
export INFLUXDB3_CHECKPOINT_INTERVAL=1h
export INFLUXDB3_WAL_REPLAY_CONCURRENCY_LIMIT=20

# Object store
export INFLUXDB3_OBJECT_STORE=file
export INFLUXDB3_DB_DIR=/var/lib/influxdb3
export INFLUXDB3_OBJECT_STORE_CONNECTION_LIMIT=32

# Licensing
export INFLUXDB3_ENTERPRISE_LICENSE_EMAIL=<YOUR_EMAIL>
```

### 4. Additional Recommendations

Mode-specific tips and best practices:

```
• Scale IO threads with concurrent writers (~1 thread per writer)
• Monitor WAL size and adjust --wal-flush-interval if needed
• Consider --wal-max-write-buffer-size for high-throughput scenarios

• Always test configuration changes in a non-production environment
• Monitor metrics endpoint at /metrics for performance insights
• Use --log-filter for debugging specific components
```

### 5. Quick Reference Table

Comparison of all modes for your specific hardware:

```
Mode            IO Threads   DF Threads   Mem Pool        Cache
--------------- ---------- ------------ ---------- ------------
all                      6           26        70%          6GB
ingest                  11           21        60%            -
query                    4           28        90%          9GB
compact                  2           30        80%            -
process                  8           24        70%          6GB
ingest,query             8           24        75%          6GB
query,compact            4           28        85%          7GB
```

---

## Usage Examples

### Basic Usage (Auto-Detect Resources)

```bash
# Interactive mode selection with auto-detected resources
./influxdb3-config-generator.sh
```

### Specify Resources and Mode

```bash
# 32-core ingest node with 64GB RAM
./influxdb3-config-generator.sh --cpus 32 --ram 64 --mode ingest

# 48-core query node with 128GB RAM
./influxdb3-config-generator.sh --cpus 48 --ram 128 --mode query

# 16-core compactor with 32GB RAM
./influxdb3-config-generator.sh --cpus 16 --ram 32 --mode compact
```

### Combined Modes

```bash
# Combined ingest and query node
./influxdb3-config-generator.sh --cpus 64 --ram 256 --mode ingest-query

# Combined query and compaction node
./influxdb3-config-generator.sh --cpus 32 --ram 64 --mode query-compact
```

### Different Object Stores

```bash
# AWS S3 configuration
./influxdb3-config-generator.sh --cpus 32 --ram 64 --mode query --object-store s3

# Google Cloud Storage configuration
./influxdb3-config-generator.sh --cpus 32 --ram 64 --mode ingest --object-store gcs

# Azure Blob Storage configuration
./influxdb3-config-generator.sh --cpus 32 --ram 64 --mode all --object-store azure

# Local file storage (default)
./influxdb3-config-generator.sh --cpus 16 --ram 32 --mode all --object-store file
```

### Include Environment Variables

```bash
# Generate both command and environment variables
./influxdb3-config-generator.sh --cpus 32 --ram 64 --mode ingest --env-vars

# S3 with environment variables
./influxdb3-config-generator.sh --cpus 48 --ram 128 --mode query --object-store s3 --env-vars
```

### Show Help

```bash
./influxdb3-config-generator.sh --help
```

---

## Thread Allocation Algorithm

### Ingest Mode
```
IO Threads = min(max(CPU * 0.35, 4), 20)
DataFusion Threads = CPU - IO Threads
Memory Pool = 60%
Parquet Cache = 0 (not needed for ingest)
```

### Query Mode
```
IO Threads = 4 (or 2 if CPU < 8)
DataFusion Threads = CPU - IO Threads
Memory Pool = 90%
Parquet Cache = min(RAM * 0.15, 16GB)
```

### Compact Mode
```
IO Threads = 2
DataFusion Threads = CPU - IO Threads
Memory Pool = 80%
Parquet Cache = 0 (not needed for compaction)
```

### Process Mode
```
IO Threads = min(max(CPU * 0.25, 2), 8)
DataFusion Threads = CPU - IO Threads
Memory Pool = 70%
Parquet Cache = min(RAM * 0.10, 8GB)
```

### All Mode (Balanced)
```
IO Threads = min(max(CPU * 0.20, 2), 12)
DataFusion Threads = CPU - IO Threads
Memory Pool = 70%
Parquet Cache = min(RAM * 0.10, 8GB)
```

---

## Sample Output

### Example: 32 CPU / 64GB RAM / Ingest Mode

```
════════════════════════════════════════════════════════════════
  InfluxDB 3 Enterprise - Configuration Generator
════════════════════════════════════════════════════════════════

ℹ  Using specified CPUs: 32
ℹ  Using specified RAM: 64GB
✓  Selected mode: ingest

── Resource Allocation Summary ──

System Resources:
  CPU Cores:        32
  RAM:              64GB

Thread Allocation:
  IO Threads:       11 (HTTP requests, line protocol parsing)
  DataFusion:       21 (queries, snapshots, compaction)

Memory Allocation:
  Exec Memory Pool: 60%

Mode: ingest
Description: Optimized for high-throughput data ingestion

── Recommended Command ──

influxdb3 --num-io-threads=11 serve \
  --mode=ingest \
  --node-id=<YOUR_NODE_ID> \
  --cluster-id=<YOUR_CLUSTER_ID> \
  --object-store=file \
  --data-dir=/var/lib/influxdb3 \
  --datafusion-num-threads=21 \
  --exec-mem-pool-bytes=60% \
  --checkpoint-interval=1h \
  --wal-replay-concurrency-limit=20 \
  --force-snapshot-mem-threshold=70% \
  --object-store-connection-limit=32 \
  --license-email=<YOUR_EMAIL>

── Additional Recommendations ──

• Scale IO threads with concurrent writers (~1 thread per writer)
• Monitor WAL size and adjust --wal-flush-interval if needed
• Consider --wal-max-write-buffer-size for high-throughput scenarios

• Always test configuration changes in a non-production environment
• Monitor metrics endpoint at /metrics for performance insights
• Use --log-filter for debugging specific components

── Quick Reference - All Modes for 32 CPUs / 64GB RAM ──

Mode            IO Threads   DF Threads   Mem Pool        Cache
--------------- ---------- ------------ ---------- ------------
all                      6           26        70%          6GB
ingest                  11           21        60%            -
query                    4           28        90%          9GB
compact                  2           30        80%            -
process                  8           24        70%          6GB
ingest,query             8           24        75%          6GB
query,compact            4           28        85%          7GB

✓  Configuration generated successfully!
```

---

## Integration Examples

### Kubernetes Deployment

Use the environment variable output to configure a Kubernetes deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: influxdb3-ingest
spec:
  template:
    spec:
      containers:
      - name: influxdb3
        image: influxdb:3-enterprise
        env:
        - name: INFLUXDB3_NUM_IO_THREADS
          value: "11"
        - name: INFLUXDB3_DATAFUSION_NUM_THREADS
          value: "21"
        - name: INFLUXDB3_ENTERPRISE_MODE
          value: "ingest"
        - name: INFLUXDB3_EXEC_MEM_POOL_BYTES
          value: "60%"
        - name: INFLUXDB3_CHECKPOINT_INTERVAL
          value: "1h"
        # ... additional env vars from script output
```

### Docker Compose

```yaml
services:
  influxdb3-query:
    image: influxdb:3-enterprise
    environment:
      INFLUXDB3_NUM_IO_THREADS: "4"
      INFLUXDB3_DATAFUSION_NUM_THREADS: "28"
      INFLUXDB3_ENTERPRISE_MODE: "query"
      INFLUXDB3_EXEC_MEM_POOL_BYTES: "90%"
      INFLUXDB3_PARQUET_MEM_CACHE_SIZE: "9GB"
      INFLUXDB3_CHECKPOINT_INTERVAL: "1h"
      # ... additional env vars from script output
```

### Systemd Service

```ini
[Service]
Environment="INFLUXDB3_NUM_IO_THREADS=11"
Environment="INFLUXDB3_DATAFUSION_NUM_THREADS=21"
Environment="INFLUXDB3_ENTERPRISE_MODE=ingest"
Environment="INFLUXDB3_EXEC_MEM_POOL_BYTES=60%"
Environment="INFLUXDB3_CHECKPOINT_INTERVAL=1h"
ExecStart=/usr/bin/influxdb3 serve
```

---

## Troubleshooting

### Auto-Detection Not Working

```bash
# Manually specify resources
./influxdb3-config-generator.sh --cpus 16 --ram 32 --mode all
```

### Invalid Mode Error

Use one of the supported mode values:
- `all`, `ingest`, `query`, `compact`, `process`
- `ingest-query` or `ingest,query`
- `query-compact` or `query,compact`

---

## References

- [InfluxDB 3 Enterprise Serve Options](./influxdb3-enterprise-serve-options.md)
- [Slow Startup Troubleshooting](./influxdb3-enterprise-slow-startup-troubleshooting.md)
- [Official Documentation](https://docs.influxdata.com/influxdb3/enterprise/reference/config-options/)
- [Clustering Configuration](https://docs.influxdata.com/influxdb3/enterprise/admin/clustering/)
