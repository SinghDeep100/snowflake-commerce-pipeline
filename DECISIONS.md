# Design Decisions

A running log of the key choices in this pipeline and the reasoning behind each.

### 1. Sample data over external S3 (for the bulk backfill)
Removes credential friction while building and learning the loading mechanics.The production form is an external stage backed by a storage integration on S3 — this project uses Snowflake sample data to stand in for that historical source.

### 2. ON_ERROR = CONTINUE for the live feed, ABORT for the financial backfill
One malformed clickstream record should not kill an entire batch — but every skipped row is surfaced via VALIDATE() so nothing is silently dropped. A bad row in a financial backfill, by contrast, must halt the load and be investigated.

### 3. Raw JSON preserved intact in a VARIANT column
The RAW layer is immutable. Storing the original payload exactly as received means downstream logic can be reprocessed without re-ingesting from the source.

### 4. Separate RAW / STAGING / MARTS schemas
RAW is immutable landing, STAGING is rebuildable transformation, MARTS serves consumers. This separation means STAGING can be dropped and rebuilt at any time without ever touching source data.
