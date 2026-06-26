# MsFabric — E-Commerce Medallion Pipeline (Microsoft Fabric)

A Microsoft Fabric data engineering project that ingests e-commerce data into a Lakehouse Bronze layer via an automated Data Factory pipeline, then transforms it toward a Silver layer using a PySpark notebook.

## Overview

This repo contains the **Bronze ingestion** and **Bronze → Silver transformation** stages of a Fabric-based e-commerce analytics pipeline. Raw CSV files are pulled over HTTP from a source GitHub repo, converted to Parquet, and landed in a Fabric Lakehouse. A notebook then promotes that data to a cleaned Silver layer.

## Tech Stack

- **Microsoft Fabric** — Lakehouse, Data Factory Pipelines, Notebooks
- **Azure Data Factory pipeline definitions** (ARM-style pipeline JSON)
- **PySpark** — notebook-based transformation
- **Parquet (Snappy compression)** — Bronze storage format
- **Data source** — Olist Brazilian E-Commerce Public Dataset + custom sample CSVs

## Pipeline: `ecom_copy_activity`

Verified directly from `ecom_copy_activity.json`:

- **Parameters**: `fileList` (array) — defaults to 5 CSV files hosted in `rohanrs09/MS-Fabric-Ecommerce-End-to-End-Project` on GitHub (`customers.csv`, `orders.csv`, `payments.csv`, `support_tickets.csv`, `web_activities.csv`)
- **Activity**: `ForEachActivity` iterates over `fileList`; inside it, a `Copy` activity named `CopyGithubtoLak` runs per file:
  - **Source**: `DelimitedTextSource` reading each file over HTTP (`HttpServerLocation`, GET request) from the GitHub-hosted CSV
  - **Sink**: `ParquetSink` writing into the **Bronze** folder of the `ecommerce_lakehouse` Lakehouse, Snappy-compressed, filename derived from the source path
- **Linked services**:
  - `finalcon shelkerohan99` → HTTP Server connection (source)
  - `ecommerce_lakehouse` → Lakehouse connection (sink)
- **Workspace ID**: `91c0411b-6b86-4060-8d72-97c281a6c67a`

This is a parameterized, reusable ingestion pattern — adding a new source file only requires appending its path to `fileList`, with no pipeline changes.

## Notebook: `ecom_bronze_to_silver.ipynb`

> **Note on accuracy:** I could not pull the actual cell-by-cell code in this notebook — GitHub serves `.ipynb` files through a JavaScript-based viewer and blocks automated/raw access to the underlying content. The description below is inferred from the notebook's name and its position in the pipeline (immediately after Bronze ingestion), **not verified from the code itself**.

Based on that inference, this notebook likely:
- Reads the Parquet files from the Lakehouse Bronze folder
- Applies PySpark transformations — cleaning, type casting, deduplication, and/or joining across the e-commerce tables (customers, orders, payments, etc.)
- Writes the cleaned result to a Silver Delta table in the Lakehouse

**Action for you:** open the notebook in Fabric and fill in the exact transformation steps here — join keys, dedup logic, any data quality rules — so this section reflects what the code actually does rather than what it's likely to do.

## Data Sources

- **Olist Brazilian E-Commerce Public Dataset** — `olist_customers_dataset.csv`, `olist_orders_dataset.csv`, `olist_order_items_dataset.csv`, `olist_order_payments_dataset.csv`, `olist_order_reviews_dataset.csv`, `olist_products_dataset.csv`, `product_category_name_translation.csv`. A well-known open dataset commonly used for e-commerce analytics demos.
- **Custom sample files** — `customers.csv`, `orders.csv`, `payments.csv`, `support_tickets.csv`, `web_activities.csv`. Small files used to drive/test the HTTP-based copy pipeline.

## Repository Structure

| File | Purpose |
|---|---|
| `ecom_copy_activity.json` | Fabric/ADF pipeline definition — Bronze ingestion via ForEach + Copy Activity |
| `manifest.json` | Auto-generated Fabric item manifest (pipeline canvas thumbnail + linked service requirements) |
| `ecom_bronze_to_silver.ipynb` | PySpark notebook — Bronze → Silver transformation |
| `ecom_bronze_to_silver (1).ipynb` | Appears to be a duplicate/backup of the above — consider removing |
| `olist_*.csv`, `product_category_name_translation.csv` | Olist public dataset source files |
| `customers.csv`, `orders.csv`, `payments.csv`, `support_tickets.csv`, `web_activities.csv` | Small sample files used by the copy pipeline |
| `part-00000-...snappy.parquet` | Sample Bronze-layer output file (Parquet, Snappy-compressed) |
| `.DS_Store` | macOS system file — safe to delete and add to `.gitignore` |

## Current Scope / Limitations

- Only **Bronze** and **Bronze → Silver** stages exist in this repo — there's no Gold-layer notebook yet. If your resume or portfolio describes a full Bronze/Silver/Gold Medallion pipeline, either add a Gold notebook here or scope the claim to match what's actually in the repo.
- No `requirements.txt` or setup instructions were present — added a basic outline below.
- `.DS_Store` and the duplicate notebook are repo clutter worth cleaning up before sharing this publicly (e.g., with recruiters).

## How to Reproduce

1. Create a Fabric workspace and a Lakehouse named `ecommerce_lakehouse`.
2. Import `ecom_copy_activity.json` as a Data Factory pipeline (or rebuild it in the Fabric UI using the structure above).
3. Create an HTTP linked service (`finalcon shelkerohan99`) pointing at your source GitHub repo's raw file URLs.
4. Run the pipeline — this lands Parquet files in the Bronze folder.
5. Open `ecom_bronze_to_silver.ipynb` in a Fabric notebook, attach the Lakehouse, and run the cells to produce the Silver output.

