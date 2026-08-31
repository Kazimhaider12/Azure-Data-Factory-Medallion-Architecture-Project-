# Silver Data Flow — DimPassenger

## Flow
```
DimPassenger (source)
     │
     ▼
renamecol              — standardizes column names
     │
     ▼
derivedgenderflag       — derives gender-related flag column
     │
     ▼
derivedgenderflag       — second derived gender-flag transformation
     │
     ▼
filterGreater25         — filters rows (e.g., age > 25)
     │
     ▼
alterUpsert2             — sets upsert logic
     │
     ▼
sinkDimPassenger         — writes cleaned data to Silver container
```

## Transformation Details
| Step | Type | Description |
|---|---|---|
| DimPassenger | Source | Reads raw passenger data from Bronze |
| renamecol | Rename Column | Standardizes column naming conventions |
| derivedgenderflag (x2) | Derived Column | Derives gender-related flag fields from raw attributes |
| filterGreater25 | Filter | Filters rows based on a condition (e.g., passenger age greater than 25) |
| alterUpsert2 | Alter Row | Sets upsert logic |
| sinkDimPassenger | Sink | Writes final cleaned DimPassenger dataset to Silver |
