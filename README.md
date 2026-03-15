# NeoStats — Virtual Server Monitoring & Performance Optimization

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
