# Pipelines

This folder is where the exported ADF pipeline JSON definitions live once you export them from the Azure Data Factory Studio (Manage → ARM Template / Export, or via `az datafactory pipeline` CLI).

## Suggested pipelines to add here

| Pipeline | Purpose |
|---|---|
| `pl_bronze_ingest.json` | Copies data from On-Premise, GitHub, and SQL sources into the Bronze layer |
| `pl_silver_transform.json` | Runs the `SilverLayer` Data Flow across all 5 Silver entities |
| `pl_gold_transform.json` | Runs the `GoldLayer` Data Flow to produce `GoldFactFlight` and `GoldFactAirline` |
| `pl_master_orchestrator.json` | Master pipeline chaining Bronze → Silver → Gold in sequence |

## How to export from ADF Studio
1. Open Azure Data Factory Studio.
2. Go to **Manage** → **ARM Template** → **Export ARM Template**, or right-click a pipeline in the Author view and choose **Download support files**.
3. Place the exported JSON files in this folder, matching the naming convention above (or your own).
4. Commit and push.

## How to import into a new ADF instance
1. In ADF Studio, go to **Manage** → **ARM Template** → **Import ARM Template**.
2. Point it to the JSON files in this folder (or deploy via Azure DevOps / GitHub Actions using the ARM template).
3. Reconnect the Linked Services (On-Premise IR, GitHub, SQL) to your own credentials/environment.
