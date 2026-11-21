# Plan

Each phase ends with a checkpoint. Don't move on until the checkpoint passes.
Small steps. One concept at a time.

## Phase 0: Setup (1 session)
- Python venv, Docker Desktop running, Postgres container via `docker-compose.yml`
- `.env` holds DB user and password; `docker-compose.yml` reads from it
- Learn: what a container is, what a volume is (storage that survives the container restarting)
- Checkpoint: `psql` connects to the container using values from `.env`, nothing hard-coded

## Phase 1: Extract (2 sessions)
- `ingest/fx.py`: fetch ZAR rates for a date range from Frankfurter
- `ingest/weather.py`: fetch Durban daily weather from Open-Meteo (lat -29.86, lon 31.03)
- Rate limiting: max N calls per second, retry with exponential backoff (wait 1s, 2s, 4s...) on 429 and 5xx
- Timeouts on every request
- Save the raw JSON untouched to `data/raw/<source>/<date>.json` (the "landing zone")
- Learn: idempotency (running the same job twice gives the same result, no duplicates)
- Checkpoint: run each script twice for the same date, no duplicate files or rows

## Phase 2: Validate and Load (2 sessions)
- Pydantic models for each source: types, ranges (rate > 0, rain >= 0, date not in future)
- Good rows go to `raw.fx_rates` and `raw.weather_daily`; bad rows go to `raw.rejects` with the reason
- Use upserts (INSERT ... ON CONFLICT) so reloads don't duplicate
- Learn: schema-on-write vs schema-on-read, why you keep raw data
- Checkpoint: feed a hand-broken JSON file in, it lands in `raw.rejects` with a clear reason

## Phase 3: Model with dbt (3 sessions)
- staging models: clean names and types (`stg_fx_rates`, `stg_weather`)
- star schema: `dim_date`, `dim_currency`, `fact_fx_daily`, `fact_weather_daily`
- marts: `mart_fx_monthly` (avg, min, max, volatility), `mart_rain_monthly`
- dbt tests: not_null, unique, relationships, accepted_values
- Learn: fact vs dimension tables, grain (what one row means), incremental models
- Checkpoint: `dbt build` passes all tests; you can explain the grain of every fact table out loud

## Phase 4: Orchestrate with Airflow (2 sessions)
- One DAG (a list of steps with an order): extract -> validate/load -> dbt build
- Daily schedule, retries, backfill (re-running past dates)
- Learn: DAGs, task dependencies, why tasks must be idempotent
- Checkpoint: backfill the last 90 days in one command, data matches a manual run

## Phase 5: Quality and Monitoring (1 session)
- Freshness check: alert if newest row is older than 2 days
- Row count checks between raw and fact tables
- Checkpoint: break a source on purpose, see the failure surface clearly

## Stretch
- Phase 6: PySpark. Backfill weather from 1940 to today and compute the same monthly mart in Spark. Compare speed vs SQL
- Phase 7: Cloud. Land raw files in S3 using your Terraform-AWS-Lab skills (OIDC, no long-lived keys)
- Phase 8: Write a short case study for the CV: problem, design, numbers, what broke

## Interview drill (every phase)
At the end of each phase, explain what you built for 2 minutes out loud with no notes:
what it does, why this design, what would break at 100x the data.
