# InfluxDB 3 Enterprise - Serve Command Startup Options

## Overview

InfluxDB 3 Enterprise uses a **single `influxdb3 serve` command** with a `--mode` flag to configure specialized nodes. There are no separate commands for ingest/query/compactor - instead, you configure the node type using the `--mode` parameter.

## Node Modes

| Mode | Description |
|------|-------------|
| `all` | All capabilities enabled (default) |
| `ingest` | Data ingestion and line protocol parsing |
| `query` | Query execution and data retrieval |
| `compact` | Background compaction and optimization |
| `process` | Data processing and transformations |

Modes can be combined using comma-separated values: `--mode=ingest,query` or `--mode=query,compact`

---

## Required Parameters

| Option | Environment Variable | Description |
|--------|---------------------|-------------|
| `--node-id` | `INFLUXDB3_NODE_IDENTIFIER_PREFIX` | Unique identifier for the server instance. Must be unique for hosts sharing the same object store. |
| `--cluster-id` | `INFLUXDB3_ENTERPRISE_CLUSTER_ID` | Unique cluster identifier. Must be different from any node-id in the cluster. |
| `--object-store` | `INFLUXDB3_OBJECT_STORE` | Storage type: `memory`, `memory-throttled`, `file`, `s3`, `google`, `azure` |

---

## Global Options

Global options must be specified **before** the `serve` command.

```bash
influxdb3 --num-io-threads=8 serve --node-id=node0 --cluster-id=cluster0
```

| Option | Environment Variable | Default | Description |
|--------|---------------------|---------|-------------|
| `--num-io-threads` | `INFLUXDB3_NUM_IO_THREADS` | `2` | Number of threads for IO runtime (HTTP requests, line protocol parsing, file operations) |

---

## Server Configuration Options

### General

| Option | Environment Variable | Default | Description |
|--------|---------------------|---------|-------------|
| `--mode` | `INFLUXDB3_ENTERPRISE_MODE` | `all` | Node mode(s): `all`, `ingest`, `query`, `compact`, `process` |
| `--data-dir` | `INFLUXDB3_DB_DIR` | | Local data directory (required for `file` object store) |
| `--node-id-from-env` | `INFLUXDB3_NODE_IDENTIFIER_FROM_ENV` | | Read node-id from specified environment variable |
| `--bucket` | `INFLUXDB3_BUCKET` | | Object storage bucket name |
| `--query-file-limit` | `INFLUXDB3_QUERY_FILE_LIMIT` | | Query file limit |
| `--use-pacha-tree` | `INFLUXDB3_ENTERPRISE_USE_PACHA_TREE` | `false` | Enable PachaTree storage engine (experimental) |

### Licensing

| Option | Environment Variable | Description |
|--------|---------------------|-------------|
| `--license-email` | `INFLUXDB3_ENTERPRISE_LICENSE_EMAIL` | Email address for license registration |
| `--license-file` | `INFLUXDB3_ENTERPRISE_LICENSE_FILE` | Path to license file |
| `--license-type` | `INFLUXDB3_ENTERPRISE_LICENSE_TYPE` | License type: `home`, `trial`, `commercial` |

### Security / TLS

| Option | Environment Variable | Default | Description |
|--------|---------------------|---------|-------------|
| `--tls-key` | `INFLUXDB3_TLS_KEY` | | Path to TLS key file |
| `--tls-cert` | `INFLUXDB3_TLS_CERT` | | Path to TLS certificate file |
| `--tls-minimum-version` | `INFLUXDB3_TLS_MINIMUM_VERSION` | `tls-1.2` | Minimum TLS version: `tls-1.2`, `tls-1.3` |
| `--without-auth` | `INFLUXDB3_START_WITHOUT_AUTH` | | Disable authentication for all requests |
| `--disable-authz` | `INFLUXDB3_DISABLE_AUTHZ` | | Disable authz for specific resources: `health`, `ping`, `metrics` |
| `--admin-token-file` | `INFLUXDB3_ADMIN_TOKEN_FILE` | | Path to offline admin token file |
| `--permission-tokens-file` | `INFLUXDB3_PERMISSION_TOKENS_FILE` | | Path to offline permission tokens file |
| `--admin-token-recovery-http-bind` | `INFLUXDB3_ADMIN_TOKEN_RECOVERY_HTTP_BIND` | `127.0.0.1:8182` | Enable admin token recovery HTTP server |
| `--admin-token-recovery-tcp-listener-file-path` | `INFLUXDB3_ADMIN_TOKEN_RECOVERY_TCP_LISTENER_FILE_PATH` | | TCP listener file path for admin token recovery |

