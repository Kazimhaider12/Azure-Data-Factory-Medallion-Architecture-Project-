# Azure-Data-Factory-Medallion-Architecture-Project-

Azure Data Factory — Medallion Architecture Project ☁️
An end-to-end data platform built using Azure Data Factory and the Medallion Architecture pattern (Bronze → Silver → Gold), ingesting data from multiple heterogeneous sources and progressively refining it into business-ready analytics datasets.
---
🏗️ Architecture Overview
```
Sources                          Bronze         Silver         Gold
─────────                        ──────         ──────         ────
☁️  On-Premise   ─┐
🐙  GitHub        ├──────────►   Raw Data  ───►  Cleaned  ───►  Business-
🗄️  SQL          ─┘                              & Transformed    Ready Data
                                                 (Data Flow:      (Data Flow:
                                                  SilverLayer)     GoldLayer)
```
Sources → Bronze
Data lands in the Bronze layer from three source systems:
On-Premise systems (via Self-Hosted Integration Runtime)
GitHub (via HTTP/Git connector)
SQL (via Azure SQL / SQL Server connector)
Bronze → Silver
A dedicated Data Flow activity (`SilverLayer`) cleans, transforms, and standardizes the raw Bronze data — deduplication, renaming, derived columns, filtering, and upserts — landing into clean Silver datasets.
Silver → Gold
A dedicated Data Flow activity (`GoldLayer`) joins, aggregates, and applies business logic across the Silver datasets to produce final business-ready datasets (KPIs, reporting-level facts) in the Gold layer.
See the full architecture diagram: `docs/architecture-overview.md`
---
📁 Repository Structure
```
azure-medallion-architecture-adf/
├── README.md
├── docs/
│   ├── architecture-overview.md
│   ├── bronze-layer.md
│   ├── silver-layer.md
│   └── gold-layer.md
├── dataflows/
│   ├── silver/
│   │   ├── dimairline-flow.md
│   │   ├── dimflight-flow.md
│   │   ├── dimpassenger-flow.md
│   │   ├── factbookings-flow.md
│   │   └── dimairport-flow.md
│   └── gold/
│       ├── goldfactflight-flow.md
│       └── goldfactairline-flow.md
└── pipelines/
    └── README.md
```
---
🥉 Bronze Layer
Raw data ingested as-is from source systems, with minimal transformation, preserving original structure for traceability and reprocessing.
Sources: On-Premise, GitHub, SQL
📄 Details: `docs/bronze-layer.md`
---
🥈 Silver Layer
Cleaned, deduplicated, and standardized datasets — built using the `SilverLayer` Data Flow, covering 5 pipelines:
Entity	Key Transformations
DimAirline	Derived column count → Alter row → Sink
DimFlight	Delete column → Alter upsert → Sink (+ branch: aggregate total flights)
DimPassenger	Rename column → Derived gender flags (x2) → Filter (age > 25) → Alter upsert → Sink
FactBookings	Cast ticket cost → Alter upsert → Sink
DimAirport	Derived column set → Alter upsert → Sink
📄 Full breakdown: `docs/silver-layer.md`
---
🥇 Gold Layer
Business-ready, aggregated datasets — built using the `GoldLayer` Data Flow, joining Silver datasets and applying business logic:
Output	Key Transformations
sinkGoldFactFlight	Join (DimFlight + FactBookings) → Aggregate by check-in status → Alter status → Sink; parallel branch aggregates Total Flights
sinkGoldFactAirline	Join (FactBookings + DimAirline) → Aggregate by airline name → Window function (top-N per group) → Filter Top 5 → Alter/update → Sink
📄 Full breakdown: `docs/gold-layer.md`
---
🛠️ Tech Stack
Azure Data Factory (Pipelines + Data Flows)
Azure Data Lake Storage Gen2 (Bronze / Silver / Gold containers)
Linked Services: On-Premise (Self-Hosted IR), GitHub, Azure SQL
Mapping Data Flows (Spark-based transformations)
---
🚀 How to Use This Repo
Clone the repo.
Review `docs/architecture-overview.md` for the full pipeline design.
Recreate the linked services (On-Premise, GitHub, SQL) in your own ADF instance.
Import/recreate the `SilverLayer` and `GoldLayer` Data Flows using the transformation breakdowns in `dataflows/`.
Wire up the pipelines per `pipelines/README.md`.
---
📌 Project Status
✅ Bronze ingestion (On-Premise, GitHub, SQL) — working
✅ Silver Data Flow (5 entities) — working
✅ Gold Data Flow (2 fact outputs) — working
Actively documenting and extending as the project grows.
#Azure #AzureDataFactory #MedallionArchitecture #DataEngineering
