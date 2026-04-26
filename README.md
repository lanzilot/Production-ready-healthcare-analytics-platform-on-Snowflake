# Production-ready-healthcare-analytics-platform-on-Snowflake

Built a production-ready healthcare analytics platform on Snowflake. The architecture follows the medallion pattern (bronze → silver → gold) and includes:
## Bronze layer – Raw CSV ingestion (patients, charges, doctors, item file) with minimal transformation, stored in staging tables.
## Silver layer – Cleaned, typed, and validated data (e.g., dates as DATE, amounts as DECIMAL, names standardized).
## Gold layer – A star schema with dim_patient, dim_doctor, dim_item, dim_date and a fact table fct_charges, plus business views (monthly revenue, patient 360, billing anomalies).

# Automation 
Streams on bronze tables to capture changes, and scheduled tasks that incrementally process data into silver and gold.

# Security & governance 
Role-based access control (finance_analyst, data_engineer), dynamic masking on patient names for non‑clinical roles.

# Performance & cost management
Clustering key on silver.charges_clean by charge_date, and a resource monitor on the ETL warehouse to limit monthly credit usage.

All of this was built manually using Snowflake worksheets and SQL commands.
