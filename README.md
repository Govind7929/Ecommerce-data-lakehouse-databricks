### E-Commerce Historical Backfill Pipeline (Databricks + PySpark + Genie)

Overview

This project implements a historical data backfill pipeline for an e-commerce business using PySpark on Databricks. Raw daily transaction data (orders, customers, products, brands, categories, pricing, discounts, taxes, and multi-currency sales across web and mobile channels) is ingested and progressively refined through a Medallion Architecture (Bronze → Silver → Gold). The final curated Gold layer powers AI-driven, natural-language dashboards built using Databricks Genie.

How the Data is Organized

All data lives inside a single Databricks catalog called ecommerce, which is split into a few schemas (think of these as "folders" that group related tables together). Here's what each one contains and why it exists.

1. source_data — Landing Zone

This is where raw files first arrive before anything is processed.


raw (volume) — A storage location holding the original, untouched source files (e.g., daily CSV exports of transactions) exactly as they were received, before any transformation. This acts as the single source of truth for backfilling history if anything downstream needs to be reprocessed.


2. bronze — Raw Ingested Data

The Bronze layer is a near-exact copy of the source data, loaded into proper tables so it can be queried with SQL/PySpark, but with little to no cleaning applied. It preserves full history for backfill purposes.


brz_customers — Raw customer records as received from source.
brz_products — Raw product catalog data (SKUs, product IDs, attributes).
brz_brands — Raw brand reference data (brand codes and names).
brz_category — Raw product category reference data.
brz_calendar — Raw date/calendar reference data used for time-based analysis.
brz_order_items — Raw line-item level transaction data: every product purchased in every order, including price, quantity, discounts, taxes, currency, channel, and timestamps.


3. silver — Cleaned & Conformed Data

The Silver layer takes Bronze data and applies cleaning, validation, deduplication, type casting, and standardization (e.g., consistent currency handling, fixing nulls, trimming text). This is the "trusted" layer used as the foundation for business reporting.


slv_customers — Cleaned customer data, ready for joining with other entities.
slv_products — Cleaned product catalog with standardized attributes (color, size, SKU, etc.).
slv_brands — Cleaned, deduplicated brand reference data.
slv_category — Cleaned product category reference data.
slv_calendar — A fully built-out date dimension (day, month, quarter, year, etc.) ready for time-series analysis.
slv_order_items — Cleaned, validated transaction line items with consistent formats and corrected data types.


4. gold — Business-Ready Analytics Layer

The Gold layer is modeled specifically for reporting and dashboards (a star-schema style design with fact and dimension tables). This is the layer that Genie and BI tools query directly.


gld_dim_customers — Customer dimension table: one row per customer with descriptive attributes for slicing reports by customer.
gld_dim_products — Product dimension table: one row per product, including brand, category, color, size, and other descriptive attributes.
gld_dim_date — Date dimension table: enables analysis by day, month, quarter, year, hour of day, etc.
gld_fact_order_items — The core fact table at the order-line-item grain, containing measures like quantity, gross amount, discounts, taxes, and net amount, linked to the dimension tables above via keys.
fact_transactions_denorm — A denormalized (flattened) version of the transaction fact table that joins in customer, product, brand, category, and date attributes directly. This is optimized for quick, ad-hoc querying and for Genie's natural-language Q&A, since it avoids the need for joins.


Other Schemas


default — The default schema automatically created by Databricks for the catalog; not used for project tables.
information_schema — A system-managed schema automatically provided by Databricks containing metadata about the catalog itself (not part of the pipeline).


Architecture Flow

Source Files (CSV)
        |
        v
  source_data.raw   (landing volume)
        |
        v
     bronze         (raw, 1:1 with source)
        |
        v
     silver         (cleaned, validated, conformed)
        |
        v
      gold          (star schema: facts + dimensions + denormalized table)
        |
        v
  Databricks Genie   (AI-generated dashboards & natural-language analytics)


Tech Stack


Databricks — Lakehouse platform (Unity Catalog, Serverless SQL Warehouse)
PySpark — Data transformation and pipeline logic for the historical backfill
Delta Lake — Underlying table format for Bronze/Silver/Gold tables
Databricks Genie — AI/ML-powered dashboards and analytics on top of the Gold layer




