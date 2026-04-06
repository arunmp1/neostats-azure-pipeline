# Virtual Server Monitoring & Performance Optimization

A production-grade data engineering pipeline built on Microsoft Azure for XYZ Corp, 
implementing automated ingestion, transformation, and visualization of virtual server 
performance logs across a global fleet of 21 servers in 5 locations.

---

## Project Overview

This pipeline ingests raw server performance logs from GitHub, applies a comprehensive 
data quality and transformation framework, and delivers enriched analytical datasets 
to a multi-page interactive Power BI dashboard. The solution is fully automated via 
Azure Data Factory and runs on a daily schedule with zero manual intervention.

---

## Architecture

GitHub (CSV) → ADF Schedule Trigger → File Validation → Copy Activity → Bronze Layer (ADLS Gen2)
→ Mapping Data Flow (Cleaning) → Silver Layer (ADLS Gen2)
→ Mapping Data Flow (Aggregation) → Gold Layer (ADLS Gen2) + Azure SQL Database
→ Power BI Dashboard

The pipeline implements the Medallion Architecture across three layers:

- Bronze — Raw ingested data, exact copy of source, never modified
- Silver — Cleaned, typed, enriched, PII-masked Parquet dataset (20 columns, 200 rows)
- Gold — Five purpose-built aggregated tables for dashboard consumption

---

## Technology Stack

- Azure Data Factory — Pipeline orchestration and scheduling
- Azure Data Lake Storage Gen2 — Bronze, Silver, Gold layer storage
- ADF Mapping Data Flows — Spark-based data transformation engine
- Azure SQL Database — Structured analytical layer for Power BI
- Power BI Desktop — Multi-page interactive dashboard
- GitHub — Source data versioning and pipeline code repository
- Apache Parquet (Snappy) — Processed data format
- T-SQL — Table creation and validation queries
- DAX — Power BI KPI measure calculations

---

## Repository Structure
```
azure-pipeline/
│
├── README.md
│
├── data/
│   └── Server_Performance_Logs.csv
│
├── adf-pipeline/
│   ├── pipeline/
│   │   └── PL_Ingestion.json
│   ├── dataflow/
│   │   ├── bronze_to_silver_cleaning.json
│   │   └── silver_to_gold_analytics.json
│   ├── linkedService/
│   │   ├── LS_GitHub_HTTP.json
│   │   ├── LS_ADLS.json
│   │   └── LS_AzureSQL.json
│   ├── dataset/
│   │   ├── DS_Bronze_CSV.json
│   │   ├── DS_Silver_Parquet.json
│   │   └── DS_Gold_Parquet.json
│   └── trigger/
│       └── TR_Daily_Refresh.json
│
├── sql/
│   ├── create_tables.sql
│   └── validation_queries.sql
│
├── powerbi/
│   └──Dashboard.pbix
│
├── docs/
│   ├── Report.docx
│   └── architecture_diagram.png
│
└── screenshots/
    ├── pipeline_overview.png
    ├── bronze_to_silver_dataflow.png
    ├── silver_to_gold_dataflow.png
    ├── sql_tables.png
    └── dashboard_page1.png
```

---

## Pipeline Details

### Data Ingestion
- Source: GitHub raw CSV via HTTP Linked Service (anonymous authentication)
- File validation: If Condition activity checks filename starts with 'Server'
- Copy Activity writes exact CSV to Bronze container in ADLS Gen2
- Schedule Trigger fires daily at 06:00 AM automatically

### Bronze to Silver Transformations
- Type cast 7 numeric columns from String to Double
- Parse Log_Timestamp using custom format dd-MM-yyyy HH:mm
- Remove records with null critical fields or physically impossible values
- Impute missing non-critical fields with safe defaults
- Decode Server_Role and Instance_Size from hostname patterns
- Split Server_Location into City and Country
- Apply MD5 PII masking to IP Address, Admin Email, Admin Phone, Admin Name
- Calculate Anomaly_Flag — CRITICAL, WARNING, NORMAL
- Calculate Availability_Pct, SLA_Breach, Health_Score, Total_Network
- Deduplicate by Server_ID and Log_Timestamp using first() aggregation
- Output: 20-column clean Parquet file to Silver container

### Silver to Gold Transformations
- 5 parallel aggregation streams in single Data Flow execution
- Each stream writes simultaneously to Gold ADLS Parquet and Azure SQL table
- Aggregations include avg metrics, SLA breach counts, critical/warning counts, server counts
- Write mode: Truncate and reload on every run

---

## Azure SQL Tables

| Table | Rows | Description |
|---|---|---|
| server_logs_full | 200 | Complete analytical fact table |
| agg_by_server | 21 | Per-server KPI averages |
| agg_by_location | 5 | Regional performance summary |
| agg_by_os | 5 | OS-level performance summary |
| agg_by_role | 7 | Role-level performance summary |

---

## Key Findings

- Fleet average health score: 50.91 / 100
- Average availability: 85.88% against 95% SLA target
- SLA breach rate: 94.5% (189 out of 200 log entries)
- Critical alerts: 35 (17.5% of all records)
- Warning alerts: 97 (48.5% of all records)
- Best performing city: Sydney
- Worst performing city: New York
- Most breached server role: Analytics servers

---

## Power BI Dashboard

The dashboard connects to Azure SQL Database via Import mode and consists of two pages:

Page 1 — Fleet Overview
- 5 fixed KPI cards using DAX measures
- Alert severity donut chart
- Health score by city with conditional green/red formatting
- SLA breaches by server role
- OS distribution donut
- Interactive tile slicers for Server Role and City

Page 2 — Server Performance Deep Dive
- 3 performance KPI cards — Avg CPU, Memory, Disk IO
- Bookmark switcher buttons for CPU, Memory, Disk IO chart toggle
- Resource utilization clustered column chart by city
- CPU vs Memory anomaly scatter plot
- CPU by OS type donut
- Performance trend line chart over time

---

## How to Deploy

### Prerequisites
- Azure subscription with ADF, ADLS Gen2, and Azure SQL provisioned
- Power BI Desktop installed

### Steps
1. Clone this repository
2. Create Bronze, Silver, Gold containers in your ADLS Gen2 storage account
3. Import ADF pipeline JSON files via ADF Studio — Manage → Import
4. Update Linked Service credentials — ADLS account key and SQL connection string
5. Run SQL scripts in sql/create_tables.sql against your Azure SQL Database
6. Trigger the pipeline manually once to validate end-to-end execution
7. Verify row counts using sql/validation_queries.sql
8. Open powerbi/Dashboard.pbix and update SQL connection string
9. Publish dashboard to Power BI Service if required

---

## Notes

- Linked Service JSON files have credentials removed for security
- Replace placeholder values in Linked Services with your own Azure credentials
- Pipeline is designed for daily batch processing — not real-time streaming
- All PII data is masked in Silver and removed entirely before Gold layer


