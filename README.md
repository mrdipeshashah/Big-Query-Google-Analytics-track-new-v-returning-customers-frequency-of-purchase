# OVERVIEW

A production-grade collection of BigQuery SQL models for categorizing e-commerce customer acquisition types and granular purchase frequency tiers using Google Analytics 4 (GA4) export data.

## REPOSITORY

### 1. 1.0_new-v-returning-customers.sql
* **Approach:** Binary classification partitioning customer order histories chronologically.
* **Output Segments:** `new customer` (rank = 1) vs. `returning customer` (rank > 1).
* **Best Used For:** Evaluating channel acquisition effectiveness, comparing new vs. returning revenue splits over time, and measuring long-term customer retention.

### 2. 1.1_frequency-of-purchase.sql
* **Approach:** Granular purchase ordinal ranking using pre-calculated integer thresholds.
* **Output Segments:** `first purchase` (rank = 1), `second purchase` (rank = 2), `repeat purchase` (ranks 3–5), and `recurring customer` (ranks > 5).
* **Best Used For:** Understanding repurchase velocity, subscription/repeat buying models, and customer lifecycle progression.

## SQL OPTIMISATION & PERFORMANCE 

Both queries in this repository follow production best practices:

1. **Single CTE Architecture:** Processes events through a single transformation CTE (`ranked_purchases`), eliminating redundant `GROUP BY` steps and nested CTE pipelines.
2. **Efficient Date Processing:** Derives `order_date` directly using `DATE(TIMESTAMP_MICROS(event_timestamp))` instead of string parsing.
3. **Data Quality Safeguard:** Explicitly filters `ecommerce.transaction_id IS NOT NULL` to exclude uncaptured or test conversion events.

## SUMMARY OUTPUT SCHEME

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


