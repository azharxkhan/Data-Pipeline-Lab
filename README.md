# Data-Pipeline-Lab

A hands-on data engineering project: extract, validate, load, model and orchestrate public data end to end.

## What it does
Pulls two free public data sources every day, checks them, stores them, and models them for analysis:

| Source | What | Key? |
|---|---|---|
| Frankfurter (api.frankfurter.dev) | ZAR exchange rates vs USD, EUR, GBP | No key |
| Open-Meteo (api.open-meteo.com) | Durban daily weather (rain, temp) | No key, non-commercial, ~10k calls/day fair use, CC BY 4.0 attribution |

Questions the finished project answers:
1. How has ZAR moved against USD, EUR and GBP, month by month?
2. Which months are most volatile (prices swing the most)?
3. How does Durban rainfall change across the year?

## Stack (one new tool per phase)
Python, PostgreSQL in Docker, Pydantic (data checks), dbt (SQL modelling), Airflow (scheduling), then PySpark and AWS S3 as stretch goals.

## Rules
- Secrets live in `.env` only. `.env` is never committed. Use `.env.example` as the template.
- Every API call is rate limited and retried with backoff.
- Every record is validated before it is stored. Bad rows go to a reject table, never silently dropped.
- Data attribution: Weather data by Open-Meteo.com (CC BY 4.0). FX data via Frankfurter (ECB and other central banks).

See `docs/PLAN.md` for phases and `docs/PROGRESS.md` for my log.
