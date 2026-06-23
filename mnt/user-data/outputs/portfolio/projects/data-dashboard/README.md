# Weather Data Pipeline & Dashboard

ETL pipeline pulling from OpenWeather API, stored in PostgreSQL, served via FastAPI, visualised in React.

## Features

- **ETL pipeline** — fetches hourly weather for configurable cities, stores in Postgres
- **Pandas aggregations** — daily highs/lows, rolling averages, anomaly detection
- **FastAPI data API** — time-series endpoints with date range filtering
- **React dashboard** — Recharts time-series, city search, historical comparison
- **Alembic migrations** — schema versioned and reproducible
- **Scheduled ingestion** — via APScheduler (no external task queue needed)

## Stack

| Layer | Tech |
|---|---|
| Ingestion | httpx (async HTTP) + APScheduler |
| Processing | Pandas + NumPy |
| Storage | PostgreSQL + SQLAlchemy + Alembic |
| API | FastAPI |
| Frontend | React + Recharts + Tailwind |

## Setup

```bash
# 1. Copy env and add your OpenWeather API key
cp .env.example .env

# 2. Start Postgres
docker compose up db -d

# 3. Run migrations
alembic upgrade head

# 4. Start the API (ingestion scheduler runs inside)
uvicorn app.main:app --reload

# 5. Frontend
cd frontend && npm install && npm run dev
```

## Pipeline architecture

```
OpenWeather API
      │ (httpx, every hour)
      ▼
  Ingestor (app/pipeline/ingest.py)
      │ raw JSON → WeatherReading ORM model
      ▼
  PostgreSQL  weather_readings table
      │
  Aggregator (app/pipeline/aggregate.py)
      │ Pandas: resample, rolling mean, z-score anomaly flag
      ▼
  daily_summaries table
      │
  FastAPI /api/weather/* endpoints
      │
  React dashboard
```

## Key endpoints

```
GET /api/weather/cities           # list tracked cities
GET /api/weather/{city}/hourly    # raw readings, ?from=&to=
GET /api/weather/{city}/daily     # daily aggregates
GET /api/weather/{city}/anomalies # days flagged as anomalous
GET /api/weather/compare          # side-by-side city comparison
```

## Anomaly detection

Daily temperature readings are flagged as anomalous when they fall outside 2 standard deviations of the trailing 30-day rolling mean. This is a simple but interpretable method that requires no ML infrastructure.

```python
df["rolling_mean"] = df["temp_max"].rolling(30, min_periods=7).mean()
df["rolling_std"]  = df["temp_max"].rolling(30, min_periods=7).std()
df["anomaly"] = (df["temp_max"] - df["rolling_mean"]).abs() > 2 * df["rolling_std"]
```
