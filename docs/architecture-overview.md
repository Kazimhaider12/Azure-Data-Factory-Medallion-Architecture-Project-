# Architecture Overview

## High-Level Flow

```
Sources
┌────────────────┐
│  On-Premise     │──┐
│  GitHub         │──┼──►  Bronze  ──►  Silver  ──►  Gold
│  SQL            │──┘     (raw)     (SilverLayer  (GoldLayer
└────────────────┘                    Data Flow)    Data Flow)
```

## Layer-by-Layer

### 1. Sources
| Source | Connection Type | Notes |
|---|---|---|
| On-Premise | Self-Hosted Integration Runtime | Local database/file system, connected securely to ADF |
| GitHub | HTTP / Git connector | Pulls versioned reference/config data |
| SQL | Azure SQL / SQL Server connector | Transactional source data |

### 2. Bronze Layer
- Raw, minimally transformed data.
- Original structure preserved.
- Landed via Copy Activities into the Bronze container in ADLS Gen2.
- Full details: [`bronze-layer.md`](bronze-layer.md)

### 3. Silver Layer
- Built using a single Data Flow: **`SilverLayer`**
- Cleans, deduplicates, renames, and standardizes 5 entities: DimAirline, DimFlight, DimPassenger, FactBookings, DimAirport.
- Full details: [`silver-layer.md`](silver-layer.md)

### 4. Gold Layer
- Built using a single Data Flow: **`GoldLayer`**
- Joins Silver entities and applies business-level aggregations for 2 fact outputs: `sinkGoldFactFlight`, `sinkGoldFactAirline`.
- Full details: [`gold-layer.md`](gold-layer.md)

## Diagram Reference

The original architecture diagram (Sources → Bronze → Silver → Gold, with `SilverLayer` and `GoldLayer` Data Flow activities annotated) is included as `docs/images/architecture-diagram.png` — add your exported diagram image to this path.
