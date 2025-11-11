# Bifrost Spark Docker Image

Custom Apache Spark image with lakehouse format support.

## Features

- **Apache Spark 3.5.0** - Latest stable Spark release
- **Delta Lake 3.0.0** - ACID transactions on data lakes
- **Apache Iceberg 1.4.3** - Netflix's table format
- **Apache Hudi 0.14.1** - Uber's incremental processing
- **AWS SDK** - S3 and MinIO support
- **Azure Storage** - Azure Blob Storage support

## Building

```bash
# Build image
docker build -t docker.io/library/spark:4.0.0 .

# Build for multiple platforms
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t docker.io/library/spark:4.0.0 \
  --push .
## Building (optional)

This repository includes Docker build assets for a custom Spark image with lakehouse format support. If you don't need a custom image, the charts and examples default to the public Docker Hub Spark image `docker.io/library/spark:4.0.0` (recommended for local testing).

If you do want to build and publish a custom image (for example to GitHub Container Registry), use the commands below and replace the image name with your target registry.

```bash
# Build image locally (example tag)
docker build -t ghcr.io/<your-org>/spark:3.5.0 .

# Build for multiple platforms and push (example)
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t ghcr.io/<your-org>/spark:3.5.0 \
  --push .
```
```

## Pushing

```bash
# Login to GitHub Container Registry
echo $GITHUB_TOKEN | docker login ghcr.io -u $GITHUB_USER --password-stdin

# Push image
docker push docker.io/library/spark:4.0.0

# Tag as latest
## Pushing (optional)

If you built a custom image and want to publish it to a registry, tag and push it to your chosen registry (GHCR, Docker Hub, ECR, etc.). Example for GHCR:

```bash
# Login to GitHub Container Registry
echo $GITHUB_TOKEN | docker login ghcr.io -u $GITHUB_USER --password-stdin

# Push image (replace <your-org>)
docker push ghcr.io/<your-org>/spark:3.5.0

# Tag as latest and push
docker tag ghcr.io/<your-org>/spark:3.5.0 ghcr.io/<your-org>/spark:latest
docker push ghcr.io/<your-org>/spark:latest
```
docker tag docker.io/library/spark:4.0.0 docker.io/library/spark:latest
docker push docker.io/library/spark:latest
```

## Usage

### In SparkApplication

```yaml
spec:
  image: "docker.io/library/spark:4.0.0"
  imagePullPolicy: IfNotPresent
```

### Local Testing

```bash
# Run Spark shell with Delta
docker run -it --rm docker.io/library/spark:4.0.0 \
  spark-shell --conf spark.sql.extensions=io.delta.sql.DeltaSparkSessionExtension

# Run PySpark
docker run -it --rm docker.io/library/spark:4.0.0 pyspark
```

## Included JARs

Located in `/opt/spark/jars/`:

- `delta-core_2.12-3.0.0.jar`
- `delta-storage-3.0.0.jar`
- `iceberg-spark-runtime-3.5_2.12-1.4.3.jar`
- `hudi-spark3.5-bundle_2.12-0.14.1.jar`
- `aws-java-sdk-bundle-1.12.262.jar`
- `hadoop-aws-3.3.4.jar`
- `hadoop-azure-3.3.4.jar`

## Pre-configured Settings

Default Spark configurations in `/opt/spark/conf/spark-defaults.conf`:

```properties
spark.sql.extensions=io.delta.sql.DeltaSparkSessionExtension,org.apache.iceberg.spark.extensions.IcebergSparkSessionExtension
spark.sql.catalog.spark_catalog=org.apache.spark.sql.delta.catalog.DeltaCatalog
spark.sql.catalog.iceberg=org.apache.iceberg.spark.SparkCatalog
spark.sql.catalog.iceberg.type=hive
spark.serializer=org.apache.spark.serializer.KryoSerializer
spark.sql.adaptive.enabled=true
```

## Testing

```bash
# Test Delta Lake
docker run -it --rm docker.io/library/spark:4.0.0 \
  spark-shell --packages io.delta:delta-core_2.12:3.0.0

# Test Iceberg
docker run -it --rm docker.io/library/spark:4.0.0 \
  spark-shell --packages org.apache.iceberg:iceberg-spark-runtime-3.5_2.12:1.4.3
```

## Security Scanning

```bash
# Scan for vulnerabilities
trivy image docker.io/library/spark:4.0.0
```

## Size

- Compressed: ~1.5 GB
- Uncompressed: ~4.0 GB
