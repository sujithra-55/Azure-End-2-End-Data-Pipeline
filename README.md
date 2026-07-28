# Azure End-to-End Data Engineering Project (Bronze → Silver → Gold)

## 📌 Project Overview

This project implements a full medallion-architecture data pipeline on Azure — ingesting raw data from multiple sources, cleaning and transforming it, and serving curated data for BI reporting.

**Flow:** SQL Database + REST API → ADF (Bronze) → Databricks/PySpark (Silver) → Synapse Analytics (Gold) → Power BI

---

## 🏗️ Architecture

```
 ┌─────────────┐      ┌─────────────┐      ┌────────────────┐      ┌───────────────┐      ┌───────────┐
 │ SQL Database│      │  REST API   │      │ Azure Data     │      │ Azure Databricks│    │  Synapse   │
 │             ├─────►│             ├─────►│ Factory (ADF)  ├─────►│ (PySpark)      ├───► │ Analytics  │
 └─────────────┘      └─────────────┘      │ Dynamic Pipelines│    │ Transformations │    │ Views +    │
                                            └────────┬────────┘    └────────┬────────┘    │ External   │
                                                     │                      │              │ Tables     │
                                                     ▼                      ▼              └─────┬─────┘
                                          ┌─────────────────┐   ┌─────────────────┐               │
                                          │  Data Lake       │   │  Data Lake       │               ▼
                                          │  Bronze Layer    │   │  Silver Layer    │        ┌───────────┐
                                          │  (Raw)           │   │  (Cleaned)       │        │  Power BI  │
                                          └─────────────────┘   └─────────────────┘        │ Dashboards │
                                                                                             └───────────┘
```

---

## 🔹 Step 1: Infrastructure Setup

- Created an Azure **Resource Group** to logically group all project resources.
- Provisioned an **Azure Data Lake Storage Gen2** account with three containers: `bronze`, `silver`, `gold`.
- Provisioned an **Azure Data Factory (ADF)** instance for orchestration.

## 🔹 Step 2: Bronze Layer — Raw Ingestion (ADF)

- Built **dynamic, parameterized pipelines** in ADF (parameterized linked services + datasets, driven by a control/config table or `ForEach` activity) so the same pipeline ingests multiple tables/endpoints without duplicating logic.
- **Source 1 — SQL Database:** Copy Activity pulls tables into the Bronze container, partitioned by ingestion date.
- **Source 2 — REST API:** Copy Activity (HTTP linked service) pulls JSON responses into Bronze, handling pagination/parameters dynamically.
- Output lands in `bronze/` in its raw, unprocessed form — schema-on-read, no transformations yet.

## 🔹 Step 3: Silver Layer — Cleaning & Transformation (Databricks + PySpark)

- Azure Databricks reads all raw files from the Bronze container.
- Using **PySpark**, applied transformations such as:
  - Null/duplicate handling
  - Schema enforcement and type casting
  - Standardizing column names/formats
  - Joining/consolidating source datasets where needed
- Wrote the cleaned output (ideally as **Delta format** for versioning/ACID) into the `silver/` container.

## 🔹 Step 4: Gold Layer — Curation & Serving (Synapse Analytics)

- Connected Synapse Analytics (Serverless SQL Pool) to the Silver layer.
- Created **views** on top of Silver data for business-friendly, query-ready datasets.
- Built **external tables** over those views/curated data — this is effectively the **Gold layer**: aggregated, reporting-ready data.

## 🔹 Step 5: Visualization (Power BI)

- Connected Power BI directly to the Synapse external tables (native Synapse connector).
- Built interactive dashboards/visualizations on top of the Gold layer data.

---

## 🛠️ Tech Stack

| Layer | Tool |
|---|---|
| Orchestration | Azure Data Factory |
| Storage | Azure Data Lake Storage Gen2 |
| Processing | Azure Databricks (PySpark) |
| Serving/Warehouse | Azure Synapse Analytics |
| Visualization | Power BI |

## 📈 Key Learnings

- Designing dynamic/parameterized pipelines to avoid one-pipeline-per-table sprawl.
- Implementing the medallion architecture (Bronze/Silver/Gold) for progressively refined data quality.
- Bridging a data lake to a serving layer using Synapse external tables.
