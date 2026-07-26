
# 🗄️ Module 2: Polyglot Storage Architecture (Fawry Case Study)

## 📌 Overview
In large-scale fintech systems like Fawry, no single database can handle everything. You cannot use a standard relational database for high-speed user lookups with millions of requests, nor can you use a NoSQL database for strict financial ledgers where money must never be lost or duplicated.

This document explores **Polyglot Persistence**—the strategy of using different database technologies for different workloads (OLTP, NoSQL, and OLAP).

---

## 🏗️ The Three Storage Tiers

### 1. Operational Data Store (OLTP) — PostgreSQL
* **Purpose:** Acts as the primary transactional ledger for processing active payments, user wallets, and merchant accounts.
* **Why PostgreSQL?** It guarantees **ACID compliance** (Atomicity, Consistency, Isolation, Durability). In financial systems, a transaction must either fully succeed or completely roll back. No partial states are allowed.

**Simulated PostgreSQL Schema (Core Ledger):**
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

### 2. High-Availability NoSQL — Apache Cassandra

* **Purpose:** Manages user profiles, session states, and high-velocity lookup tables.
* **Why Cassandra?** It features a masterless architecture with zero single points of failure. Even if a node crashes, the system remains fully online, ensuring high availability during massive traffic peaks.

**Simulated Cassandra Schema (User Profiles):**

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

### 3. Analytical Data Lake (OLAP) — Hadoop (HDFS) & Hive

* **Purpose:** Archiving over 10 years of historical transactional data for long-term analytics, auditing, and machine learning training.
* **Why Hadoop?** It provides a highly scalable, cost-effective distributed storage layer where data is stored in immutable formats (like Parquet) and partitioned by Year/Month/Day for rapid querying.

---

## ⚙️ Why Polyglot Persistence Matters for FinTech

* **Separation of Concerns:** Active money movement (PostgreSQL) is strictly isolated from historical data analysis (Hadoop).
* **Fault Tolerance:** If the analytical cluster experiences heavy batch loads, the core payment gateway remains completely unaffected.


