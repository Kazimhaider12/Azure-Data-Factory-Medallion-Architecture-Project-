# Azure Data Factory — Medallion Architecture Project ☁️

An end-to-end data platform built using **Azure Data Factory** and the **Medallion Architecture** pattern (Bronze → Silver → Gold), ingesting data from multiple heterogeneous sources and progressively refining it into business-ready analytics datasets.

---

## 🏗️ Architecture Overview

```
Sources                          Bronze         Silver              Gold
─────────                        ──────         ──────              ────
☁️  On-Premise   ─┐
🐙  GitHub        ├──────────►   Raw Data  ───►  Cleaned &     ───►  Business-
🗄️  SQL          ─┘                              Transformed         Ready Data
                                                 (Data Flow:         (Data Flow:
                                                  SilverLayer)        GoldLayer)
```

**Sources → Bronze**
Data lands in the Bronze layer from three source systems:
- **On-Premise** systems (via Self-Hosted Integration Runtime)
- **GitHub** (via HTTP/Git connector)
- **SQL** (via Azure SQL / SQL Server connector)

**Bronze → Silver**
A dedicated Data Flow activity (**`SilverLayer`**) cleans, transforms, and standardizes the raw Bronze data — deduplication, renaming, derived columns, filtering, and upserts — landing into clean Silver datasets.

**Silver → Gold**
A dedicated Data Flow activity (**`GoldLayer`**) joins, aggregates, and applies business logic across the Silver datasets to produce final business-ready datasets (KPIs, reporting-level facts) in the Gold layer.

---

## 🛠️ Tech Stack

- Azure Data Factory (Pipelines + Data Flows)
- Azure Data Lake Storage Gen2 (Bronze / Silver / Gold containers)
- Linked Services: On-Premise (Self-Hosted IR), GitHub, Azure SQL
- Mapping Data Flows (Spark-based transformations)

---

## 🥉 Bronze Layer

### Purpose
The Bronze layer is the landing zone for raw data, ingested as close to its original form as possible from all source systems. It preserves data lineage and allows reprocessing without going back to the source.

### Sources Ingested

**1. On-Premise**
- Connected via **Self-Hosted Integration Runtime (IR)**, installed within the on-prem network.
- Allows ADF to securely reach local databases/file systems without exposing them directly to the internet.

**2. GitHub**
- Connected via ADF's HTTP/Git-based connector.
- Used to pull versioned reference or configuration-style data into the pipeline.

**3. SQL**
- Connected via Azure SQL / SQL Server linked service.
- Source of core transactional/relational data (e.g., bookings, flights, passengers, airports, airlines).

### Process
1. Copy Activities pull data from each source.
2. Data lands in the **Bronze container** in Azure Data Lake Storage Gen2, organized by entity/source.
3. No business logic is applied at this stage — only raw ingestion.

### Output
Raw datasets available for the Silver layer to consume: Raw Airline, Raw Flight, Raw Passenger, Raw Bookings, Raw Airport data.

---

## 🥈 Silver Layer

### Purpose
The Silver layer cleans, standardizes, and transforms Bronze data so it's trustworthy and consistent for downstream use. Built as a single ADF **Data Flow** named **`SilverLayer`**.

### 1. DimAirline
```
DimAirline → derivedColCount → alterRow1 → sinkDimAirline
```
| Step | Type | Description |
|---|---|---|
| DimAirline | Source | Reads raw airline data from Bronze |
| derivedColCount | Derived Column | Adds a derived/count column to the dataset |
| alterRow1 | Alter Row | Defines row-level insert/update/upsert/delete conditions |
| sinkDimAirline | Sink | Writes final cleaned DimAirline dataset to Silver |

### 2. DimFlight
```
DimFlight → deletecolumn (branch point)
                ├──► alterUpsert → sinkDimFlight
                └──► deletecolumn → totalflights (aggregate)
```
| Step | Type | Description |
|---|---|---|
| DimFlight | Source | Reads raw flight data from Bronze |
| deletecolumn | Delete Column | Removes unneeded columns from raw flight data |
| alterUpsert | Alter Row | Sets upsert logic for the main flight dimension |
| sinkDimFlight | Sink | Writes final cleaned DimFlight dataset to Silver |
| deletecolumn (branch) | Delete Column | Second cleanup path feeding the aggregate branch |
| totalflights | Aggregate | Produces flight-count metrics (e.g., total flights) |

### 3. DimPassenger
```
DimPassenger → renamecol → derivedgenderflag → derivedgenderflag → filterGreater25 → alterUpsert2 → sinkDimPassenger
```
| Step | Type | Description |
|---|---|---|
| DimPassenger | Source | Reads raw passenger data from Bronze |
| renamecol | Rename Column | Standardizes column naming conventions |
| derivedgenderflag (x2) | Derived Column | Derives gender-related flag fields from raw attributes |
| filterGreater25 | Filter | Filters rows based on a condition (e.g., passenger age greater than 25) |
| alterUpsert2 | Alter Row | Sets upsert logic |
| sinkDimPassenger | Sink | Writes final cleaned DimPassenger dataset to Silver |

