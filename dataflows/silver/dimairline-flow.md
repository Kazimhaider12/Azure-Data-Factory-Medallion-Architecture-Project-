# Silver Data Flow — DimAirline

## Flow
```
DimAirline (source)
     │
     ▼
derivedColCount   — derived column transformation, adds a count-based column
     │
     ▼
alterRow1         — sets insert/upsert/delete row policy
     │
     ▼
sinkDimAirline    — writes cleaned data to Silver container
```

## Transformation Details
| Step | Type | Description |
|---|---|---|
| DimAirline | Source | Reads raw airline data from Bronze |
| derivedColCount | Derived Column | Adds a derived/count column to the dataset |
| alterRow1 | Alter Row | Defines row-level insert/update/upsert/delete conditions |
| sinkDimAirline | Sink | Writes final cleaned DimAirline dataset to Silver |
