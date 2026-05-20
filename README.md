# Product Analytics Engineering Portfolio - Project 1

An end-to-end analytics engineering project built on the the [TheLook eCommerce](https://console.cloud.google.com/marketplace/product/bigquery-public-data/thelook-ecommerce) public dataset. Demonstrates a full production-grade data pipeline from raw source data to ML-powered churn scoring.

## Architechture
BigQuery Public Dataset (TheLook eCommerce)
↓
Staging Layer (dbt views): cleaned, typed, rename
↓
Marts Layer (dbt tables): business-ready facts & dimensions
↓
Analysis Notebooks (Python): EDA, funnel analysis, cohort retention
↓
ML Model (XGBoost): churn probability scoring
↓
ML Output(dbt table): predictions surfaced alongside user context

## Project Structure
```
ae_product_analytics/
├── models/
│   ├── staging/thelook/       # 5 staging models
│   └── marts/
│       ├── product/           # dim_users, fct_orders, fct_user_sessions
│       │                      # fct_funnel_events, fct_retention
│       └── ml_outputs/        # fct_churn_scores
├── analysis/
│   ├── 01_eda.ipynb
│   ├── 02_funnel_analysis.ipynb
│   └── 03_retention_cohorts.ipynb
├── ml/
│   ├── 01_churn_model.ipynb
│   └── models/                # saved model artifacts
└── tests/
```

## Data Models

| Model | Layer | Rows | Description |
|---|---|---|---|
| `stg_thelook__users` | Staging | 100k | Cleaned users |
| `stg_thelook__orders` | Staging | 125k | Cleaned orders |
| `stg_thelook__order_items` | Staging | — | Cleaned line items |
| `stg_thelook__events` | Staging | — | Cleaned clickstream |
| `stg_thelook__products` | Staging | — | Cleaned catalogue |
| `dim_users` | Marts | 100k | User attributes + order summary |
| `fct_orders` | Marts | 125k | Order facts + revenue metrics |
| `fct_user_sessions` | Marts | 181k | Session-level engagement |
| `fct_funnel_events` | Marts | 181k | Funnel progression per session |
| `fct_retention` | Marts | 1.6k | Cohort retention by month |
| `fct_churn_scores` | ML Outputs | 80k | Churn probability + risk tier |

## Testing

69 total tests across sources, staging, and marts.

```bash
dbt build #runs all models + tests in dependency order
```

| Result | Count |
|---|---|
| Pass | 67 |
| Warn | 2 (known duplicate emails in source data) |
| Error | 0 |

## Analysis Highlights

- **Funnel Analysis** - conversion drop-off by stage and traffic source, two-proportion z-test for statistical significance across segments,
- **Cohort Retention** - monthly cohort heatmap, retention curves, biggest drop-off indentification
- **Churn Model** - XGBoost classifier on 20+ behavioral features, predictions written back to BigQuery and wrapped in a tested dbt model

## Modeling Decisions

**Sessionization:** The events table contains a native `session_id` field with reliable session boundaries. After exploring custom window-function sessionization (30-minute gap logic), the native `session_id` was used as it correctly handles interleaved sessions from different devices/browsers that a gap-based approach would incorrectly merge.

**Churn definition:** A user is defined as churned if they have not placed an order in the 90 days prior to the most recent order data in the dataset. This is a standard RFM-based definition suitable for an apparel retailer with irregular purchase cadence.

**Email uniqueness:** The source data contains ~10,500 duplicate email addresses (users with multiple accounts). This is treated as a known data quality issue and surfaced as a warning rather than a hard test failure, as deduplication requires business logic outside the scope of this project.

## Setup

**Requirements:** Python 3.9+, dbt-core 1.11+, dbt-bigquery, Google Cloud SDK

```bash
# 1.authenticate
gcloud auth application-default login

#2.install dbt dependencies
dbt deps

#3.run the full pipeline
dbt build

#4. install python dependencies
pip install google-cloud-bigquery pandas matplotlib seaborn sgboost scikit-learn

#5. run notebooks in order
jupyter notebook
```

## Stack
- **Warehouse:** Google BigQuery
- **Transformation:** dbt Core 1.11
- **Languages:** SQL, Python
- **ML:** XGBoost, scikit-learn
- **Visualization:** matplotlib, seaborn