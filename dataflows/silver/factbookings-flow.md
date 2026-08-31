# Silver Data Flow — FactBookings

## Flow
```
FactBookings (source)
     │
     ▼
castticketcost   — casts ticket cost column to the correct data type
     │
     ▼
alterUpsert3     — sets upsert logic
     │
     ▼
sinkFactBookings — writes cleaned data to Silver container
```

## Transformation Details
| Step | Type | Description |
|---|---|---|
| FactBookings | Source | Reads raw bookings data from Bronze |
| castticketcost | Derived Column / Cast | Casts the ticket cost field to the correct numeric type |
| alterUpsert3 | Alter Row | Sets upsert logic |
| sinkFactBookings | Sink | Writes final cleaned FactBookings dataset to Silver |
