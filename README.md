# GA4 BigQuery New vs. Returning Customer & Purchase Frequency Toolkit

A production-grade collection of BigQuery SQL models for categorizing e-commerce customer acquisition types and granular purchase frequency tiers using Google Analytics 4 (GA4) export data.

---

## 📌 Context & Data Maturity

This repository provides foundational SQL architectures for measuring customer loyalty, repurchase behavior, and campaign impact:
* **Cross-Device Stitching:** Integrates `COALESCE(user_id, user_pseudo_id)` to unify guest browsing cookies with authenticated account logins, preventing cross-device customer duplicate counts.
* **Optimized Single-Pass Ranking:** Leverages `DENSE_RANK()` within a single CTE to minimize compute costs and execution time.
* **Partition Pruning:** Enforces strict `_TABLE_SUFFIX` date filtering to limit BigQuery byte scans over large event tables.

---

## 📁 Repository File Layout

├── 1.0_new-v-returning-customers.sql
├── 1.1_frequency-of-purchase.sql
└── README.md

### 1. 1.0_new-v-returning-customers.sql
* **Approach:** Binary classification partitioning customer order histories chronologically.
* **Output Segments:** `new customer` (rank = 1) vs. `returning customer` (rank > 1).
* **Best Used For:** Evaluating channel acquisition effectiveness, comparing new vs. returning revenue splits over time, and measuring long-term customer retention.

### 2. 1.1_frequency-of-purchase.sql
* **Approach:** Granular purchase ordinal ranking using pre-calculated integer thresholds.
* **Output Segments:** `first purchase` (rank = 1), `second purchase` (rank = 2), `repeat purchase` (ranks 3–5), and `recurring customer` (ranks > 5).
* **Best Used For:** Understanding repurchase velocity, subscription/repeat buying models, and customer lifecycle progression.

---

## ⚡ Key Optimizations & Data Hygiene

Both queries in this repository follow production best practices:

1. **Single CTE Architecture:** Processes events through a single transformation CTE (`ranked_purchases`), eliminating redundant `GROUP BY` steps and nested CTE pipelines.
2. **Efficient Date Processing:** Derives `order_date` directly using `DATE(TIMESTAMP_MICROS(event_timestamp))` instead of string parsing.
3. **Data Quality Safeguard:** Explicitly filters `ecommerce.transaction_id IS NOT NULL` to exclude uncaptured or test conversion events.

---

## 📊 Summary Output Schema

Running either query generates a clean transaction-level output table:

| Column | Type | Description |
| :--- | :--- | :--- |
| `customer_id` | STRING | Stitched user identifier (`user_id` or `user_pseudo_id`). |
| `order_id` | STRING | Unique e-commerce transaction ID. |
| `revenue` | NUMERIC | Transaction revenue amount. |
| `order_date` | DATE | Date of transaction. |
| `source` | STRING | Traffic source associated with the acquisition/session. |
| `medium` | STRING | Traffic medium. |
| `campaign` | STRING | Campaign name. |
| `customer_type` | STRING | Assigned behavioral segment label. |

---

## 🛠 Prerequisites & Usage

1. Replace `enter.tablename_123456.events_*` with your GA4 BigQuery export table path.
2. Adjust the evaluation window interval in `_TABLE_SUFFIX` (default: 24 months) to fit your operational timeframe.
3. Run directly in BigQuery or schedule as a materialized view/table for Looker Studio or BI dashboarding.
