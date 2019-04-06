# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.
ARG BASE_CONTAINER=jupyter/scipy-notebook
FROM $BASE_CONTAINER

LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"

USER root

# Spark dependencies
ENV APACHE_SPARK_VERSION 2.4.0
ENV HADOOP_VERSION 2.7

RUN apt-get -y update && \
    apt-get install --no-install-recommends -y openjdk-8-jre-headless ca-certificates-java && \
    rm -rf /var/lib/apt/lists/*

RUN cd /tmp && \
        wget -q http://mirrors.ukfast.co.uk/sites/ftp.apache.org/spark/spark-${APACHE_SPARK_VERSION}/spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz && \
        echo "5F4184E0FE7E5C8AE67F5E6BC5DEEE881051CC712E9FF8AEDDF3529724C00E402C94BB75561DD9517A372F06C1FCB78DC7AE65DCBD4C156B3BA4D8E267EC2936 *spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz" | sha512sum -c - && \
        tar xzf spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz -C /usr/local --owner root --group root --no-same-owner && \
        rm spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz
RUN cd /usr/local && ln -s spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION} spark

# Mesos dependencies
# Install from the Xenial Mesosphere repository since there does not (yet)
# exist a Bionic repository and the dependencies seem to be compatible for now.
COPY mesos.key /tmp/
RUN apt-get -y update && \
    apt-get install --no-install-recommends -y gnupg && \
    apt-key add /tmp/mesos.key && \
    echo "deb http://repos.mesosphere.io/ubuntu xenial main" > /etc/apt/sources.list.d/mesosphere.list && \
    apt-get -y update && \
    apt-get --no-install-recommends -y install mesos=1.2\* && \
    apt-get purge --auto-remove -y gnupg && \
    rm -rf /var/lib/apt/lists/*

# Spark and Mesos config
ENV SPARK_HOME /usr/local/spark
ENV PYTHONPATH $SPARK_HOME/python:$SPARK_HOME/python/lib/py4j-0.10.7-src.zip
ENV MESOS_NATIVE_LIBRARY /usr/local/lib/libmesos.so
ENV SPARK_OPTS --driver-java-options=-Xms1024M --driver-java-options=-Xmx4096M --driver-java-options=-Dlog4j.logLevel=info



# Azure Storage provider for Spark 
ENV HADOOP_SEMVER $HADOOP_VERSION.0

RUN apt-get update -y && \
    apt-get install -y --no-install-recommends software-properties-common && \
    apt-add-repository universe && \
    apt-get install -y --no-install-recommends  maven && \
    echo "<project>" \
                    "<modelVersion>4.0.0</modelVersion>" \
                    "<groupId>groupId</groupId>" \
                    "<artifactId>artifactId</artifactId>" \
                    "<version>1.0</version>" \
                "<dependencies>" \
                    "<dependency>" \
                        "<groupId>org.apache.hadoop</groupId>" \
                        "<artifactId>hadoop-azure</artifactId>" \
                        "<version>${HADOOP_SEMVER}</version>" \
                    "<exclusions>" \
                        "<exclusion>" \
                            "<groupId>org.apache.hadoop</groupId>" \
                            "<artifactId>hadoop-common</artifactId>" \
                        "</exclusion>" \
                        "<exclusion>" \
                            "<groupId>com.fasterxml.jackson.core</groupId>" \
                            "<artifactId>jackson-core</artifactId>" \
                        "</exclusion>" \
                    "</exclusions> " \
                    "</dependency>" \
                    "<dependency>" \
                        "<groupId>com.microsoft.sqlserver</groupId>" \
                        "<artifactId>mssql-jdbc</artifactId>" \
                        "<version>6.4.0.jre8</version>" \
                    "</dependency>" \
                    "<dependency>" \
                        "<groupId>com.microsoft.azure</groupId>" \
                        "<artifactId>azure-storage</artifactId>" \
                        "<version>2.2.0</version>" \
                        "<exclusions>" \
                            "<exclusion>" \
                                "<groupId>com.fasterxml.jackson.core</groupId>" \
                                "<artifactId>jackson-core</artifactId>" \
                            "</exclusion>" \
                            "<exclusion>" \
                                "<groupId>org.apache.commons</groupId>" \
                                "<artifactId>commons-lang3</artifactId>" \
                            "</exclusion>" \
                            "<exclusion>" \
                                "<groupId>org.slf4j</groupId>" \
                                "<artifactId>slf4j-api</artifactId>" \
                            "</exclusion>" \
                        "</exclusions>" \
                    "</dependency>" \
                    "<dependency>" \
                        "<groupId>com.microsoft.azure</groupId>" \
                        "<artifactId>azure-cosmosdb-spark_2.2.0_2.11</artifactId>" \
                        "<version>1.1.1</version>" \
                        "<exclusions>" \
                            "<exclusion>" \
                                "<groupId>org.apache.tinkerpop</groupId>" \
                                "<artifactId>tinkergraph-gremlin</artifactId>" \
                            "</exclusion>" \
                            "<exclusion>" \
                                "<groupId>org.apache.tinkerpop</groupId>" \
                                "<artifactId>spark-gremlin</artifactId>" \
                            "</exclusion>" \
                            "<exclusion>" \
                                "<groupId>io.netty</groupId>" \
                                "<artifactId>*</artifactId>" \
                            "</exclusion>" \
                            "<exclusion>" \
                                "<groupId>com.fasterxml.jackson.core</groupId>" \
                                "<artifactId>jackson-annotations</artifactId>" \
                            "</exclusion>" \
                        "</exclusions> " \
                    "</dependency>" \
                "</dependencies>" \
            "</project>" > /tmp/pom.xml \
    && cd /tmp \
    && mvn dependency:copy-dependencies -DoutputDirectory="${SPARK_HOME}/jars/" \
    && apt-get --purge autoremove -y maven \
    && rm -rf /tmp/*  

USER $NB_UID

# Install pyarrow
RUN conda install --quiet -y 'pyarrow' && \
    conda clean -tipsy && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER
