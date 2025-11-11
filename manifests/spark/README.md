# Spark Example Manifests

Example SparkApplication manifests demonstrating lakehouse format usage.

## Important: Namespace Configuration

These manifests are **templates** without a hardcoded namespace. When deploying, either:

1. **Apply with namespace flag** (recommended):
   ```bash
   kubectl apply -f spark-pi.yaml -n <your-namespace>
   ```

2. **Add namespace to manifest before applying**:
   ```yaml
   metadata:
     name: spark-pi
     namespace: my-environment  # Add this line
   ```

The SparkApplication will run in the namespace you specify, keeping workloads isolated per environment.

## Examples

| File | Description | Format |
|------|-------------|--------|
| `spark-pi.yaml` | Calculate Pi (basic test) | N/A |
| `delta-write.yaml` | Write to Delta Lake | Delta |
| `iceberg-etl.yaml` | ETL with Iceberg | Iceberg |
| `hudi-upsert.yaml` | Upsert with Hudi | Hudi |
| `s3-credentials-secret.yaml` | S3 credentials template | N/A |

## Prerequisites

**Note:** The Spark Operator is installed as part of the Bifrost Core umbrella chart and watches all configured namespaces.

1. **Ensure the namespace exists and has a ServiceAccount for Spark**:
   ```bash
   # Create your environment namespace if it doesn't exist
   kubectl create namespace my-environment

   # Create ServiceAccount for Spark jobs
   kubectl create serviceaccount spark -n my-environment

   # Grant permissions to create executor pods
   kubectl create clusterrolebinding spark-role-my-environment \
     --clusterrole=edit \
     --serviceaccount=my-environment:spark
   ```

2. **Create S3 credentials secret** (for lakehouse examples):
   ```bash
   kubectl create secret generic s3-credentials \
     --from-literal=access-key=<your-key> \
     --from-literal=secret-key=<your-secret> \
     -n my-environment
   ```

## Running Examples

```bash
# Test Spark Operator (use -n flag to specify namespace)
kubectl apply -f spark-pi.yaml -n my-environment

# Delta Lake example
kubectl apply -f delta-write.yaml -n my-environment

# Iceberg example
kubectl apply -f iceberg-etl.yaml -n my-environment

# Hudi example
kubectl apply -f hudi-upsert.yaml -n my-environment
```

## Monitoring

```bash
# List applications in your namespace
kubectl get sparkapplications -n my-environment

# View logs
kubectl logs -n my-environment <app-name>-driver

# Spark UI
kubectl port-forward -n my-environment <app-name>-driver 4040:4040
```
