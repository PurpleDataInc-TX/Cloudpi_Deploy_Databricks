# CloudPi — Databricks Integration

Set up CloudPi to collect Databricks **cost (billing)** and **metrics (optimization
recommendations)**. Unlike AWS/GCP/Azure, Databricks has **no install script** — it is
set up by following the two guides and running one export notebook **inside Databricks**.
CloudPi only reads the S3 output; it never calls Databricks or cloud APIs directly.

## Files

| File | What it is |
|------|-----------|
| `CloudPi_SystemTables_S3_Export_manifest_06.py` | The **export notebook**. Runs inside your Databricks workspace; exports 11 Unity Catalog system tables (8 required + 3 best-effort) to S3 as Parquet + per-month `manifest.json` + `_SUCCESS`. Scheduled as a daily Databricks Job. |
| `CloudPi_06_Databricks_Billing_Daily_Jobs_0624.docx` | **Billing setup guide** — enable the `system.billing` schema → create the S3 bucket → Unity Catalog Storage Credential + External Location → import the notebook → set widgets → schedule the daily Job → verify S3 → connect CloudPi. |
| `CloudPi_07_Databricks_Metrics_Recommendations_0624.docx` | **Metrics / recommendations guide** — enable the compute/lakeflow/access metric schemas (same daily job), then enable **Recommendations** in CloudPi for idle / over-provisioned clusters. |

## Order

1. **Do `06` first** (billing): get the notebook exporting to S3 and connected to CloudPi → Databricks spend appears in dashboards.
2. **Then `07`** (metrics): enable the extra system schemas and turn on Recommendations — it reuses the **same** daily job, no second notebook.

## Prerequisites

- Unity Catalog–enabled Databricks workspace + **account-level Metastore Admin**.
- An S3 bucket + a Unity Catalog Storage Credential / External Location for it.
- A CloudPi **Master Service Account (MSA)** that can read the bucket.

## Notes

- The notebook is configured via Job **widgets** (`bucket`, `org`, `cloud`, `lookback`,
  `metric_lookback`, `backfill_months`, `correction_lookback`, `workspace_region_map`).
- S3 layout (fixed): `s3://<bucket>/org=<ORG>/cloud=<CLOUD>/source=system_tables/table=<name>/...`
- Required tables fail the run if unreadable; best-effort tables (`pipelines`,
  `job_run_timeline`, `query_history`) are skipped, never fatal.
