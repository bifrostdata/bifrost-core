# Hive Metastore Helm Chart

Hive Metastore with PostgreSQL backend for managing lakehouse table metadata.

## Overview

This chart deploys:
- **Hive Metastore** - Centralized metadata repository for lakehouse tables
- **PostgreSQL** - Backend database for storing metadata

The metastore is used by:
- Apache Spark (via SparkApplications)
- Trino (for querying tables)
- Iceberg, Delta Lake, and Hudi (for table management)

## Prerequisites

- Kubernetes 1.24+
- Helm 3.x
- Object storage (S3/MinIO/Azure/GCS)
- Persistent volumes for PostgreSQL

## Installation

### Install Chart

```bash
# Basic installation
helm install metastore . -n bifrost-core --create-namespace

# With custom values
helm install metastore . \
  -n bifrost-core \
  --create-namespace \
  -f custom-values.yaml
```

### Verify Installation

```bash
# Check pods
kubectl get pods -n bifrost-core -l app=hive-metastore

# Check PostgreSQL
kubectl get pods -n bifrost-core -l app.kubernetes.io/name=postgresql

# Check service
kubectl get svc -n bifrost-core hive-metastore
```

## Configuration

### Key Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `postgresql.enabled` | Enable PostgreSQL dependency | `true` |
| `postgresql.auth.database` | Metastore database name | `metastore` |
| `postgresql.primary.persistence.size` | PostgreSQL storage size | `10Gi` |
| `hive.image.tag` | Hive Metastore version | `3.1.3` |
| `hive.replicaCount` | Number of metastore replicas | `1` |
| `hive.config.warehouse.dir` | Warehouse location | `s3a://bifrost-data/warehouse` |

### Example Custom Values

```yaml
# custom-values.yaml
postgresql:
  auth:
    password: "my-secure-password"
  primary:
    persistence:
      size: 20Gi
    resources:
      limits:
        memory: "2Gi"
        cpu: "2000m"

hive:
  replicaCount: 2  # HA setup
  resources:
    limits:
      memory: "4Gi"
      cpu: "2000m"

bifrost:
  objectStorage:
    type: s3
    endpoint: s3.amazonaws.com
    bucket: my-data-lake
    warehouse: "/warehouse"
```

## Usage

### Test Connectivity

```bash
# From a Spark pod
kubectl exec -it -n bifrost-core <spark-pod> -- bash

# Test thrift connection
telnet hive-metastore 9083
```

### View Metastore Tables

Connect via beeline CLI:

```bash
# Port-forward
kubectl port-forward -n bifrost-core svc/hive-metastore 9083:9083

# Connect with beeline
docker run -it --rm --network host apache/hive:3.1.3 \
  beeline -u "jdbc:hive2://localhost:10000"

# Show databases
SHOW DATABASES;

# Show tables
USE default;
SHOW TABLES;

# Describe table
DESCRIBE FORMATTED my_table;
```

### Query Metadata Directly (PostgreSQL)

```bash
# Connect to PostgreSQL
kubectl exec -it -n bifrost-core metastore-postgresql-0 -- \
  psql -U hive -d metastore

# Query tables
SELECT * FROM "TBLS";
SELECT * FROM "DBS";
SELECT * FROM "PARTITIONS";
```

## Schema Management

### Initialize Schema

Schema is automatically initialized on first startup. To manually run:

```bash
kubectl exec -it -n bifrost-core <metastore-pod> -- \
  schematool -dbType postgres -initSchema
```

### Upgrade Schema

```bash
kubectl exec -it -n bifrost-core <metastore-pod> -- \
  schematool -dbType postgres -upgradeSchema
```

### Validate Schema

```bash
kubectl exec -it -n bifrost-core <metastore-pod> -- \
  schematool -dbType postgres -info
```

## Backup & Restore

### Backup PostgreSQL

```bash
# Dump metastore database
kubectl exec -it -n bifrost-core metastore-postgresql-0 -- \
  pg_dump -U hive metastore > metastore-backup.sql
```

### Restore PostgreSQL

```bash
# Restore from backup
kubectl exec -i -n bifrost-core metastore-postgresql-0 -- \
  psql -U hive -d metastore < metastore-backup.sql
```

## High Availability

### Run Multiple Metastore Replicas

```yaml
hive:
  replicaCount: 3
```

### PostgreSQL HA

For production, consider:
- PostgreSQL with replication (Bitnami chart supports this)
- External managed PostgreSQL (AWS RDS, Azure Database, etc.)

```yaml
postgresql:
  enabled: false  # Disable bundled PostgreSQL

hive:
  config:
    database:
      host: "my-external-postgres.example.com"
      port: 5432
```

## Monitoring

### Prometheus Metrics

Metastore exposes JMX metrics:

```bash
# Port-forward to metrics
kubectl port-forward -n bifrost-core <metastore-pod> 9404:9404

# Scrape metrics
curl http://localhost:9404/metrics
```

### Health Check

```bash
# Check if metastore is responding
kubectl exec -it -n bifrost-core <metastore-pod> -- \
  curl -f http://localhost:9083 || echo "Metastore not responding"
```

## Troubleshooting

### Metastore Not Starting

```bash
# Check logs
kubectl logs -n bifrost-core -l app=hive-metastore

# Common issues:
# 1. PostgreSQL not ready
kubectl get pods -n bifrost-core -l app.kubernetes.io/name=postgresql

# 2. Schema not initialized
kubectl exec -it -n bifrost-core <metastore-pod> -- \
  schematool -dbType postgres -info
```

### Connection Issues

```bash
# Test PostgreSQL connectivity
kubectl run -it --rm psql-test --image=postgres:14 -- \
  psql -h metastore-postgresql -U hive -d metastore

# Test Thrift port
kubectl run -it --rm telnet-test --image=busybox -- \
  telnet hive-metastore 9083
```

### Schema Version Mismatch

```bash
# Check current schema version
kubectl exec -it -n bifrost-core <metastore-pod> -- \
  schematool -dbType postgres -info

# Upgrade if needed
kubectl exec -it -n bifrost-core <metastore-pod> -- \
  schematool -dbType postgres -upgradeSchema
```

## Uninstallation

```bash
# Uninstall chart (keeps PVCs)
helm uninstall metastore -n bifrost-core

# Delete PVCs (WARNING: deletes all metadata)
kubectl delete pvc -n bifrost-core -l app.kubernetes.io/name=postgresql
```

## References

- [Apache Hive Metastore Documentation](https://cwiki.apache.org/confluence/display/Hive/Design)
- [Hive Metastore Schema Tool](https://cwiki.apache.org/confluence/display/Hive/Hive+Schema+Tool)
- [PostgreSQL Chart](https://github.com/bitnami/charts/tree/main/bitnami/postgresql)
