# 🥇 Gold Layer

## Purpose
The Gold layer applies business-level logic — joins, aggregations, and window functions across Silver datasets — to produce final, reporting-ready fact tables. Built as a single ADF **Data Flow** named **`GoldLayer`**.

## Transformation Chains

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
- **joinFactBooking** — left outer join between `DimFlight` and `FactBookings`.
- **Status** — aggregates data by `checkin_status`, producing check-in status metrics.
- **alterStatus** — applies expressions to alter/format rows before sinking.
- **sinkGoldFactFlight** — writes the final flight-level fact table to the Gold container.
- Parallel branch: **selectcols** renames the joined output, then **totalflights** aggregates to produce a `Total Flights` metric.

📄 Full detail: [`dataflows/gold/goldfactflight-flow.md`](../dataflows/gold/goldfactflight-flow.md)

---

### 2. sinkGoldFactAirline
```
FactBookings ─┐
              ├─► joinAirline (left outer join on FactBookings + DimAirline)
DimAirline ────┘
                    │
                    ▼
              selectcol (rename with booking_id, etc.)
                    │
                    ▼
              aggregateCol (aggregate by 'airline_name')
                    │
                    ▼
              windowfunction (aggregates over a window, joins back with original data)
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
- **joinAirline** — left outer join between `FactBookings` and `DimAirline`.
- **selectcol** — renames/selects relevant columns (e.g., `booking_id`).
- **aggregateCol** — aggregates data by `airline_name`.
- **windowfunction** — applies a window function (ranking/aggregation over a partition), joined back with the original dataset.
- **filtertop5** — filters to the top 5 rows based on the ranking column.
- **alterupdate** — applies final row-level expressions before sinking.
- **sinkGoldFactAirline** — writes the final airline-level fact table (Top 5 airlines) to the Gold container.

📄 Full detail: [`dataflows/gold/goldfactairline-flow.md`](../dataflows/gold/goldfactairline-flow.md)

---

## Output
Business-ready datasets consumed directly by BI tools (e.g., Power BI):
- **GoldFactFlight** — flight-level fact table with check-in status metrics + total flights
- **GoldFactAirline** — top 5 airlines by aggregated booking metrics
