# Custom Bifrost Spark Image
# Includes Delta Lake, Apache Iceberg, and Apache Hudi support

FROM apache/spark:3.5.0-scala2.12-java11-python3-ubuntu

USER root

# Install system dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    curl \
    wget \
    unzip \
    ca-certificates && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Set versions for lakehouse formats
ENV DELTA_VERSION=3.0.0 \
    ICEBERG_VERSION=1.4.3 \
    HUDI_VERSION=0.14.1 \
    AWS_SDK_VERSION=1.12.262 \
    HADOOP_VERSION=3.3.4

WORKDIR /opt/spark/jars

# Download Delta Lake JARs
RUN curl -L -o delta-core_2.12-${DELTA_VERSION}.jar \
    https://repo1.maven.org/maven2/io/delta/delta-core_2.12/${DELTA_VERSION}/delta-core_2.12-${DELTA_VERSION}.jar && \
    curl -L -o delta-storage-${DELTA_VERSION}.jar \
    https://repo1.maven.org/maven2/io/delta/delta-storage/${DELTA_VERSION}/delta-storage-${DELTA_VERSION}.jar

# Download Apache Iceberg JAR
RUN curl -L -o iceberg-spark-runtime-3.5_2.12-${ICEBERG_VERSION}.jar \
    https://repo1.maven.org/maven2/org/apache/iceberg/iceberg-spark-runtime-3.5_2.12/${ICEBERG_VERSION}/iceberg-spark-runtime-3.5_2.12-${ICEBERG_VERSION}.jar

# Download Apache Hudi JAR
RUN curl -L -o hudi-spark3.5-bundle_2.12-${HUDI_VERSION}.jar \
    https://repo1.maven.org/maven2/org/apache/hudi/hudi-spark3.5-bundle_2.12/${HUDI_VERSION}/hudi-spark3.5-bundle_2.12-${HUDI_VERSION}.jar

# Download AWS SDK for S3 support
RUN curl -L -o aws-java-sdk-bundle-${AWS_SDK_VERSION}.jar \
    https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk-bundle/${AWS_SDK_VERSION}/aws-java-sdk-bundle-${AWS_SDK_VERSION}.jar && \
    curl -L -o hadoop-aws-${HADOOP_VERSION}.jar \
    https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-aws/${HADOOP_VERSION}/hadoop-aws-${HADOOP_VERSION}.jar

# Download Azure Storage support
RUN curl -L -o hadoop-azure-${HADOOP_VERSION}.jar \
    https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-azure/${HADOOP_VERSION}/hadoop-azure-${HADOOP_VERSION}.jar

# Set proper permissions
RUN chmod -R 755 /opt/spark/jars && \
    chown -R spark:spark /opt/spark/jars

# Create spark-defaults.conf with common configurations
RUN echo "spark.sql.extensions=io.delta.sql.DeltaSparkSessionExtension,org.apache.iceberg.spark.extensions.IcebergSparkSessionExtension" > /opt/spark/conf/spark-defaults.conf && \
    echo "spark.sql.catalog.spark_catalog=org.apache.spark.sql.delta.catalog.DeltaCatalog" >> /opt/spark/conf/spark-defaults.conf && \
    echo "spark.sql.catalog.iceberg=org.apache.iceberg.spark.SparkCatalog" >> /opt/spark/conf/spark-defaults.conf && \
    echo "spark.sql.catalog.iceberg.type=hive" >> /opt/spark/conf/spark-defaults.conf && \
    echo "spark.serializer=org.apache.spark.serializer.KryoSerializer" >> /opt/spark/conf/spark-defaults.conf && \
    echo "spark.sql.adaptive.enabled=true" >> /opt/spark/conf/spark-defaults.conf && \
    chown -R spark:spark /opt/spark/conf

USER spark

WORKDIR /opt/spark/work-dir

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:4040 || exit 1

ENTRYPOINT ["/opt/entrypoint.sh"]