### HTTP Server

| Option | Environment Variable | Default | Description |
|--------|---------------------|---------|-------------|
| `--http-bind` | `INFLUXDB3_HTTP_BIND_ADDR` | `0.0.0.0:8181` | HTTP server bind address |
| `--max-http-request-size` | `INFLUXDB3_MAX_HTTP_REQUEST_SIZE` | | Maximum HTTP request size in bytes |

### DataFusion / Query Performance

| Option | Environment Variable | Description |
|--------|---------------------|-------------|
| `--datafusion-num-threads` | `INFLUXDB3_DATAFUSION_NUM_THREADS` | Number of DataFusion query execution threads |
| `--datafusion-config` | `INFLUXDB3_DATAFUSION_CONFIG` | Custom DataFusion configuration (key:value pairs) |
| `--datafusion-max-parquet-fanout` | `INFLUXDB3_DATAFUSION_MAX_PARQUET_FANOUT` | Maximum parquet fanout |
| `--datafusion-use-cached-parquet-loader` | `INFLUXDB3_DATAFUSION_USE_CACHED_PARQUET_LOADER` | Enable cached parquet loader |
| `--num-cores` | `INFLUXDB3_NUM_CORES` | Number of CPU cores to use |
| `--query-log-size` | `INFLUXDB3_QUERY_LOG_SIZE` | Query log size (default: 1000) |

#### DataFusion Runtime Options (Development)

| Option | Description |
|--------|-------------|
| `--datafusion-runtime-disable-lifo-slot` | Development-only Tokio runtime configuration |
| `--datafusion-runtime-event-interval` | Development-only Tokio runtime configuration |
| `--datafusion-runtime-global-queue-interval` | Development-only Tokio runtime configuration |
| `--datafusion-runtime-max-blocking-threads` | Development-only Tokio runtime configuration |
| `--datafusion-runtime-max-io-events-per-tick` | Development-only Tokio runtime configuration |
| `--datafusion-runtime-thread-keep-alive` | Development-only Tokio runtime configuration |
| `--datafusion-runtime-thread-priority` | Development-only Tokio runtime configuration |
| `--datafusion-runtime-type` | Development-only Tokio runtime configuration |

### Memory Configuration

| Option | Environment Variable | Description |
|--------|---------------------|-------------|
| `--exec-mem-pool-bytes` | `INFLUXDB3_EXEC_MEM_POOL_BYTES` | Execution memory pool size (bytes or percentage like `60%`) |
| `--force-snapshot-mem-threshold` | `INFLUXDB3_FORCE_SNAPSHOT_MEM_THRESHOLD` | Memory threshold for forced snapshots |

### Write-Ahead Log (WAL)

| Option | Environment Variable | Description |
|--------|---------------------|-------------|
| `--wal-flush-interval` | `INFLUXDB3_WAL_FLUSH_INTERVAL` | WAL flush interval |
| `--wal-max-write-buffer-size` | `INFLUXDB3_WAL_MAX_WRITE_BUFFER_SIZE` | Maximum WAL write buffer size |
| `--wal-snapshot-size` | `INFLUXDB3_WAL_SNAPSHOT_SIZE` | WAL snapshot size |
| `--wal-replay-concurrency-limit` | `INFLUXDB3_WAL_REPLAY_CONCURRENCY_LIMIT` | WAL replay concurrency limit |
| `--wal-replay-fail-on-error` | `INFLUXDB3_WAL_REPLAY_FAIL_ON_ERROR` | Fail on WAL replay error |
| `--snapshotted-wal-files-to-keep` | `INFLUXDB3_SNAPSHOTTED_WAL_FILES_TO_KEEP` | Number of snapshotted WAL files to retain |

### Compaction

These options are particularly relevant for nodes running in `compact` mode.

| Option | Environment Variable | Description |
|--------|---------------------|-------------|
| `--compaction-check-interval` | `INFLUXDB3_COMPACTION_CHECK_INTERVAL` | Interval between compaction checks |
| `--compaction-cleanup-wait` | `INFLUXDB3_COMPACTION_CLEANUP_WAIT` | Wait time before cleanup |
| `--compaction-gen2-duration` | `INFLUXDB3_COMPACTION_GEN2_DURATION` | Gen2 compaction duration |
| `--compaction-max-num-files-per-plan` | `INFLUXDB3_COMPACTION_MAX_NUM_FILES_PER_PLAN` | Maximum files per compaction plan |
| `--compaction-multipliers` | `INFLUXDB3_COMPACTION_MULTIPLIERS` | Compaction multipliers (e.g., `4,8,16`) |

