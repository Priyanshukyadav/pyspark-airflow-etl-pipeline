FROM apache/airflow:2.8.1
USER root

# Install system dependencies with Java available for PySpark
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    openjdk-17-jre-headless \
    postgresql-client \
    curl \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

ENV JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
ENV PATH=${JAVA_HOME}/bin:${PATH}

USER airflow

# Install Python packages
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt