# Explainable AI-Driven Driver Nudge System
### Demand-Aware Decision Support for Ride-Hailing Platforms

**Team 7** | Master of Science in Business Analytics | Carlson School of Management, University of Minnesota

> This project repository is created in partial fulfillment of the requirements for the Big Data Analytics course offered by the Master of Science in Business Analytics program at the Carlson School of Management, University of Minnesota.

---

## Team Members
- Davey Johnson
- Hengrui Li
- Huiguo Liu
- Mansi Malpani
- Mounika Polamreddy

---

## Project Overview

Ride-hailing platforms frequently experience mismatches between driver availability and rider demand, leading to increased passenger wait times and inefficient driver utilization. While large platforms deploy complex dispatch systems, these are typically opaque and provide limited guidance to individual drivers.

This project proposes a batch-based, demand-aware driver decision support system that leverages large-scale historical trip data to identify spatiotemporal demand patterns across NYC taxi zones. Instead of automating dispatch, the system generates simple, interpretable heuristics and ranked recommendations, then uses a large language model (LLM) to translate these signals into clear, explainable nudge messages that help drivers make better repositioning decisions independently.

---

## Dataset Sources

| Dataset | Source |
|--------|--------|
| NYC TLC Yellow Taxi Trip Records (Jan–Mar 2023) | https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page |
| NYC Taxi Zone Lookup CSV | https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page |
| City of Chicago Taxi Trips 2013–2023 (optional) | https://data.cityofchicago.org/Transportation/Taxi-Trips-2013-2023-/wrvz-psew/about_data |
| City of Chicago Taxi Trips 2024+ (optional) | https://data.cityofchicago.org/Transportation/Taxi-Trips-2024-/ajtu-isnz/about_data |

---

## Technology Stack

| Layer | Tool |
|-------|------|
| Compute & Orchestration | Databricks (Apache Spark) |
| Batch ETL & Aggregation | Apache Spark (PySpark) |
| Storage | Delta Lake (DBFS) |
| Heuristic Ranking | PySpark |
| LLM Nudge Generation | OpenAI API |
| Visualization | Matplotlib, Seaborn |

---

## Repository Structure

```
team7-driver-nudge-system/
├── data/               # Sample data and zone lookup CSV (large files not committed)
├── notebooks/          # Databricks notebooks exported as .ipynb
│   ├── 00_data_load_test.ipynb
│   ├── 01_ingestion_cleaning.ipynb
│   ├── 02_demand_aggregation.ipynb
│   ├── 03_heuristic_ranking.ipynb
│   ├── 04_llm_nudge_generation.ipynb
│   └── 05_evaluation_visualization.ipynb
├── docs/               # Project handout, proposal PDF, and reference materials
├── .gitignore
└── README.md
```

---

## Setup & Installation

### Prerequisites
- Databricks Community Edition account (https://community.cloud.databricks.com)
- Python 3.8+
- OpenAI API key (stored in a local `.env` file — never committed to GitHub)

### Steps
1. Clone this repository:
```bash
git clone https://github.com/DRJohnson21/team7-driver-nudge-system.git
cd team7-driver-nudge-system
```

2. Download the NYC TLC data from the link above and place Parquet files in the `data/` folder

3. Import the notebooks from the `notebooks/` folder into your Databricks workspace via **Workspace → Import**

4. Set your OpenAI API key as an environment variable in your Databricks notebook:
```python
import os
os.environ["OPENAI_API_KEY"] = "your-key-here"
```

5. Run the notebooks in order: `00` → `01` → `02` → `03` → `04` → `05`

---

## Pipeline Architecture

```
NYC TLC Parquet Files
        ↓
  [01] Spark Ingestion & Cleaning
        ↓
  [02] Zone-Level Demand Aggregation (hourly buckets)
        ↓
  [03] Heuristic Ranking & Confidence Filtering
        ↓
  [04] LLM Nudge Generation (OpenAI API)
        ↓
  [05] Evaluation & Visualization
```

---

## Bibliography & Credits
- NYC Taxi and Limousine Commission (TLC) — https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page
- City of Chicago Data Portal — https://data.cityofchicago.org
- Apache Spark — https://spark.apache.org
- OpenAI API — https://platform.openai.com
- Delta Lake — https://delta.io
