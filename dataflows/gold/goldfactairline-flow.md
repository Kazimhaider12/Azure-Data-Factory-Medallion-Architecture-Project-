# Gold Data Flow — sinkGoldFactAirline

## Flow
```
FactBookings (source: linkedservices_datalake) ─┐
                                                 ├─► joinAirline (left outer join)
DimAirline ───────────────────────────────────────┘
                       │
                       ▼
                 selectcol (rename joinAirline → selectcol, includes 'booking_id')
                       │
                       ▼
                 aggregateCol (aggregate by 'airline_name')
                       │
                       ▼
                 windowfunction (window-based aggregation, joins with original data)
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

## Transformation Details
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
