
# ⚙️ Module 3: Processing Layer & ETL Pipelines (Fawry Case Study)

## 📌 Overview
Once data is ingested from millions of endpoints and stored across our hybrid storage tiers, it needs to be processed. In a FinTech ecosystem like Fawry, processing happens on two separate tracks simultaneously: **Real-time Streaming** (for fraud detection and instant notifications) and **Batch Processing** (for end-of-day merchant settlements and credit scoring).

This document breaks down the core processing engines: Apache Kafka, Apache Spark, and Apache Airflow.

---

## ⚡ 1. The Speed Layer (Apache Kafka)
Kafka acts as the distributed commit log and central nervous system of the pipeline. It handles millions of events per second with single-digit millisecond latency, decoupling data producers from consumers.

* **Core Topics:**
  * `txn-incoming`: Raw transactions hitting the API gateway.
  * `txn-fraud-alerts`: Transactions flagged instantly by AI security models.
  * `txn-settled`: Successfully processed payment confirmations.

**Simulated Kafka Consumer Configuration (Python):**
```python
from confluent_kafka import Consumer

conf = {
    'bootstrap.servers': 'kafka-cluster-01.fawry.local:9092',
    'group.id': 'fraud_detection_group',
    'auto.offset.reset': 'earliest',
    'enable.auto.commit': False
}

consumer = Consumer(conf)
consumer.subscribe(['txn-incoming'])

while True:
    msg = consumer.poll(timeout=1.0)
    if msg is None:
        continue
    if msg.error():
        print(f"Consumer error: {msg.error()}")
        continue
    
    # Process event for real-time fraud checks
    print(f"Processing live transaction: {msg.value().decode('utf-8')}")

```

---

## 🔄 2. The Batch Layer (Apache Spark)

While Kafka handles live streams, Apache Spark is utilized for heavy, end-of-day (EOD) processing. It extracts historical data, transforms it, and computes financial settlements for over 395,700 merchants.

**Simulated PySpark ETL Job (Merchant Settlement):**

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, sum, round

spark = SparkSession.builder \
    .appName("Fawry_EOD_Merchant_Settlement") \
    .getOrCreate()

# Load raw transactions from Hadoop Data Lake
df_transactions = spark.read.parquet("hdfs://namenode:8020/data/raw/transactions/date=2026-07-25/")

# Filter successful transactions
df_success = df_transactions.filter(col("status") == "SUCCESS")

# Calculate Commission (1.5% fee) and Net Payout
df_commission = df_success.withColumn("fawry_fee", round(col("amount_egp") * 0.015, 2)) \
                          .withColumn("merchant_payout", col("amount_egp") - col("fawry_fee"))

# Aggregate by Merchant ID
df_settlement = df_commission.groupBy("merchant_id") \
                             .agg(
                                 sum("amount_egp").alias("total_sales"),
                                 sum("fawry_fee").alias("total_fees"),
                                 sum("merchant_payout").alias("net_payout")
                             )

# Write results to database for next-day payouts
df_settlement.write \
    .format("jdbc") \
    .option("url", "jdbc:postgresql://oltp-db.fawry.local/finance") \
    .option("dbtable", "daily_settlements") \
    .save()

```

---

## ⏱️ 3. Pipeline Orchestration (Apache Airflow)

Airflow manages the dependencies and execution schedules of all batch jobs. It ensures that critical tasks—like calculating merchant settlements—never run before the raw data extraction from the data lake is 100% complete.

**Simulated Airflow DAG Configuration:**

```python
from airflow import DAG
from airflow.providers.apache.spark.operators.spark_submit import SparkSubmitOperator
from datetime import datetime, timedelta

default_args = {
    'owner': 'data_engineering_team',
    'depends_on_past': True,
    'start_date': datetime(2026, 7, 25),
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
}

with DAG('fawry_daily_etl_pipeline', default_args=default_args, schedule_interval='@daily') as dag:
    
    extract_to_hdfs = SparkSubmitOperator(
        task_id='extract_rdbms_to_hdfs',
        application='/opt/spark_jobs/extract_job.py'
    )
    
    calculate_settlements = SparkSubmitOperator(
        task_id='calculate_merchant_settlements',
        application='/opt/spark_jobs/merchant_settlement.py'
    )
    
    # Define task dependencies
    extract_to_hdfs >> calculate_settlements


