# Production-ready-healthcare-analytics-platform-on-Snowflake

Built a production-ready healthcare analytics platform on Snowflake. The architecture follows the medallion pattern (bronze → silver → gold) and includes:
- Bronze layer – Raw CSV ingestion (patients, charges, doctors, item file) with minimal transformation, stored in staging tables.
- Silver layer – Cleaned, typed, and validated data (e.g., dates as DATE, amounts as DECIMAL, names standardized).
- Gold layer – A star schema with dim_patient, dim_doctor, dim_item, dim_date and a fact table fct_charges, plus business views (monthly revenue, patient 360, billing anomalies).

# Automation 
Streams on bronze tables to capture changes, and scheduled tasks that incrementally process data into silver and gold.

# Security & governance 
Role-based access control (finance_analyst, data_engineer), dynamic masking on patient names for non‑clinical roles.

# Performance & cost management
Clustering key on silver.charges_clean by charge_date, and a resource monitor on the ETL warehouse to limit monthly credit usage.

All of this was built manually using Snowflake worksheets and SQL commands.

┌─────────────────────────────────────────────────────────────────────────────┐
│                              DATA SOURCES                                   │
│  raw_patient.csv │ raw_charges.csv │ raw_doctors.csv │ raw_itemfile5.csv    │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  │ (Upload to stage)
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              BRONZE LAYER                                   │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌────────────────────┐  │
│  │    patients  │ │   charges    │ │   doctors    │ │     item_file      │  │
│  │  (raw data)  │ │  (raw data)  │ │  (raw data)  │ │    (raw data)      │  │
│  └──────┬───────┘ └──────┬───────┘ └──────┬───────┘ └─────────┬──────────┘  │
└─────────┼────────────────┼────────────────┼──────────────────┼─────────────┘
          │                │                │                  │
          │ (Streams capture changes)        │                  │
          ▼                ▼                ▼                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              SILVER LAYER                                   │
│  ┌────────────────┐ ┌────────────────┐ ┌────────────────┐ ┌──────────────┐  │
│  │patients_clean  │ │charges_clean   │ │doctors_clean   │ │item_file_clean│ │
│  │ (cleaned,typed)│ │(cleaned,typed) │ │(cleaned,typed) │ │(cleaned,typed)│ │
│  └───────┬────────┘ └───────┬────────┘ └───────┬────────┘ └──────┬───────┘  │
└──────────┼──────────────────┼──────────────────┼────────────────┼─────────┘
           │                  │                  │                │
           │ (Incremental tasks)                 │                │
           ▼                  ▼                  ▼                ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                               GOLD LAYER                                    │
│                              (Star Schema)                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │  dim_date    │  │ dim_patient  │  │  dim_doctor  │  │   dim_item   │     │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘     │
│         │                 │                 │                 │             │
│         └─────────────────┼─────────────────┼─────────────────┘             │
│                           ▼                 ▼                               │
│                    ┌─────────────────────────────────┐                      │
│                    │          fct_charges            │                      │
│                    │   (fact table with foreign keys)│                      │
│                    └─────────────────────────────────┘                      │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                         BUSINESS VIEWS                                │  │
│  │  • monthly_revenue (doctor, procedure, time)                          │  │
│  │  • patient_360 (patient summary)                                      │  │
│  │  • billing_anomalies (data quality / fraud detection)                 │  │
│  └──────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘

                                   │
                                   │ 
                                   ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           SECURITY & GOVERNANCE                             │
│  • Roles: FINANCE_ANALYST, DATA_ENGINEER                                    │
│  • Dynamic masking: patient names → 'REDACTED' for finance_analyst          │
│  • Row‑level security (future)                                              │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                         AUTOMATION & COST CONTROL                           │
│  • Streams on bronze tables (change data capture)                           │
│  • Tasks scheduled every 5/10 minutes (incremental processing)              │
│  • Resource monitor on ETL_WH (monthly credit limit, auto‑suspend)          │
│  • Clustering key on silver.charges_clean (charge_date) for performance     │
└─────────────────────────────────────────────────────────────────────────────┘
