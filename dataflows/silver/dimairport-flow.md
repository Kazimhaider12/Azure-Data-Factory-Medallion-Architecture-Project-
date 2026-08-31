# Silver Data Flow — DimAirport

## Flow
```
DimAirport (source)
     │
     ▼
DerCSet          — derives a column set on raw airport data
     │
     ▼
alterUpsert4     — sets upsert logic
     │
     ▼
sinkDimAirport   — writes cleaned data to Silver container
```

## Transformation Details
| Step | Type | Description |
|---|---|---|
| DimAirport | Source | Reads raw airport data from Bronze |
| DerCSet | Derived Column | Derives a set of columns on the raw airport data |
| alterUpsert4 | Alter Row | Sets upsert logic |
| sinkDimAirport | Sink | Writes final cleaned DimAirport dataset to Silver |
