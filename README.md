# Snowflake Commerce ELT Pipeline

An end-to-end ELT pipeline on Snowflake that ingests e-commerce order data two ways — a bulk historical backfill and an incremental JSON feed — models it into a star schema, and refreshes incrementally using Streams and Tasks. No external scheduler required; orchestration is native to Snowflake.

## Architecture
Sources                RAW                STAGING              MARTS

─────────              ─────              ───────              ─────

Historical files  →    Landing zone   →   Cleaned + typed  →   Star schema   →  Analytics

(CSV / Parquet)        External +         LATERAL FLATTEN      dim_customer

internal stages    one row per grain    dim_product

Live order feed   →    VARIANT tables     casts, dedup         fact_order_items

(nested JSON)          COPY INTO                ▲

│

Stream + Task (CDC)

tracks changes, MERGEs on schedule

- **RAW** — immutable landing zone. External and internal stages, `COPY INTO`, and `VARIANT` columns that preserve raw JSON exactly as received.
- **STAGING** — cleaned and typed. Nested JSON is shredded into one row per grain using `LATERAL FLATTEN`; values are cast out of `VARIANT` and deduplicated.
- **MARTS** — dimensional star schema: `dim_customer`, `dim_product`,`fact_order_items`.
- **Orchestration** — Streams capture changes on the staging tables; Tasks `MERGE` those changes into the marts on a schedule, only running when there is new data to process.

## What this project demonstrates

- **Semi-structured data handling** — `VARIANT`, dot-notation access, and LATERAL FLATTEN` to turn messy nested JSON into clean relational tables.
- **Loading mechanics with error handling** — file formats, internal and external stages, `COPY INTO`, and resilient ingestion using `ON_ERROR` and `VALIDATE` to surface skipped rows instead of silently dropping data.
- **Native change-data-capture and orchestration** — Streams, Tasks, and the `MERGE`-from-stream pattern to build a self-running incremental pipeline with
  no external scheduler.
- **Performance and cost awareness** — warehouse sizing, auto-suspend, partition pruning, and reading the Query Profile to diagnose slow queries.

## Design approach

This pipeline deliberately ingests data two ways — a one-time bulk backfill for historical orders and an ongoing incremental feed for new orders — because that split mirrors how production systems actually work.The full set of design choices and the reasoning behind each one is documented in [DECISIONS.md](./DECISIONS.md).

## Repository structure
snowflake-commerce-pipeline/

├── README.md            This file

├── DECISIONS.md         Design decisions and the reasoning behind them

├── raw/                 Landing-layer setup, backfill, and file-load scripts

├── staging/             Transformation and JSON flattening

└── marts/               Dimensional model (dimensions + fact tables)

## Status

- [x] RAW layer — schemas, bulk backfill, file ingestion with error handling
- [ ] STAGING layer — JSON flattening and typing
- [ ] MARTS layer — star schema
- [ ] Orchestration — Streams and Tasks
- [ ] Performance pass — Query Profile review, warehouse tuning

## Tech

Snowflake (Streams, Tasks, stages, `COPY INTO`, `LATERAL FLATTEN`, `MERGE`), SQL.
