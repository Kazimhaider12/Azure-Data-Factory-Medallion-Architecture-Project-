# 🥈 Silver Layer

## Purpose
The Silver layer cleans, standardizes, and transforms Bronze data so it's trustworthy and consistent for downstream use. Built as a single ADF **Data Flow** named **`SilverLayer`**.

## Pipelines / Transformation Chains

### 1. DimAirline
```
DimAirline → derivedColCount → alterRow1 → sinkDimAirline
```
- **derivedColCount** — adds a derived column (e.g., a count-based field) to the airline dimension.
- **alterRow1** — defines insert/update/upsert/delete conditions for the row.
- **sinkDimAirline** — writes the cleaned airline dimension to the Silver container.

📄 Full detail: [`dataflows/silver/dimairline-flow.md`](../dataflows/silver/dimairline-flow.md)

---

### 2. DimFlight
```
DimFlight → deletecolumn → alterUpsert → sinkDimFlight
                └────────→ deletecolumn → totalflights (aggregate)
```
- **deletecolumn** — removes unneeded columns from the raw flight data.
- **alterUpsert** — sets upsert logic before writing to sink.
- **sinkDimFlight** — writes the cleaned flight dimension to the Silver container.
- Parallel branch: a second `deletecolumn` step feeds into **totalflights**, an aggregate transformation producing flight-count metrics.

📄 Full detail: [`dataflows/silver/dimflight-flow.md`](../dataflows/silver/dimflight-flow.md)

---

### 3. DimPassenger
```
DimPassenger → renamecol → derivedgenderflag → derivedgenderflag → filterGreater25 → alterUpsert2 → sinkDimPassenger
```
- **renamecol** — standardizes column names.
- **derivedgenderflag (x2)** — derives gender-related flag columns from raw passenger attributes.
- **filterGreater25** — filters rows (e.g., passengers older than 25).
- **alterUpsert2** — sets upsert logic.
- **sinkDimPassenger** — writes the cleaned passenger dimension to the Silver container.

📄 Full detail: [`dataflows/silver/dimpassenger-flow.md`](../dataflows/silver/dimpassenger-flow.md)

---

### 4. FactBookings
```
FactBookings → castticketcost → alterUpsert3 → sinkFactBookings
```
- **castticketcost** — casts the ticket cost column to the correct data type.
- **alterUpsert3** — sets upsert logic.
- **sinkFactBookings** — writes the cleaned bookings fact table to the Silver container.

📄 Full detail: [`dataflows/silver/factbookings-flow.md`](../dataflows/silver/factbookings-flow.md)

---

### 5. DimAirport
```
DimAirport → DerCSet → alterUpsert4 → sinkDimAirport
```
- **DerCSet** — derives a column set on the raw airport data.
- **alterUpsert4** — sets upsert logic.
- **sinkDimAirport** — writes the cleaned airport dimension to the Silver container.

📄 Full detail: [`dataflows/silver/dimairport-flow.md`](../dataflows/silver/dimairport-flow.md)

---

## Output
Cleaned, standardized Silver datasets ready to be joined and aggregated in the Gold layer:
- DimAirline
- DimFlight (+ TotalFlights aggregate)
- DimPassenger (filtered, flagged)
- FactBookings
- DimAirport
