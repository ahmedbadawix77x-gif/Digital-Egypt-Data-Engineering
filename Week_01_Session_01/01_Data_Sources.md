# 📥 Module 1: Data Sources & Ingestion Layer (Fawry Case Study)

## 📌 Overview
In any robust data engineering pipeline, understanding where the data originates is the critical first step. Fawry operates as an omnichannel financial ecosystem, meaning data does not come from a single database, but from millions of distributed endpoints across Egypt.

This document breaks down the primary data sources, the velocity and volume of incoming data, and simulated payloads representing how transactions enter Fawry's infrastructure.

---

## 🌐 Omnichannel Ingestion Sources

### 1. Physical Point of Sale (POS) Terminals & ATMs
* **Scale:** Over 395,700 physical POS terminals and ATMs distributed across merchants, pharmacies, and convenience stores nationwide.
* **Data Nature:** Highly structured transactional payloads (ISO-8583 standards or JSON webhooks).
* **Generated Events:** Bill payments, mobile top-ups, cash-in/cash-out requests, and merchant settlements.

### 2. Digital Channels (myFawry Mobile App & Wallets)
* **Scale:** Over 24.2 million downloads of the *myFawry* application.
* **Data Nature:** High-frequency, semi-structured JSON data.
* **Generated Events:** Peer-to-peer (P2P) transfers, credit card top-ups, digital bill settlements, and telemetry clickstream logs (used for tracking user behavior and UI/UX optimization).

### 3. B2B & Open Banking APIs
* **Scale:** Direct gateway integrations with telecom operators (Vodafone, Orange, Etisalat, WE), utility providers (electricity, water, gas), and commercial banks.
* **Data Nature:** Secure, encrypted REST APIs and SOAP web services operating via API Gateways.

---
## ⚙️ Engineering Challenges at the Ingestion Point
* **High Concurrency:** Handling massive traffic spikes during peak hours (e.g., end-of-month utility billing deadlines).
* **Network Resilience:** POS terminals in remote areas may suffer from intermittent internet connectivity, requiring local queuing and retry mechanisms.
* **Security & Validation:** Every incoming payload must be instantly authenticated using merchant tokens and device signatures before hitting the core pipeline.
## 💻 Simulated Ingestion Payload (JSON)

When a customer pays a utility bill at a local merchant POS machine, a raw JSON event is transmitted to Fawry's edge ingestion servers:

```json
{
  "transaction_id": "TXN-88492011-A",
  "timestamp": "2026-07-26T14:32:01.000Z",
  "merchant_id": "MERCH-9948",
  "terminal_id": "POS-395700",
  "location": {
    "lat": 30.0444,
    "lon": 31.2357,
    "governorate": "Cairo"
  },
  "transaction_type": "UTILITY_BILL_PAYMENT",
  "biller_id": "ELEC-001",
  "customer_phone_masked": "+2010********",
  "amount_egp": 450.50,
  "currency": "EGP",
  "payment_method": "CASH",
  "status": "INITIATED"
}
