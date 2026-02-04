# FROM apache/zeppelin:0.11.1

# ENV SPARK_VERSION=3.5.0
# ENV HADOOP_VERSION=3
# ENV SPARK_HOME=/opt/spark

# USER root

# RUN apt-get update && apt-get install -y curl tar \
#  && curl -L https://archive.apache.org/dist/spark/spark-${SPARK_VERSION}/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz \
#  | tar -xz -C /opt \
#  && mv /opt/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION} /opt/spark \
#  && rm -rf /var/lib/apt/lists/*

# ENV PATH=$SPARK_HOME/bin:$PATH

FROM apache/zeppelin:0.11.1

ENV SPARK_VERSION=3.5.0
ENV HADOOP_VERSION=3
ENV HIVE_VERSION=3.1.3

ENV SPARK_HOME=/opt/spark
ENV HIVE_HOME=/opt/hive

USER root

# ===== Installer Spark =====
RUN apt-get update && apt-get install -y curl tar wget \
 && curl -L https://archive.apache.org/dist/spark/spark-${SPARK_VERSION}/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz \
 | tar -xz -C /opt \
 && mv /opt/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION} /opt/spark

# ===== Installer Hive =====
RUN wget https://archive.apache.org/dist/hive/hive-${HIVE_VERSION}/apache-hive-${HIVE_VERSION}-bin.tar.gz \
 && tar -xzf apache-hive-${HIVE_VERSION}-bin.tar.gz -C /opt \
 && mv /opt/apache-hive-${HIVE_VERSION}-bin /opt/hive \
 && rm apache-hive-${HIVE_VERSION}-bin.tar.gz

# ===== Variables d'environnement =====
ENV PATH=$SPARK_HOME/bin:$HIVE_HOME/bin:$PATH
ENV SPARK_CLASSPATH=$HIVE_HOME/lib/*


