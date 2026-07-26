
# 📊 Module 4: Analytics & Serving Layer (Fawry Case Study)

## 📌 Overview
The final layer of our data architecture is the **Serving Layer**. After ingesting, storing, and processing millions of transactions, this layer is where business value is actually delivered. It powers AI-driven credit scoring for micro-lending, real-time fraud detection analytics, and executive business intelligence (BI) dashboards.

This document breaks down how Fawry translates raw engineered data into actionable financial products.

---

## 🤖 1. Machine Learning: Micro-Lending & Credit Scoring
Fawry manages a massive micro-lending portfolio (over EGP 5.7 Billion). To evaluate whether a customer or merchant is eligible for a digital loan with zero collateral, data scientists deploy machine learning models trained on historical payment consistency.

* **Target Variable:** Probability of Default (PD).
* **Feature Engineering via Spark:**
  * Average monthly utility bill payments.
  * Frequency of wallet reloads vs. cash-outs.
  * Account age and historical transaction success rate.

**Simulated Python Inference Snippet (XGBoost Credit Scoring):**
```python
import xgboost as xgb
import pandas as pd

# Load pre-processed features for a merchant/user
customer_features = pd.DataFrame({
    'avg_monthly_volume_egp': [15400.0],
    'transaction_success_rate': [0.98],
    'account_age_months': [24],
    'utility_payment_consistency': [1.0]
})

# Load trained credit scoring model
model = xgb.Booster()
model.load_model("fawry_credit_scoring_model.json")

# Convert to DMatrix and predict Risk Score (0 to 1)
dmatrix = xgb.DMatrix(customer_features)
default_probability = model.predict(dmatrix)[0]

print(f"Calculated Default Probability: {default_probability:.4f}")
if default_probability < 0.15:
    print("Loan Status: APPROVED (Low Risk)")
else:
    print("Loan Status: REJECTED / MANUAL REVIEW")

```

---

## 📈 2. Business Intelligence & Executive Dashboards

Executives, risk managers, and operations teams cannot query Hadoop or PostgreSQL directly using code every day. They rely on executive BI dashboards (such as Apache Superset, Tableau, or Power BI) connected to fast query engines like Presto or Hive.

* **Key Metrics Monitored Real-Time:**
* **Liquidity & Cash Flow:** Total EGP volume moving across regions per hour.
* **System Health:** Success vs. Failure rates of POS terminals across Egyptian governorates (Cairo, Giza, Alexandria, etc.).
* **Fraud Trends:** Sudden spikes in disputed or blocked transactions.



---

## 🏁 Summary of the End-to-End Pipeline

1. **Ingestion Layer:** Captures over 6 million daily transactions from 395,700+ POS terminals and the myFawry app.
2. **Storage Layer:** Balances strict ACID compliance in PostgreSQL with high-availability Cassandra and a 10-year Hadoop Data Lake.
3. **Processing Layer:** Streams data instantly via Kafka for fraud detection while Spark handles heavy batch settlements orchestrated by Airflow.
4. **Serving Layer:** Powers AI credit scoring for micro-lending and real-time BI dashboards.



```