### Caching

| Option | Environment Variable | Description |
|--------|---------------------|-------------|
| `--parquet-mem-cache-size` | `INFLUXDB3_PARQUET_MEM_CACHE_SIZE` | Parquet memory cache size (e.g., `8GB`) |
| `--parquet-mem-cache-prune-interval` | `INFLUXDB3_PARQUET_MEM_CACHE_PRUNE_INTERVAL` | Cache prune interval |
| `--parquet-mem-cache-prune-percentage` | `INFLUXDB3_PARQUET_MEM_CACHE_PRUNE_PERCENTAGE` | Percentage to prune from cache |
| `--parquet-mem-cache-query-path-duration` | `INFLUXDB3_PARQUET_MEM_CACHE_QUERY_PATH_DURATION` | Query path cache duration |
| `--disable-parquet-mem-cache` | `INFLUXDB3_DISABLE_PARQUET_MEM_CACHE` | Disable parquet memory cache |
| `--preemptive-cache-age` | `INFLUXDB3_PREEMPTIVE_CACHE_AGE` | Preemptive cache age |
| `--last-cache-eviction-interval` | `INFLUXDB3_LAST_CACHE_EVICTION_INTERVAL` | Last cache eviction interval |
| `--distinct-cache-eviction-interval` | `INFLUXDB3_DISTINCT_CACHE_EVICTION_INTERVAL` | Distinct cache eviction interval |
| `--last-value-cache-disable-from-history` | `INFLUXDB3_LAST_VALUE_CACHE_DISABLE_FROM_HISTORY` | Disable last value cache from history |
| `--distinct-value-cache-disable-from-history` | `INFLUXDB3_DISTINCT_VALUE_CACHE_DISABLE_FROM_HISTORY` | Disable distinct value cache from history |
| `--table-index-cache-concurrency-limit` | `INFLUXDB3_TABLE_INDEX_CACHE_CONCURRENCY_LIMIT` | Table index cache concurrency limit |
| `--table-index-cache-max-entries` | `INFLUXDB3_TABLE_INDEX_CACHE_MAX_ENTRIES` | Maximum table index cache entries |

### Processing Engine

These options are relevant for nodes running in `process` mode.

| Option | Environment Variable | Description |
|--------|---------------------|-------------|
| `--plugin-dir` | `INFLUXDB3_PLUGIN_DIR` | Directory containing plugins |
| `--virtual-env-location` | `INFLUXDB3_VIRTUAL_ENV_LOCATION` | Python virtual environment location |
| `--package-manager` | `INFLUXDB3_PACKAGE_MANAGER` | Package manager to use |

> **Note:** Setting `--plugin-dir` automatically adds `process` mode to the node.

### Cluster Management

| Option | Environment Variable | Description |
|--------|---------------------|-------------|
| `--catalog-sync-interval` | `INFLUXDB3_CATALOG_SYNC_INTERVAL` | Catalog synchronization interval |
| `--replication-interval` | `INFLUXDB3_REPLICATION_INTERVAL` | Data replication interval |
| `--wait-for-running-ingestor` | `INFLUXDB3_WAIT_FOR_RUNNING_INGESTOR` | Wait for a running ingestor before starting |

### Resource Limits

| Option | Environment Variable | Description |
|--------|---------------------|-------------|
| `--num-database-limit` | `INFLUXDB3_NUM_DATABASE_LIMIT` | Maximum number of databases |
| `--num-table-limit` | `INFLUXDB3_NUM_TABLE_LIMIT` | Maximum number of tables |
| `--num-total-columns-per-table-limit` | `INFLUXDB3_NUM_TOTAL_COLUMNS_PER_TABLE_LIMIT` | Maximum columns per table |

### Data Lifecycle Management

| Option | Environment Variable | Description |
|--------|---------------------|-------------|
| `--gen1-duration` | `INFLUXDB3_GEN1_DURATION` | Gen1 duration |
| `--gen1-lookback-duration` | `INFLUXDB3_GEN1_LOOKBACK_DURATION` | Gen1 lookback duration |
| `--retention-check-interval` | `INFLUXDB3_RETENTION_CHECK_INTERVAL` | Retention policy check interval |
| `--delete-grace-period` | `INFLUXDB3_DELETE_GRACE_PERIOD` | Delete grace period |
| `--hard-delete-default-duration` | `INFLUXDB3_HARD_DELETE_DEFAULT_DURATION` | Default duration for hard deletes |

