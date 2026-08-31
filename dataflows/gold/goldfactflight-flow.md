# Gold Data Flow — sinkGoldFactFlight

## Flow
```
DimFlight (source: linkedservices_datalake) ─┐
                                              ├─► joinFactBooking (left outer join)
FactBookings ─────────────────────────────────┘
                       │
                       ├──► Status (aggregate by 'checkin_status') ──► alterStatus ──► sinkGoldFactFlight
                       │
                       └──► selectcols (rename) ──► totalflights (aggregate → 'Total Flights')
```

## Transformation Details
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
