# ⚙️ Module 3: Processing Layer & ETL Pipelines

## 📌 Introduction & Simple Concept
Once data is collected and stored, it needs to be moved around and processed. In a financial system, this happens in two main ways: **Real-time Processing** (handling things instantly as they happen, like fraud detection) and **Batch Processing** (handling heavy workloads at the end of the day, like calculating merchant salaries and settlements).

This module explains the core components of the processing layer in simple terms.

---

## ⚡ 1. The Speed Layer (Real-time Streaming)
* **What is it?** A system designed to handle live incoming events instantly (with millisecond latency).
* **Why use it?** It ensures that if a fraudulent transaction happens, the system can catch it and block it before it goes through.

## 🔄 2. The Batch Layer (Heavy Processing)
* **What is it?** A processing engine that runs massive data tasks all at once, usually at night or at the end of the business day.
* **Why use it?** It calculates commissions, daily sales totals, and payouts for over 395,700 merchants across Egypt.

## ⏱️ 3. Pipeline Orchestration (Scheduling)
* **What is it?** The manager or traffic controller of our data pipelines.
* **Why use it?** It makes sure that tasks run in the correct order automatically (for example: don't calculate settlements until all raw data is safely extracted).

---

## 💻 Assignment Solution: Processing Pipeline Summary

Below is the simple summary representing the assignment answers for this module:

* **Real-time Track (Streaming):** Handles instant transaction events for immediate security and status updates.
* **Batch Track (EOD Jobs):** Handles end-of-day merchant financial summaries, commission calculations, and database updates.
* **Orchestration Task:** Manages workflow dependencies, schedules, and automated retries if a task fails.
