# Trino Helm Chart

Fast distributed SQL query engine for querying lakehouse formats (Delta Lake, Iceberg, Hudi).

## Overview

This chart deploys [Trino](https://trino.io/) (formerly PrestoSQL) configured to query:
- Apache Iceberg tables
- Delta Lake tables
- Apache Hudi tables
- Traditional Hive tables

## Prerequisites

- Kubernetes 1.24+
- Helm 3.x
- Hive Metastore (see `../metastore/`)
- Object storage (S3/MinIO/Azure/GCS)

## Installation

### Add Helm Repository

```bash
helm repo add trino https://trinodb.github.io/charts
helm repo update
```

### Install Chart

```bash
# Basic installation
helm install trino . -n bifrost-core

# With custom values
helm install trino . \
  -n bifrost-core \
  -f custom-values.yaml
```

### Verify Installation

```bash
# Check pods
kubectl get pods -n bifrost-core -l app=trino

# Check coordinator
kubectl get pods -n bifrost-core -l component=coordinator

# Check workers
kubectl get pods -n bifrost-core -l component=worker
```

## Configuration

### Key Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `trino.image.tag` | Trino version | `431` |
| `trino.server.workers` | Number of worker nodes | `2` |
| `trino.coordinator.resources.memory` | Coordinator memory | `8Gi` |
| `trino.worker.resources.memory` | Worker memory | `8Gi` |
| `bifrost.metastore.uri` | Hive Metastore URI | `thrift://hive-metastore:9083` |

### Example Custom Values

```yaml
# custom-values.yaml
trino:
  server:
    workers: 5  # Scale workers

  coordinator:
    resources:
      requests:
        memory: "8Gi"
        cpu: "4"
      limits:
        memory: "16Gi"
        cpu: "8"

  worker:
    resources:
      requests:
        memory: "8Gi"
        cpu: "4"
      limits:
        memory: "16Gi"
        cpu: "8"

bifrost:
  objectStorage:
    type: s3
    endpoint: s3.amazonaws.com
    bucket: my-data
```

## Usage

### Connect to Trino CLI

```bash
# Port-forward to Trino coordinator
kubectl port-forward -n bifrost-core svc/trino 8080:8080

# Connect with Trino CLI
docker run -it --rm --network host trinodb/trino:431 trino --server localhost:8080

# Or exec into coordinator pod
kubectl exec -it -n bifrost-core <coordinator-pod> -- trino
```

### Query Examples

```sql
-- Show catalogs
SHOW CATALOGS;

-- Iceberg queries
SHOW SCHEMAS FROM iceberg;
SHOW TABLES FROM iceberg.default;
SELECT * FROM iceberg.default.my_table LIMIT 10;

-- Delta Lake queries
SHOW SCHEMAS FROM delta;
SELECT * FROM delta.default.employees;

-- Hudi queries
SHOW SCHEMAS FROM hudi;
SELECT * FROM hudi.default.orders;

-- Cross-catalog join
SELECT
  i.id,
  i.name,
  d.department
FROM iceberg.default.users i
JOIN delta.default.departments d ON i.dept_id = d.id;
```

### Query Performance Tuning

```sql
-- Set session properties for better performance
SET SESSION query_max_memory = '10GB';
SET SESSION join_distribution_type = 'PARTITIONED';
SET SESSION task_concurrency = 16;
```

## Catalogs

### Iceberg Catalog

Properties:
```properties
connector.name=iceberg
iceberg.catalog.type=hive_metastore
hive.metastore.uri=thrift://hive-metastore:9083
iceberg.file-format=PARQUET
```

### Delta Lake Catalog

Properties:
```properties
connector.name=delta_lake
hive.metastore.uri=thrift://hive-metastore:9083
delta.enable-non-concurrent-writes=true
```

### Hudi Catalog

Properties:
```properties
connector.name=hudi
hive.metastore.uri=thrift://hive-metastore:9083
```

## Monitoring

### Access Trino Web UI

```bash
# Port-forward to web UI
kubectl port-forward -n bifrost-core svc/trino 8080:8080

# Open in browser
open http://localhost:8080
```

### Prometheus Metrics

```bash
# Port-forward to JMX exporter
kubectl port-forward -n bifrost-core <coordinator-pod> 9090:9090

# Scrape metrics
curl http://localhost:9090/metrics
```

### Key Metrics

- `trino_execution_QueryManager_RunningQueries` - Active queries
- `trino_execution_QueryManager_QueuedQueries` - Queued queries
- `trino_memory_ClusterMemoryPool_FreeBytes` - Available memory
- `trino_execution_executor_RunningSplits` - Running splits

## Troubleshooting

### Connection Issues

```bash
# Check coordinator logs
kubectl logs -n bifrost-core -l component=coordinator

# Check worker logs
kubectl logs -n bifrost-core -l component=worker

# Test metastore connectivity
kubectl run -it --rm debug --image=busybox -- \
  telnet hive-metastore 9083
```

### Query Failures

```sql
-- Check query details in Trino UI
-- Or query system tables
SELECT * FROM system.runtime.queries WHERE state = 'FAILED';
```

### Out of Memory

Increase coordinator/worker memory:

```yaml
trino:
  coordinator:
    jvm:
      maxHeapSize: "16G"
  worker:
    jvm:
      maxHeapSize: "16G"
```

## Scaling

### Scale Workers

```bash
# Via Helm
helm upgrade trino . \
  -n bifrost-core \
  --set trino.server.workers=5

# Via kubectl
kubectl scale deployment trino-worker -n bifrost-core --replicas=5
```

## Uninstallation

```bash
helm uninstall trino -n bifrost-core
```

## References

- [Trino Documentation](https://trino.io/docs/current/)
- [Iceberg Connector](https://trino.io/docs/current/connector/iceberg.html)
- [Delta Lake Connector](https://trino.io/docs/current/connector/delta-lake.html)
- [Hudi Connector](https://trino.io/docs/current/connector/hudi.html)
