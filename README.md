# campaign_vault

A GDPR-aware marketing campaign data pipeline built with Python, Apache Spark, and Delta Lake on Databricks, using the Medallion architecture.

---

## Project Overview

CampaignVault simulates a real-world data engineering pipeline for a fictional e-commerce company's marketing campaigns. The project demonstrates:

- **Medallion Architecture** (Bronze → Silver → Gold)
- **GDPR-compliant PII isolation** using separate schemas
- **Delta Lake** features: ACID transactions, time travel, schema enforcement
- **End-to-end pipeline** from synthetic data generation to business-ready aggregations

---

## Architecture

```
customers_raw.csv  ──┐
                     ├──► Bronze (raw Delta tables)
campaigns_raw.csv  ──┘         │
                                ▼
                    Silver (cleaned, PII separated)
                    ├── silver.customers         (no PII)
                    ├── silver.campaigns         (no PII)
                    └── silver_pii.customer_identifiers  (PII isolated)
                                │
                                ▼
                    Gold (business-ready aggregations)
                    ├── gold.campaign_performance
                    ├── gold.channel_roi
                    └── gold.city_segments
```

---

## GDPR & PII Design

Personal data (name, email, phone) is isolated in a dedicated `silver_pii` schema, separate from all analytical tables. This means:

- Analysts can access `silver` and `gold` schemas without ever seeing personal data
- A "right to erasure" request can be fulfilled by deleting a single row from `silver_pii.customer_identifiers` — without touching any analytical tables
- Delta Lake time travel provides a full audit trail of all data changes
- In a production environment, Unity Catalog row-level and column-level security would complement this design

| Schema | Contains | Access |
|---|---|---|
| `bronze` | Raw ingested data | Data Engineers |
| `silver` | Cleaned, PII-free data | Analysts, Data Scientists |
| `silver_pii` | Personal identifiers | Restricted — need-to-know only |
| `gold` | Aggregated business metrics | All stakeholders |

---

## Project Structure

```
CampaignVault/
├── data/
│   ├── customers.csv       # Synthetic customer data (1000 rows)
│   └── campaigns.csv       # Synthetic campaign events (1000 rows)
├── notebooks/
│   ├── 00_setup.ipynb          # Schema creation
│   ├── 01_bronze.ipynb         # Raw ingestion to Delta tables
│   ├── 02_silver.ipynb         # Cleaning, transformation, PII separation
│   └── 03_gold.ipynb           # Business aggregations
└── src/
    └── generate_data.ipynb        # Synthetic data generation (Faker)
```

---

## Tech Stack

| Tool | Purpose |
|---|---|
| Python + Faker | Synthetic data generation |
| Apache Spark (PySpark) | Data transformation |
| Delta Lake | ACID storage, time travel |
| Databricks (Serverless) | Compute and orchestration |

---

## How to Run

### 1. Generate synthetic data

```bash
pip install faker unidecode pandas
python src/generate_data.ipynb
```

This produces `customers.csv` and `campaigns.csv` in the `data/` folder.

### 2. Upload to Databricks

Upload both CSV files to a Databricks Volume:
`/Volumes/<catalog>/<schema>/<volume_name>/`

### 3. Run notebooks in order

Open each notebook in Databricks and run all cells in sequence:

```
00_setup → 01_bronze → 02_silver → 03_gold
```

---

## Key Delta Lake Features Demonstrated

- **Schema enforcement** — Delta rejects writes that don't match the table schema
- **ACID transactions** — all writes are atomic; no partial data
- **Time travel** — query any previous version with `VERSION AS OF`
- **Audit history** — `DESCRIBE HISTORY <table>` shows all operations

---

## Data Model

### customers_raw (Bronze)
| Column | Type | Notes |
|---|---|---|
| customer_id | string | UUID, primary key |
| first_name | string | PII |
| last_name | string | PII |
| email | string | PII |
| phone_number | string | PII |
| user_name | string | |
| age | integer | |
| city | string | |
| country | string | |
| latitude | double | |
| longitude | double | |
| postal_code | string | |
| timezone | string | |
| ingested_at | timestamp | Technical metadata |
| source_file | string | Technical metadata |

### campaigns_raw (Bronze)
| Column | Type | Notes |
|---|---|---|
| event_id | string | Primary key |
| customer_id | string | Foreign key → customers |
| campaign_name | string | |
| channel | string | email / SMS / social / push |
| currency_code | string | |
| revenue | double | |
| clicked | boolean | |
| converted | boolean | |
| sent_at | timestamp | |
| opened_at | timestamp | |
| this_year | integer | |
| unix_time | double | |
| company_name | string | |
| business_details | string | |
| catch_phrase | string | |
| fake_sentence | string | |
| product_id | string | |
| ingested_at | timestamp | Technical metadata |
| source_file | string | Technical metadata |
