# Equinor & Brent Market Data Pipeline

Small Databricks data engineering project comparing Equinor (`EQNR.OL`) with Brent crude oil (`BZ=F`) using `yfinance`.

## Architecture

`yfinance → Bronze → Silver → Gold → Databricks BI Dashboard`

The project uses two grains: **5-minute data** for recent intraday analysis and **daily data** for longer-term historical analysis.

## Data Model

| Layer  | 5-minute           | Daily                 |
| ------ | ------------------ | --------------------- |
| Bronze | `bronze.market_5m` | `bronze.market_daily` |
| Silver | `silver.market_5m` | `silver.market_daily` |
| Gold   | `gold.market_5m`   | `gold.market_daily`   |

Bronze stores source-level market data and metadata. Silver contains cleaned and standardized data. Gold contains business-ready measures such as returns, normalized performance, rolling correlation, and rolling volatility.

Returns and volatility are stored as decimal ratios, e.g. `0.0123 = 1.23%`, and formatted as percentages in the dashboard.

## Processing

**Intraday Job:** `5m ingestion → Silver 5m → Gold 5m`
Runs every 30 minutes.

**Daily Job:** `Daily ingestion → Silver daily → Gold daily`
Runs once per trading day. Historical daily data is seeded with an initial backfill, then maintained incrementally with Delta `MERGE`.

Silver and Gold are rebuilt with overwrite because the datasets are small.

## Dashboard

The AI/BI dashboard has separate intraday and historical views with:

* Normalized EQNR vs Brent performance
* Rolling correlation
* Rolling volatility
* Price and return KPI cards
* Date and date-range filtering

`EQNR.OL` is quoted in NOK while Brent (`BZ=F`) is quoted in USD. Normalized performance and returns are therefore used when comparing relative movement.

## Stack

Databricks Free Edition · PySpark · Delta Lake · Unity Catalog · Lakeflow Jobs · AI/BI Dashboards · Python · `yfinance`