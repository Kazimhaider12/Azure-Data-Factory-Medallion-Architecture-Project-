# 🥉 Bronze Layer

## Purpose
The Bronze layer is the landing zone for raw data, ingested as close to its original form as possible from all source systems. It preserves data lineage and allows reprocessing without going back to the source.

## Sources Ingested

### 1. On-Premise
- Connected via **Self-Hosted Integration Runtime (IR)**, installed within the on-prem network.
- Allows ADF to securely reach local databases/file systems without exposing them directly to the internet.

### 2. GitHub
- Connected via ADF's HTTP/Git-based connector.
- Used to pull versioned reference or configuration-style data into the pipeline.

### 3. SQL
- Connected via Azure SQL / SQL Server linked service.
- Source of core transactional/relational data (e.g., bookings, flights, passengers, airports, airlines).

## Process
1. Copy Activities pull data from each source.
2. Data lands in the **Bronze container** in Azure Data Lake Storage Gen2, organized by entity/source.
3. No business logic is applied at this stage — only raw ingestion.

## Output
Raw datasets available for the Silver layer to consume:
- Raw Airline data
- Raw Flight data
- Raw Passenger data
- Raw Bookings data
- Raw Airport data