### Logging

| Option | Environment Variable | Default | Description |
|--------|---------------------|---------|-------------|
| `--log-filter` | `INFLUXDB3_LOG_FILTER` | `info` | Log filter level: `error`, `warn`, `info`, `debug`, `trace` |
| `--log-destination` | `INFLUXDB3_LOG_DESTINATION` | `stdout` | Log destination: `stdout`, `stderr` |
| `--log-format` | `INFLUXDB3_LOG_FORMAT` | `full` | Log format |
| `--verbose` | | | Enable verbose logging |

#### Targeted Log Filtering

You can set different log levels for specific components:

```bash
# Debug write buffer operations
--log-filter info,influxdb3_write_buffer=debug

# Trace WAL operations
--log-filter info,influxdb3_wal=trace

# Multiple targeted filters
--log-filter info,influxdb3_write_buffer=debug,influxdb3_wal=debug
```

### Tracing


| Option | Environment Variable | Default | Description |
|--------|---------------------|---------|-------------|
| `--traces-exporter` | `INFLUXDB3_TRACES_EXPORTER` | `none` | Tracing exporter type |
| `--traces-exporter-jaeger-agent-host` | `INFLUXDB3_TRACES_EXPORTER_JAEGER_AGENT_HOST` | `0.0.0.0` | Jaeger agent hostname |
| `--traces-exporter-jaeger-agent-port` | `INFLUXDB3_TRACES_EXPORTER_JAEGER_AGENT_PORT` | `6831` | Jaeger agent port |
| `--traces-exporter-jaeger-service-name` | `INFLUXDB3_TRACES_EXPORTER_JAEGER_SERVICE_NAME` | `iox-conductor` | Jaeger service name |
| `--traces-exporter-jaeger-trace-context-header-name` | `INFLUXDB3_TRACES_EXPORTER_JAEGER_TRACE_CONTEXT_HEADER_NAME` | `uber-trace-id` | Trace context header name |
| `--traces-jaeger-debug-name` | `INFLUXDB3_TRACES_JAEGER_DEBUG_NAME` | `jaeger-debug-id` | Jaeger debug header name |
| `--traces-jaeger-tags` | `INFLUXDB3_TRACES_JAEGER_TAGS` | | Jaeger tags |
| `--traces-jaeger-max-msgs-per-second` | `INFLUXDB3_TRACES_JAEGER_MAX_MSGS_PER_SECOND` | | Max trace messages per second |

### Telemetry

| Option | Environment Variable | Description |
|--------|---------------------|-------------|
| `--telemetry-disable-upload` | `INFLUXDB3_DISABLE_TELEMETRY_UPLOAD` | Disable telemetry upload |
| `--telemetry-endpoint` | `INFLUXDB3_TELEMETRY_ENDPOINT` | Custom telemetry endpoint |

### TCP Listeners

| Option | Environment Variable | Description |
|--------|---------------------|-------------|
| `--tcp-listener-file-path` | `INFLUXDB3_TCP_LISTENER_FILE_PATH` | TCP listener file path |

### AWS S3 Configuration

Required when using `--object-store=s3`.

| Option | Environment Variable | Default | Description |
|--------|---------------------|---------|-------------|
| `--aws-access-key-id` | `AWS_ACCESS_KEY_ID` | | AWS access key ID |
| `--aws-secret-access-key` | `AWS_SECRET_ACCESS_KEY` | | AWS secret access key |
| `--aws-default-region` | `AWS_DEFAULT_REGION` | `us-east-1` | AWS region |
| `--aws-endpoint` | `AWS_ENDPOINT` | | Custom S3 endpoint (for S3-compatible storage) |
| `--aws-session-token` | `AWS_SESSION_TOKEN` | | AWS session token |
| `--aws-allow-http` | `AWS_ALLOW_HTTP` | | Allow unencrypted HTTP to AWS |
| `--aws-skip-signature` | `AWS_SKIP_SIGNATURE` | | Skip request signing |
| `--aws-credentials-file` | `AWS_CREDENTIALS_FILE` | | Path to AWS credentials file |

### Google Cloud Storage Configuration

Required when using `--object-store=google`.

| Option | Environment Variable | Description |
|--------|---------------------|-------------|
| `--google-service-account` | `GOOGLE_SERVICE_ACCOUNT` | Path to Google service account JSON file |

