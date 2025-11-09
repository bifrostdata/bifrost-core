# Bifrost Core

> Data plane: Spark, Trino, Metadata, and Lakehouse components

## Overview

The core repository contains all data processing and query engine components that form the foundation of the Bifrost lakehouse platform.

## Components

### 🔥 Apache Spark
- **Spark Operator**: Kubernetes-native Spark job management
- **Spark Executors**: Scalable compute for ETL workloads
- **Delta Lake Support**: ACID transactions on data lakes
- **Iceberg Support**: Netflix's table format for big data
- **Hudi Support**: Uber's incremental data processing

### 🔍 Trino/Presto
- **Query Engine**: Fast, distributed SQL query engine
- **Connectors**: S3, PostgreSQL, MySQL, and more
- **Federation**: Query across multiple data sources

### 📊 Metadata Management
- **Hive Metastore**: Schema and table catalog
- **PostgreSQL**: Persistent metadata storage
- **Schema Registry**: Data schema versioning

## Architecture

```
bifrost-core/
├── spark/
│   ├── operator/           # Spark Operator Helm charts
│   ├── docker/             # Custom Spark images
│   └── examples/           # Sample Spark jobs
├── trino/
│   ├── helm/               # Trino Helm charts
│   ├── catalogs/           # Catalog configurations
│   └── connectors/         # Custom connectors
├── metadata/
│   ├── hive-metastore/     # HMS deployment
│   └── postgres/           # Metadata DB
├── storage/
│   ├── delta/              # Delta Lake configs
│   ├── iceberg/            # Iceberg configs
│   └── hudi/               # Hudi configs
└── scripts/
    └── init/               # Initialization scripts
```

## Prerequisites

- Kubernetes 1.24+
- Object Storage (S3, MinIO, Azure Blob, GCS)
- PostgreSQL 12+ (for metadata)

## Installation

```bash
# Install via Helm
helm install bifrost-core ./helm/bifrost-core \
  --namespace bifrost-core \
  --create-namespace \
  --set objectStorage.endpoint=s3.amazonaws.com \
  --set objectStorage.bucket=bifrost-data

# Verify installation
kubectl get pods -n bifrost-core
```

## Configuration

### Object Storage

```yaml
objectStorage:
  type: s3  # s3, minio, azure, gcs
  endpoint: s3.amazonaws.com
  bucket: bifrost-data
  accessKey: <access-key>
  secretKey: <secret-key>
```

### Spark Configuration

```yaml
spark:
  executor:
    instances: 3
    memory: "4g"
    cores: 2
  driver:
    memory: "2g"
    cores: 1
```

## Usage

### Submit a Spark Job

```bash
kubectl apply -f - <<EOF
apiVersion: sparkoperator.k8s.io/v1beta2
kind: SparkApplication
metadata:
  name: spark-pi
  namespace: bifrost-core
spec:
  type: Scala
  mode: cluster
  image: "ghcr.io/bifrost/spark:3.5.0"
  mainClass: org.apache.spark.examples.SparkPi
  mainApplicationFile: "local:///opt/spark/examples/jars/spark-examples.jar"
  sparkVersion: "3.5.0"
  driver:
    cores: 1
    memory: "512m"
  executor:
    cores: 1
    instances: 2
    memory: "512m"
EOF
```

### Query with Trino

```bash
# Connect to Trino CLI
kubectl exec -it -n bifrost-core trino-coordinator-0 -- trino

# Run queries
SHOW CATALOGS;
SHOW SCHEMAS FROM iceberg;
SELECT * FROM iceberg.default.my_table LIMIT 10;
```

## Development

### Building Custom Spark Images

```bash
cd spark/docker
docker build -t bifrost-spark:latest .
docker push ghcr.io/bifrost/spark:latest
```

### Running Tests

```bash
# Unit tests
make test

# Integration tests
make test-integration
```

## Monitoring

Metrics are exported to Prometheus:
- Spark metrics: `spark_*`
- Trino metrics: `trino_*`
- HMS metrics: `hive_metastore_*`

## Troubleshooting

### Spark Jobs Failing

```bash
# Check Spark Operator logs
kubectl logs -n bifrost-core -l app=spark-operator

# Check executor logs
kubectl logs -n bifrost-core <spark-executor-pod>
```

### Trino Connection Issues

```bash
# Check Trino coordinator logs
kubectl logs -n bifrost-core trino-coordinator-0

# Verify catalog configuration
kubectl exec -n bifrost-core trino-coordinator-0 -- cat /etc/trino/catalog/iceberg.properties
```

## Contributing

See [CONTRIBUTING.md](../CONTRIBUTING.md)

## License

Apache 2.0
