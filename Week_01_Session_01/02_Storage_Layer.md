# 🗄️ Module 2: Polyglot Storage Architecture

## 📌 Introduction & Simple Concept
In large-scale financial and enterprise systems, relying on a single database is a major architectural mistake. Different workloads require different database engines. **Polyglot Persistence** is the practice of using multiple database technologies within the same architecture, choosing the right database tool for the specific job.

For a platform like Fawry, we cannot use a standard relational database for high-speed, millions-of-requests user lookups, nor can we use a NoSQL database for strict financial ledgers where money transactions must be 100% accurate and permanent. This module explores how data is stored across different specialized tiers.

---

## 🏗️ The Three Core Storage Tiers

### 1. Operational Data Store (OLTP) — PostgreSQL
* **Purpose:** Acts as primary transactional ledger for processing active payments, user wallets, and merchant accounts.
* **Why PostgreSQL?** It guarantees strict **ACID compliance** (Atomicity, Consistency, Isolation, Durability) to ensure no money is ever lost or duplicated.

### 2. High-Availability NoSQL — Apache Cassandra
* **Purpose:** Manages user profiles, app session states, and high-velocity lookup tables.
* **Why Cassandra?** It uses a masterless architecture with zero single points of failure, ensuring the system stays online during massive traffic surges.

### 3. Analytical Data Lake (OLAP) — Hadoop (HDFS) & Hive
* **Purpose:** Archiving years of historical transactional data for long-term business analytics, auditing, and machine learning training.
* **Why Hadoop?** It provides a scalable, cost-effective distributed storage layer for large-scale historical data.

---

## 💻 Assignment Solution: Storage Architecture Summary

Below is the simple summary representing the assignment answers for this storage module:

* **Transactional Layer (PostgreSQL):** Used for accurate financial ledgers and account balances.
* **Fast Lookup Layer (Cassandra):** Used for user profiles and quick data retrieval without downtime.
* **Historical Data Layer (Hadoop):** Used for long-term data archiving and big data analytics.
* **Core Benefit:** Complete separation of concerns, ensuring heavy analytics never slow down live customer payments.
