# End-to-End Data Pipeline with ML & SQL Analytics

Sara | MSc Data Science | B.Tech Artificial Intelligence and Machine Learning

---

## Project Overview

This project builds a complete data engineering and machine learning pipeline for financial transaction data. It generates 50,000 synthetic transactions using the Faker library, ingests them into PySpark, runs a 7-stage ETL pipeline with data quality validation, engineers 21 features including window aggregates, trains an XGBoost risk-scoring model, runs SQL analytics queries with pandasql, and exports the enriched dataset for visualisation in Google Looker Studio.

---

## Pipeline Performance

| Metric | Value |
|---|---|
| Total Transactions | 50,000 |
| Fraud Rate | 3.0% (realistic class imbalance) |
| ETL Stages | 7 |
| DQ Checks Passed | 5 / 5 |
| Engineered Features | 21 |
| XGBoost ROC-AUC | 0.947 |
| XGBoost Avg Precision | 0.812 |
| SQL Business Queries | 4 |

---

## Technical Stack

| Category | Tools |
|---|---|
| Data Generation | Faker |
| Distributed Processing | PySpark 3.x |
| ETL Orchestration | PySpark DataFrame API |
| ML Model | XGBoost |
| SQL Analytics | pandasql (BigQuery-compatible syntax) |
| Visualisation | Plotly |
| Dashboard | Google Looker Studio |
| Language | Python 3.10 |

---

## Pipeline Architecture

**Stage 1 — Generate:** Faker creates 50,000 transactions with realistic distributions across 10 categories, 4 account types, 10 countries, and 4 channels. A 3% fraud rate with injected data quality issues (2% null rate, 1.5% refund rate) mimics production data.

**Stage 2 — Ingest:** PySpark reads the CSV with an explicit typed schema into a distributed DataFrame, simulating ingestion from a cloud data lake or HDFS landing zone.

**Stage 3 — Clean:** Null amounts are imputed to the category median. Negative amounts are flagged as refunds and converted to absolute values. Unknown categories are standardised to 'other'. Null merchant names are replaced with 'UNKNOWN_MERCHANT'.

**Stage 4 — Validate:** Five automated data quality checks run against the cleaned DataFrame — null checks on key fields, amount positivity, duplicate transaction ID detection, and valid category enumeration. All results are logged with pass/fail status.

**Stage 5 — Engineer:** PySpark window functions compute customer-level aggregates (total transaction count, rolling average amount, max amount). Time features are extracted from the timestamp (hour, day of week, month, is_weekend, is_night). Risk flags are derived (is_high_risk_country, is_international, amount_vs_avg ratio).

**Stage 6 — Score:** XGBoost is trained on 21 features with class weighting to handle the 3% fraud imbalance. The model assigns a continuous risk score between 0 and 1 to every transaction. Transactions are bucketed into Low (< 0.30), Medium (0.30–0.60), and High (> 0.60) risk tiers.

**Stage 7 — Export:** The enriched DataFrame is written to CSV for Looker Studio. Summary SQL tables are also exported for business reporting.

---

## SQL Analytics Queries

Four business-facing queries are run with pandasql using BigQuery-compatible syntax:

- Fraud rate and average transaction value by category
- High-risk transaction volume by account type
- Monthly transaction volume and fraud trend across 24 months
- Top 10 riskiest customers by average risk score

---

## Key Technical Decisions

### Why PySpark for 50,000 records

In production, financial transaction pipelines process millions of records per day. Writing the pipeline in PySpark from the start ensures it scales horizontally without code changes — the same logic that runs locally in Colab can be deployed to AWS EMR, Databricks, or Google Dataproc with no modifications.

### Why XGBoost with scale_pos_weight

With only 3% fraud, a naive model would predict all transactions as legitimate and still achieve 97% accuracy. scale_pos_weight = 32 tells XGBoost to penalise missing fraud cases 32 times more than missing legitimate ones, shifting the decision boundary toward higher recall on the minority class. The result is evaluated with ROC-AUC and Average Precision, both of which are insensitive to class imbalance.

### Why pandasql for analytics

pandasql lets analysts write standard SQL that runs on Pandas DataFrames in memory, making it trivial to migrate the same queries to BigQuery, Snowflake, or Redshift in production. The queries are written to be fully BigQuery-compatible.

### Why Looker Studio for the dashboard

Looker Studio (formerly Google Data Studio) provides a permanently hosted, publicly shareable dashboard URL with no infrastructure cost. Uploading the enriched CSV takes under 2 minutes and supports interactive filters, time series charts, and geographic maps out of the box.

---

## Repository Structure

```
etl-pipeline-risk-scoring/
    ETL_Pipeline_Risk_Scoring.ipynb    Full notebook, runs end to end
    README.md
    outputs/
        transactions_enriched.csv      Enriched data for Looker Studio
        summary_by_category.csv
        summary_monthly.csv
        risk_model.pkl                 Saved XGBoost model
        feature_cols.pkl
```

---

## How to Run

Open the Colab notebook and run all cells in order. No GPU is needed — this project runs on CPU. Runtime is approximately 4–6 minutes for the full pipeline including data generation, ETL, model training, and SQL queries.

To deploy the Looker Studio dashboard:

1. Run the notebook and download `transactions_enriched.csv`
2. Go to lookerstudio.google.com
3. Create Report → Add Data → File Upload
4. Upload `transactions_enriched.csv`
5. Build charts using `risk_score`, `risk_label`, `category`, `amount`, `is_fraud`
6. Click Share → Anyone with link → Copy public URL

---

## Links

- Looker Studio Dashboard: https://lookerstudio.google.com/reporting/b7d766bb-2a9b-47dd-a535-28b2a1906ee5
- Google Colab Notebook: https://drive.google.com/file/d/1gRwLGuADXv-AZBn4rJOF9KkQRWplM3pN/view?usp=sharing
- Dashboard of Project: 
---

## About

Built by Sara. MSc Data Science. B.Tech in Artificial Intelligence and Machine Learning. Based in London.

This project demonstrates production-grade data engineering skills covering PySpark ETL, automated data quality validation, ML-based risk scoring, SQL business analytics, and cloud dashboard deployment.
