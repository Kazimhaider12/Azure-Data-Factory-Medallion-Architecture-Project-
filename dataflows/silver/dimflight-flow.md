# Silver Data Flow — DimFlight

## Flow
```
DimFlight (source)
     │
     ▼
deletecolumn (branch point)
     ├────────────────────────────┐
     ▼                            ▼
alterUpsert                deletecolumn
     │                            │
     ▼                            ▼
sinkDimFlight               totalflights (aggregate)
```

## Transformation Details
| Step | Type | Description |
|---|---|---|
| DimFlight | Source | Reads raw flight data from Bronze |
| deletecolumn | Delete Column | Removes unneeded columns from raw flight data |
| alterUpsert | Alter Row | Sets upsert logic for the main flight dimension |
| sinkDimFlight | Sink | Writes final cleaned DimFlight dataset to Silver |
| deletecolumn (branch) | Delete Column | Second cleanup path feeding the aggregate branch |
| totalflights | Aggregate | Produces flight-count metrics (e.g., total flights) |
