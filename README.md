# Driving Demand: Explainable AI-Driven Driver Nudge System
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

Ride-hailing platforms such as Uber and Lyft frequently face mismatches between driver supply and rider demand. These inefficiencies lead to longer wait times, increased driver idle time, and lost revenue.
Current platform systems rely on automated dispatch and surge mechanisms that operate behind the scenes. While drivers receive recommendations, they are not provided with the underlying data or reasoning that explains why certain areas are prioritized. As a result, drivers have limited visibility into how these suggestions are generated, which can reduce trust and make it harder to act on them effectively.

This project introduces a data-driven, explainable decision support system that surfaces high-demand locations and communicates them through concise, natural-language nudges. The goal is to provide clear, data-backed insights that enable drivers to make informed repositioning decisions independently.


> **The core insight:** Uber tells drivers WHERE to go. We tell them WHY — and let them decide.

Recommendation quality is evaluated using a temporal holdout approach: January–February rankings are validated against held-out March data, achieving a **100% hit rate across all 8 time windows** — 5x better than the 20% random baseline.

---

## Dataset Sources

| Dataset | Description | Source |
|--------|-------------|--------|
| NYC TLC Yellow Taxi Trip Records (Jan–Mar 2023) | 9.4M trip records, primary dataset | https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page |
| NYC Taxi Zone Lookup CSV | 265 geographic zones mapped to boroughs | https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page |

**Big Data characteristics covered:**
- **Volume** — 9.4 million records across three monthly Parquet files (~145 MB compressed)
- **Veracity** — real-world data with erroneous dates, null zone IDs, outlier fares, and duplicate records, all addressed in the cleaning pipeline

---

## Technology Stack

| Layer | Tool |
|-------|------|
| Compute & Orchestration | Databricks (shared workspace via Carlson School IT) |
| Batch ETL & Aggregation | Apache Spark / PySpark |
| Storage | Delta Lake (Unity Catalog) |
| Heuristic Ranking | PySpark window functions |
| LLM Nudge Generation | OpenAI API (gpt-4o-mini) |
| Visualization | Matplotlib, Seaborn |

*Shared Databricks workspace provided by Carlson School IT.*

---

## Repository Structure

```
team7-driver-nudge-system/
├── notebooks/
│   └── driving_demand_pipeline.ipynb    # Full end-to-end pipeline (Parts 1–11)
├── data/
│   ├── taxi_zone_lookup.csv             # NYC taxi zone lookup (265 zones)
│   └── yellow_tripdata_2023_sample.parquet  # Stratified sample for testing
├── docs/
│   ├── team7_handout.pdf                # Two-page project handout
│   └── team7_flier.pdf                  # Project flier
├── .gitignore
└── README.md
```

> **Note:** The full monthly Parquet files (~145 MB total) are not committed to this repository. Download them directly from the NYC TLC link above and upload to your Databricks volume at `/Volumes/msbabigdata/spark/trend_market_project/`.

---

## Pipeline Architecture

The entire pipeline runs as a single notebook (`driving_demand_pipeline.ipynb`) with 11 sequential parts:

```
NYC TLC Parquet Files (Jan, Feb, Mar 2023)
            ↓
  Part 1  — Initialize storage, ingest and normalize all three Parquet files
            ↓
  Part 2  — Clean trip records (date filter, zone ID validation, outlier removal,
            deduplication, timezone normalization)
            ↓
  Part 3  — Join with taxi zone lookup (attach borough and zone names)
            ↓
  Part 4  — Write cleaned trips to Delta Lake
            ↓
  Part 5  — Aggregate demand by zone × time bucket × day type
            Compute demand scores (zone trips ÷ citywide average)
            Flag low-confidence cells (insufficient trips, days, or high variance)
            Split into training (Jan–Feb) and validation (March) sets
            ↓
  Part 6  — Write demand scores and validation scores to Delta Lake
            ↓
  Part 7  — Heuristic ranking: select top-5 high-confidence zones per window
            ↓
  Part 8  — Write ranked zones to Delta Lake
            ↓
  Part 9  — LLM nudge generation (OpenAI API)
            Grounding checks: zone cited, no forbidden words, under 60 words,
            no raw decimal scores leaked
            ↓
  Part 10 — Quantitative evaluation: temporal holdout hit-rate analysis
            Recommended zones (Jan–Feb) validated against March demand
            ↓
  Part 11 — Visualizations (5 figures saved to Unity Catalog volume)
```

---

## Setup & Installation

### Prerequisites
- Databricks workspace with Unity Catalog enabled
- OpenAI API key
- The three NYC TLC Parquet files uploaded to `/Volumes/msbabigdata/spark/trend_market_project/`
- The taxi zone lookup CSV uploaded to the same volume path

### Steps

1. Clone this repository:
```bash
git clone https://github.com/DRJohnson21/team7-driver-nudge-system.git
```

2. Download the NYC TLC Yellow Taxi data (January, February, and March 2023) from the link above and upload to your Databricks volume:
```
/Volumes/msbabigdata/spark/trend_market_project/yellow_tripdata_2023-Jan.parquet
/Volumes/msbabigdata/spark/trend_market_project/yellow_tripdata_2023-Feb.parquet
/Volumes/msbabigdata/spark/trend_market_project/yellow_tripdata_2023-March.parquet
/Volumes/msbabigdata/spark/trend_market_project/Taxi_zone_lookup.csv
```

3. Store your OpenAI API key in Databricks Secrets:
```bash
# Run in a %sh cell or your local terminal with the Databricks CLI installed
databricks secrets create-scope team7
databricks secrets put-secret team7 openai_api_key --string-value "your-key-here"
```

4. Import `notebooks/driving_demand_pipeline.ipynb` into your Databricks workspace via **Workspace → Import**

5. Run all cells from top to bottom (Parts 1–11 in order)

---

## Delta Tables Created

All intermediate and final outputs are persisted as Delta Lake tables under the `msbabigdata.spark` catalog:

| Table | Description |
|-------|-------------|
| `trend_market_cleaned_trips` | Cleaned and zone-enriched trip records |
| `trend_market_demand_scores` | Zone-window demand scores (Jan–Feb training data) |
| `trend_market_ranked_zones` | Top-5 ranked zones per time window |
| `trend_market_nudge_messages` | LLM-generated driver nudge messages |
| `trend_market_validation_scores` | March holdout demand scores for evaluation |

---

## Evaluation Results

| Metric | Value |
|--------|-------|
| Overall hit rate | 100% (40/40 zones) |
| vs Random baseline (20%) | 5.0x better |
| Nudges passing all grounding checks | 8/8 |
| Time windows with 100% hit rate | All 8 |

A recommended zone is confirmed as a true hit if it ranks in the top 20th percentile of demand within its time window in the held-out March data.

---

## Sample Generated Nudge

**Weekday Night — Top zone: Upper East Side North (13.4x)**
> *"Weekday night: Upper East Side North is about 13.5x busier than average with 52,602 pickups recorded across the training window. Penn Station/Madison Sq West follows at roughly 13.3x with 52,200 pickups. Focus on Upper East Side North for potential opportunities."*

---

## Bibliography & Credits
- NYC Taxi and Limousine Commission (TLC) — https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page
- Apache Spark — https://spark.apache.org
- Delta Lake — https://delta.io
- OpenAI API — https://platform.openai.com
- Carlson School IT — shared Databricks workspace provisioned for this project
