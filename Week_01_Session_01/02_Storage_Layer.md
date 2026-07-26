
# 🗄️ Module 2: Polyglot Storage Architecture

## 📌 Introduction & Simple Concept
In large-scale financial and enterprise systems, relying on a single database is a major architectural mistake. Different workloads require different database engines. **Polyglot Persistence** is the practice of using multiple database technologies within the same architecture, choosing the right database tool for the specific job.

For a platform like Fawry, we cannot use a standard relational database for high-speed, millions-of-requests user lookups, nor can we use a NoSQL database for strict financial ledgers where money transactions must be 100% accurate and permanent. This module explores how data is stored across different specialized tiers.

---

## 🏗️ The Three Core Storage Tiers

### 1. Operational Data Store (OLTP) — PostgreSQL
* **Purpose:** Acts as the primary transactional ledger for processing active payments, user wallets, and merchant accounts.
* **Why PostgreSQL?** It guarantees strict **ACID compliance** (Atomicity, Consistency, Isolation, Durability). In financial systems, a transaction must either fully succeed or completely roll back, ensuring no money is ever lost or duplicated.

### 2. High-Availability NoSQL — Apache Cassandra
* **Purpose:** Manages user profiles, app session states, and high-velocity lookup tables.
* **Why Cassandra?** It uses a masterless architecture with zero single points of failure. Even if a node goes down, the system remains completely online during massive traffic surges.

### 3. Analytical Data Lake (OLAP) — Hadoop (HDFS) & Hive
* **Purpose:** Archiving years of historical transactional data for long-term business analytics, auditing, and machine learning training.
* **Why Hadoop?** It provides a scalable, cost-effective distributed storage layer where data is stored in immutable formats like Apache Parquet and partitioned efficiently.

---

## 💻 Assignment Solution: Database Schemas & Code

Below are the assignment templates representing the database schemas used across the storage layers:

### A. PostgreSQL Schema (Core Financial Ledger)
```sql
CREATE TABLE fawry_accounts (
    account_id UUID PRIMARY KEY,
    user_id UUID NOT NULL,
    account_type VARCHAR(50) NOT NULL,
    balance_egp DECIMAL(15, 2) DEFAULT 0.00,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE fawry_ledger_entries (
    entry_id UUID PRIMARY KEY,
    transaction_id VARCHAR(100) NOT NULL,
    from_account UUID REFERENCES fawry_accounts(account_id),
    to_account UUID REFERENCES fawry_accounts(account_id),
    amount_egp DECIMAL(15, 2) NOT NULL,
    status VARCHAR(20) DEFAULT 'PENDING',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

```

### B. Cassandra Schema (User Profiles)

```cql
CREATE KEYSPACE fawry_distributed WITH replication = {
    'class': 'NetworkTopologyStrategy', 
    'datacenter1': 3
};

CREATE TABLE fawry_distributed.user_profiles (
    user_id uuid,
    phone_text text,
    kyc_verified boolean,
    loyalty_tier int,
    last_active_timestamp timestamp,
    PRIMARY KEY (user_id)
);

```

---

## ⚙️ Why Polyglot Persistence Matters

* **Separation of Concerns:** Active money transactions (PostgreSQL) are completely isolated from historical big data analysis (Hadoop).
* **Fault Tolerance:** Heavy analytical queries will never slow down or crash the core payment processing gateway.
