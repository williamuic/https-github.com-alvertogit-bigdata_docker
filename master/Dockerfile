FROM openjdk:11-jdk-slim as spark

# FROM gettyimages/spark

RUN apt-get update \
    && apt-get install -y curl unzip \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

ENV LANG=C.UTF-8 LC_ALL=C.UTF-8

# HADOOP

ENV HADOOP_VERSION 3.3.0
ENV HADOOP_HOME /usr/hadoop-$HADOOP_VERSION
ENV HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
ENV PATH $PATH:$HADOOP_HOME/bin
RUN curl -sL --retry 3 \
    "https://www-eu.apache.org/dist/hadoop/common/hadoop-$HADOOP_VERSION/hadoop-$HADOOP_VERSION.tar.gz" \
    | gunzip \
    | tar -x -C /usr/ \
    && rm -rf $HADOOP_HOME/share/doc \
    && chown -R root:root $HADOOP_HOME

# SPARK

ENV SPARK_VERSION 3.0.1
ENV SPARK_PACKAGE spark-${SPARK_VERSION}-bin-without-hadoop
ENV SPARK_HOME /usr/spark-${SPARK_VERSION}
ENV SPARK_DIST_CLASSPATH="$HADOOP_HOME/etc/hadoop/*:$HADOOP_HOME/share/hadoop/common/lib/*:$HADOOP_HOME/share/hadoop/common/*:$HADOOP_HOME/share/hadoop/hdfs/*:$HADOOP_HOME/share/hadoop/hdfs/lib/*:$HADOOP_HOME/share/hadoop/hdfs/*:$HADOOP_HOME/share/hadoop/yarn/lib/*:$HADOOP_HOME/share/hadoop/yarn/*:$HADOOP_HOME/share/hadoop/mapreduce/lib/*:$HADOOP_HOME/share/hadoop/mapreduce/*:$HADOOP_HOME/share/hadoop/tools/lib/*"
ENV PATH $PATH:${SPARK_HOME}/bin
RUN curl -sL --retry 3 \
    "https://www.apache.org/dyn/mirrors/mirrors.cgi?action=download&filename=spark/spark-${SPARK_VERSION}/${SPARK_PACKAGE}.tgz" \
    | gunzip \
    | tar x -C /usr/ \
    && mv /usr/$SPARK_PACKAGE $SPARK_HOME \
    && chown -R root:root $SPARK_HOME

WORKDIR $SPARK_HOME

CMD ["bin/spark-class", "org.apache.spark.deploy.master.Master"]

FROM spark as spark-master

# PYTHON

RUN apt-get update --fix-missing \
    && apt-get install -y wget bzip2 ca-certificates git python3.7 python3.7-venv python3-pip \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN ln -s /usr/bin/python3.7 /usr/bin/python
RUN ln -s /usr/bin/pip3 /usr/bin/pip

# MAVEN 

WORKDIR /usr/spark-${SPARK_VERSION}/

ENV MAVEN_VERSION 3.6.3
RUN mkdir /opt/apache-maven
RUN wget --quiet -c https://apache.rediris.es/maven/maven-3/${MAVEN_VERSION}/binaries/apache-maven-${MAVEN_VERSION}-bin.tar.gz -O apache-maven.tar.gz && \
    tar xzvf apache-maven.tar.gz -C /opt/apache-maven --strip-components=1 && \
    rm -f apache-maven.tar.gz && \
    ln -s /opt/apache-maven/bin/mvn /usr/bin/mvn

ENV MAVEN_HOME /opt/apache-maven
ENV PATH MAVEN_HOME:$PATH

# Work dir files

RUN mkdir /usr/spark-${SPARK_VERSION}/work_dir
WORKDIR /usr/spark-${SPARK_VERSION}/work_dir
ADD ./work_dir /usr/spark-${SPARK_VERSION}/work_dir

# Virtual environment

RUN python -m venv .
RUN pip install pip==20.2.4
RUN pip install setuptools==51.0.0
RUN pip install --no-cache-dir -r requirements.txt

# Compiling Scala example with MAVEN

WORKDIR /usr/spark-${SPARK_VERSION}/work_dir/scala_apps/example
RUN mvn package

WORKDIR $SPARK_HOME
CMD ["bin/spark-class", "org.apache.spark.deploy.master.Master"]
