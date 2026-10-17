# 💳 Financial Fraud Detection & Risk Analysis

> An exploratory data analytics and rule-based risk-scoring pipeline designed to evaluate transaction telemetry, flag fraudulent anomalies, and minimize false positives.

[![Python 3.10](https://img.shields.io/badge/python-3.10-blue.svg)](https://www.python.org/)
[![SQL / DuckDB](https://img.shields.io/badge/SQL-DuckDB-orange.svg)](https://duckdb.org/)
[![Pandas](https://img.shields.io/badge/Pandas-Data%20Wrangling-brightgreen.svg)](https://pandas.pydata.org/)

---

## 📊 Executive Summary & Business Objective
Financial platforms lose revenue to fraudulent credit card transactions, but blocking transactions too aggressively creates false-positive blocks that frustrate legitimate customers.
* **The Goal:** Analyze historical transaction telemetry using statistical anomaly detection (Z-scores) and multi-factor rules to categorize transaction risk tiers.
* **The Impact:** Equips risk management teams with clear analytical guardrails to isolate high-risk international and anomalous purchases without interrupting standard user experiences.

---

## 🏗️ Technical Architecture & Workflow
1. **Data Ingestion & Wrangling:** Loaded raw account payment telemetry into Python via Pandas.
2. **Statistical Feature Engineering:** Computed distribution metrics and statistical Z-scores (`amount_zscore`) to identify transaction volume outliers.
3. **Rule-Based Risk Classification:** Executed multi-factor SQL queries via `DuckDB` to segment transactions into `High Risk`, `Medium Risk`, and `Low Risk` tiers.

---

## 💻 Code Snippet: Risk-Scoring & Aggregation Query
```sql
WITH RiskEvaluated AS (
    SELECT 
        transaction_id,
        account_id,
        merchant_category,
        amount,
        is_international,
        ROUND(amount_zscore, 2) AS amount_zscore,
        -- Rule: Flag as High Risk if Z-score > 1.0 AND transaction is international
        CASE 
            WHEN amount_zscore > 1.0 AND is_international = 1 THEN 'High Risk'
            WHEN amount_zscore > 0.5 OR is_international = 1 THEN 'Medium Risk'
            ELSE 'Low Risk'
        END AS risk_tier
    FROM df_txns
)
SELECT 
    merchant_category,
    COUNT(transaction_id) AS total_txns,
    SUM(CASE WHEN risk_tier = 'High Risk' THEN 1 ELSE 0 END) AS high_risk_count,
    ROUND(AVG(amount), 2) AS avg_txn_amount
FROM RiskEvaluated
GROUP BY merchant_category
ORDER BY high_risk_count DESC;
financial-fraud-risk-analytics/
├── financial_fraud_analysis.ipynb   # Main analysis notebook
└── README.md                        # Project documentation