### Microsoft Azure Configuration

Required when using `--object-store=azure`.

| Option | Environment Variable | Default | Description |
|--------|---------------------|---------|-------------|
| `--azure-storage-account` | `AZURE_STORAGE_ACCOUNT` | | Azure storage account name |
| `--azure-storage-access-key` | `AZURE_STORAGE_ACCESS_KEY` | | Azure storage access key |
| `--azure-endpoint` | `AZURE_ENDPOINT` | | Azure Blob Storage endpoint |
| `--azure-allow-http` | `AZURE_ALLOW_HTTP` | `false` | Allow unencrypted HTTP to Azure |

### Object Storage (General)

| Option | Environment Variable | Default | Description |
|--------|---------------------|---------|-------------|
| `--object-store-connection-limit` | `INFLUXDB3_OBJECT_STORE_CONNECTION_LIMIT` | `16` | Connection limit for object store |
| `--object-store-http2-only` | `INFLUXDB3_OBJECT_STORE_HTTP2_ONLY` | | Force HTTP/2 connections |
| `--object-store-http2-max-frame-size` | `INFLUXDB3_OBJECT_STORE_HTTP2_MAX_FRAME_SIZE` | | HTTP/2 max frame size |
| `--object-store-max-retries` | `INFLUXDB3_OBJECT_STORE_MAX_RETRIES` | | Maximum retry attempts |
| `--object-store-retry-timeout` | `INFLUXDB3_OBJECT_STORE_RETRY_TIMEOUT` | | Retry timeout |
| `--object-store-cache-endpoint` | `INFLUXDB3_OBJECT_STORE_CACHE_ENDPOINT` | | S3-compatible cache endpoint |

---

## Example Configurations by Node Type

### Ingest Node

Optimized for high-throughput data writes. Allocate more IO threads for line protocol parsing.

```bash
influxdb3 --num-io-threads=12 serve \
  --mode=ingest \
  --node-id=ingester-01 \
  --cluster-id=prod-cluster \
  --object-store=s3 \
  --bucket=my-bucket \
  --aws-default-region=us-west-2 \
  --datafusion-num-threads=20 \
  --exec-mem-pool-bytes=60% \
  --license-email=user@example.com
```

**Thread allocation rationale:**
- **12 IO threads**: Handle multiple concurrent writers (Telegraf agents, applications)
- **20 DataFusion threads**: Required for data snapshot operations that convert buffered writes to Parquet files
- **60% memory pool**: Balance between write buffers and data snapshot operations

### Query Node

Optimized for complex analytical queries. Maximize DataFusion threads.

```bash
influxdb3 --num-io-threads=4 serve \
  --mode=query \
  --node-id=query-01 \
  --cluster-id=prod-cluster \
  --object-store=s3 \
  --bucket=my-bucket \
  --aws-default-region=us-west-2 \
  --datafusion-num-threads=60 \
  --exec-mem-pool-bytes=90% \
  --parquet-mem-cache-size=8GB \
  --license-email=user@example.com
```

**Thread allocation rationale:**
- **4 IO threads**: Minimal, just for HTTP request handling
- **60 DataFusion threads**: Maximum parallelism for query execution
- **90% memory pool**: Maximize memory for complex aggregations
- **8 GB Parquet cache**: Keep frequently accessed data in memory

### Compactor Node

Dedicated to background data compaction and optimization.

```bash
influxdb3 --num-io-threads=2 serve \
  --mode=compact \
  --node-id=compactor-01 \
  --cluster-id=prod-cluster \
  --object-store=s3 \
  --bucket=my-bucket \
  --aws-default-region=us-west-2 \
  --datafusion-num-threads=30 \
  --compaction-gen2-duration=24h \
  --compaction-check-interval=5m \
  --compaction-max-num-files-per-plan=100 \
  --license-email=user@example.com
```

**Thread allocation rationale:**
- **2 IO threads**: Minimal, compaction is DataFusion-intensive
- **30 DataFusion threads**: Maximum threads for sort/merge operations
- **24h gen2 duration**: Time-based compaction strategy

### Process Node

Focused on data processing and Python plugin transformations.

```bash
influxdb3 --num-io-threads=4 serve \
  --mode=process \
  --node-id=processor-01 \
  --cluster-id=prod-cluster \
  --object-store=s3 \
  --bucket=my-bucket \
  --aws-default-region=us-west-2 \
  --datafusion-num-threads=12 \
  --plugin-dir=/path/to/plugins \
  --license-email=user@example.com
```