### 4. FactBookings
```
FactBookings → castticketcost → alterUpsert3 → sinkFactBookings
```
| Step | Type | Description |
|---|---|---|
| FactBookings | Source | Reads raw bookings data from Bronze |
| castticketcost | Derived Column / Cast | Casts the ticket cost field to the correct numeric type |
| alterUpsert3 | Alter Row | Sets upsert logic |
| sinkFactBookings | Sink | Writes final cleaned FactBookings dataset to Silver |

### 5. DimAirport
```
DimAirport → DerCSet → alterUpsert4 → sinkDimAirport
```
| Step | Type | Description |
|---|---|---|
| DimAirport | Source | Reads raw airport data from Bronze |
| DerCSet | Derived Column | Derives a set of columns on the raw airport data |
| alterUpsert4 | Alter Row | Sets upsert logic |
| sinkDimAirport | Sink | Writes final cleaned DimAirport dataset to Silver |

### Silver Output
Cleaned, standardized Silver datasets ready to be joined and aggregated in the Gold layer:
DimAirline, DimFlight (+ TotalFlights aggregate), DimPassenger (filtered, flagged), FactBookings, DimAirport.

---

## 🥇 Gold Layer

### Purpose
The Gold layer applies business-level logic — joins, aggregations, and window functions across Silver datasets — to produce final, reporting-ready fact tables. Built as a single ADF **Data Flow** named **`GoldLayer`**.

### 1. sinkGoldFactFlight
```
DimFlight ─┐
           ├─► joinFactBooking (left outer join on DimFlight + FactBookings)
FactBookings┘
                    │
                    ├──► Status (aggregate by 'checkin_status') ──► alterStatus ──► sinkGoldFactFlight
                    │
                    └──► selectcols (rename) ──► totalflights (aggregate, produces 'Total Flights')
```
| Step | Type | Description |
|---|---|---|
| DimFlight | Source | Imports data from `linkedservices_datalake` |
| FactBookings | Source | Bookings fact data (joined in) |
| joinFactBooking | Join | Left outer join on `DimFlight` and `FactBookings` |
| Status | Aggregate | Aggregates joined data by `checkin_status`, producing check-in status metrics |
| alterStatus | Alter Row | Adds expressions to alter rows before sinking |
| sinkGoldFactFlight | Sink | Writes final flight-level fact table to Gold |
| selectcols | Select/Rename | Renames `joinFactBooking` output columns |
| totalflights | Aggregate | Aggregates data producing the `Total Flights` metric |

### 2. sinkGoldFactAirline
```
FactBookings ─┐
              ├─► joinAirline (left outer join on FactBookings + DimAirline)
DimAirline ────┘
                    │
                    ▼
              selectcol (rename, includes 'booking_id')
                    │
                    ▼
              aggregateCol (aggregate by 'airline_name')
                    │
                    ▼
              windowfunction (window aggregation, joins with original data)
                    │
                    ▼
              filtertop5 (filter using expressions on column 'Top')
                    │
                    ▼
              alterupdate (add expressions to alter rows)
                    │
                    ▼
              sinkGoldFactAirline
```
| Step | Type | Description |
|---|---|---|
| FactBookings | Source | Imports data from `linkedservices_datalake` |
| DimAirline | Source | Airline dimension data (joined in) |
| joinAirline | Join | Left outer join on `FactBookings` and `DimAirline` |
| selectcol | Select/Rename | Renames `joinAirline` output, includes columns such as `booking_id` |
| aggregateCol | Aggregate | Aggregates data by `airline_name` |
| windowfunction | Window | Applies a window function, aggregating over a partition and joining back with original data |
| filtertop5 | Filter | Filters rows using expressions on the `Top` ranking column (Top 5) |
| alterupdate | Alter Row | Adds expressions to alter rows before sinking |
| sinkGoldFactAirline | Sink | Writes final Top-5-airline fact table to Gold |

### Gold Output
Business-ready datasets consumed directly by BI tools (e.g., Power BI):
- **GoldFactFlight** — flight-level fact table with check-in status metrics + total flights
- **GoldFactAirline** — top 5 airlines by aggregated booking metrics

---


## 📌 Project Status

✅ Bronze ingestion (On-Premise, GitHub, SQL) — working
✅ Silver Data Flow (5 entities) — working
✅ Gold Data Flow (2 fact outputs) — working

Actively documenting and extending as the project grows.

#Azure #AzureDataFactory #MedallionArchitecture #DataEngineering
