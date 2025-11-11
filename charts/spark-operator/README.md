# Spark Operator Helm Chart

Kubernetes operator for managing Apache Spark applications using the `SparkApplication` custom resource.

## Overview

This chart deploys the [Spark Operator](https://github.com/GoogleCloudPlatform/spark-on-k8s-operator) which enables running Apache Spark applications natively on Kubernetes with support for:

- Delta Lake
- Apache Iceberg
- Apache Hudi
- S3/MinIO/Azure/GCS object storage
- Hive Metastore integration

## Prerequisites

- Kubernetes 1.24+
- Helm 3.x
- kubectl configured with cluster access

## Installation

### Add Helm Repository

```bash
helm repo add spark-operator https://googlecloudplatform.github.io/spark-on-k8s-operator
helm repo update
```

### Install Chart

```bash
# Basic installation
helm install spark-operator . -n bifrost-core --create-namespace

# With custom values
helm install spark-operator . \
  -n bifrost-core \
  --create-namespace \
  -f custom-values.yaml
```

### Verify Installation

```bash
# Check operator pod
kubectl get pods -n bifrost-core -l app.kubernetes.io/name=spark-operator

# Check CRDs
kubectl get crd | grep sparkoperator

# Check webhook
kubectl get validatingwebhookconfigurations | grep spark
```

## Configuration

### Key Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `spark-operator.image.tag` | Spark Operator image tag | `v1beta2-1.4.3-3.5.0` |
| `spark-operator.replicaCount` | Number of operator replicas | `1` |
| `spark-operator.sparkJobNamespace` | Namespace to watch | `bifrost-core` |
| `spark-operator.webhook.enable` | Enable admission webhook | `true` |
| `bifrost.sparkImage.repository` | Default Spark image | `docker.io/library/spark` |
| `bifrost.sparkImage.tag` | Default Spark image tag | `4.0.0` |

| `bifrost.defaults.driver.memory` | Default driver memory | `1g` |
| `bifrost.defaults.executor.instances` | Default executor count | `2` |

### Example Custom Values

```yaml
# custom-values.yaml
spark-operator:
  replicaCount: 2  # HA setup
  resources:
    limits:
      cpu: 1000m
      memory: 1Gi

bifrost:
  defaults:
    driver:
      cores: 2
      memory: "4g"
    executor:
      cores: 4
      instances: 5
      memory: "8g"

  objectStorage:
    type: s3
    endpoint: s3.amazonaws.com
    bucket: my-bifrost-data

  metastore:
    uri: "thrift://hive-metastore.bifrost-core.svc:9083"
```

## Usage

### Submit a Spark Job

```bash
kubectl apply -f ../../manifests/spark/spark-pi.yaml
```

### Monitor Jobs

```bash
# List all SparkApplications
kubectl get sparkapplications -n bifrost-core

# Get job details
kubectl describe sparkapplication spark-pi -n bifrost-core

# View logs
kubectl logs -n bifrost-core spark-pi-driver
```

### Delete a Job

```bash
kubectl delete sparkapplication spark-pi -n bifrost-core
```

## Metrics

The operator exposes Prometheus metrics on port 10254:

```bash
# Port-forward to metrics endpoint
kubectl port-forward -n bifrost-core svc/spark-operator-metrics 10254:10254

# Scrape metrics
curl http://localhost:10254/metrics
```

### Key Metrics

- `spark_app_count` - Number of SparkApplications
- `spark_app_submit_count` - Total submissions
- `spark_app_success_count` - Successful completions
- `spark_app_failure_count` - Failed applications
- `spark_app_running_count` - Currently running apps

## Troubleshooting

### Operator Not Starting

```bash
# Check logs
kubectl logs -n bifrost-core -l app.kubernetes.io/name=spark-operator

# Check RBAC
kubectl auth can-i create sparkapplications --as=system:serviceaccount:bifrost-core:spark-operator -n bifrost-core
```

### SparkApplication Stuck

```bash
# Check events
kubectl describe sparkapplication <name> -n bifrost-core

# Check resource availability
kubectl top nodes
kubectl describe nodes
```

### Webhook Issues

If webhook causes issues, temporarily disable it:

```bash
helm upgrade spark-operator . \
  -n bifrost-core \
  --set spark-operator.webhook.enable=false
```

## Uninstallation

```bash
# Uninstall chart
helm uninstall spark-operator -n bifrost-core

# Clean up CRDs (optional)
kubectl delete crd sparkapplications.sparkoperator.k8s.io
kubectl delete crd scheduledsparkapplications.sparkoperator.k8s.io
```

## References

- [Spark Operator Documentation](https://github.com/GoogleCloudPlatform/spark-on-k8s-operator)
- [SparkApplication API](https://github.com/GoogleCloudPlatform/spark-on-k8s-operator/blob/master/docs/api-docs.md)
- [Apache Spark on Kubernetes](https://spark.apache.org/docs/latest/running-on-kubernetes.html)