### Combined Ingest + Query Node

Balanced for mixed workloads.

```bash
influxdb3 --num-io-threads=8 serve \
  --mode=ingest,query \
  --node-id=hybrid-01 \
  --cluster-id=prod-cluster \
  --object-store=s3 \
  --bucket=my-bucket \
  --aws-default-region=us-west-2 \
  --datafusion-num-threads=24 \
  --exec-mem-pool-bytes=75% \
  --license-email=user@example.com
```

### All-in-One Node (Default)

All capabilities enabled.

```bash
influxdb3 --num-io-threads=6 serve \
  --mode=all \
  --node-id=node-01 \
  --cluster-id=prod-cluster \
  --object-store=file \
  --data-dir=/var/lib/influxdb3 \
  --datafusion-num-threads=26 \
  --license-email=user@example.com
```

---

## Environment Variable Configuration

You can configure nodes entirely via environment variables:

```bash
# Licensing
export INFLUXDB3_ENTERPRISE_LICENSE_EMAIL=user@example.com
export INFLUXDB3_ENTERPRISE_CLUSTER_ID=prod-cluster

# Node configuration
export INFLUXDB3_NODE_IDENTIFIER_PREFIX=ingester-01
export INFLUXDB3_ENTERPRISE_MODE=ingest

# Object storage
export INFLUXDB3_OBJECT_STORE=s3
export INFLUXDB3_BUCKET=my-bucket
export AWS_ACCESS_KEY_ID=your-access-key
export AWS_SECRET_ACCESS_KEY=your-secret-key
export AWS_DEFAULT_REGION=us-west-2

# Performance tuning
export INFLUXDB3_NUM_IO_THREADS=12
export INFLUXDB3_DATAFUSION_NUM_THREADS=20
export INFLUXDB3_EXEC_MEM_POOL_BYTES=60%

# Logging
export INFLUXDB3_LOG_FILTER=info

# Start server
influxdb3 serve
```

---

## Cluster Architecture Examples

### Small Cluster (3 Nodes)

```yaml
Node 1 (All-in-one primary):
  mode: all
  cores: 32
  io_threads: 8
  datafusion_threads: 24

Node 2 (All-in-one secondary):
  mode: all
  cores: 32
  io_threads: 8
  datafusion_threads: 24

Node 3 (All-in-one tertiary):
  mode: all
  cores: 32
  io_threads: 8
  datafusion_threads: 24
```

### Medium Cluster (6 Nodes)

```yaml
Nodes 1-2 (Ingesters):
  mode: ingest
  cores: 48
  io_threads: 16
  datafusion_threads: 32

Nodes 3-4 (Query nodes):
  mode: query
  cores: 48
  io_threads: 4
  datafusion_threads: 44

Nodes 5-6 (Compactor + Process):
  mode: compact,process
  cores: 32
  io_threads: 4
  datafusion_threads: 28
```

### Large Cluster (12+ Nodes)

```yaml
Nodes 1-4 (High-throughput ingesters):
  mode: ingest
  cores: 96
  io_threads: 20
  datafusion_threads: 76

Nodes 5-8 (Query nodes):
  mode: query
  cores: 64
  io_threads: 4
  datafusion_threads: 60

Nodes 9-10 (Dedicated compactors):
  mode: compact
  cores: 32
  io_threads: 2
  datafusion_threads: 30

Nodes 11-12 (Process nodes):
  mode: process
  cores: 32
  io_threads: 6
  datafusion_threads: 26
```

---

## Quick Reference

### Help Options

| Option | Description |
|--------|-------------|
| `-h`, `--help` | Print help information |
| `--help-all` | Print detailed help information |

### Quick-Start Mode

For development/testing, run without `serve` subcommand for auto-configuration:

```bash
# Zero-config startup
influxdb3

# Override specific defaults
influxdb3 --object-store memory
```

Auto-generated values:
- **node-id**: `{hostname}-node`
- **cluster-id**: `{hostname}-cluster`
- **object-store**: `file`
- **data-dir**: `~/.influxdb`

---

## References

- [InfluxDB 3 Enterprise Documentation](https://docs.influxdata.com/influxdb3/enterprise/)
- [Configuration Options Reference](https://docs.influxdata.com/influxdb3/enterprise/reference/config-options/)
- [CLI Reference](https://docs.influxdata.com/influxdb3/enterprise/reference/cli/influxdb3/serve/)
- [Clustering Configuration](https://docs.influxdata.com/influxdb3/enterprise/admin/clustering/)
