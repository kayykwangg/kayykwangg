# URL Shortener

High-throughput URL shortener with click analytics. FastAPI + Redis for sub-millisecond redirects.

## Features

- **Sub-millisecond redirects** — resolved from Redis cache first, Postgres fallback
- **Click analytics** — country, referrer, device type, timestamp
- **Custom slugs** — or auto-generated 6-char alphanumeric codes
- **Link expiry** — set optional TTL on any link
- **Public API** — rate-limited per IP (100 requests / 15 min)
- **Dashboard** — click charts, top links, geography breakdown

## Stack

| Layer | Tech |
|---|---|
| Framework | FastAPI |
| Cache | Redis (redirect lookup + rate limiting) |
| Database | PostgreSQL (link storage + analytics) |
| GeoIP | MaxMind GeoLite2 (local) |
| Deploy | Docker + Nginx |

## Quick start

```bash
docker compose up --build
```

Shorten a URL:
```bash
curl -X POST http://localhost:8000/api/shorten \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com/very/long/path"}'
# → {"short_url": "http://localhost:8000/abc123", "slug": "abc123"}
```

## Redirect flow

```
Browser → Nginx → FastAPI
                    │
              Redis hit? ──YES──► 301 redirect (< 1ms)
                    │
                   NO
                    │
            Postgres lookup
                    │
              Cache in Redis (TTL = 24h)
                    │
              Log click async (Celery)
                    │
              301 redirect
```

## Rate limiting

Rate limiting is implemented as a Redis sliding window counter. Each IP gets a key `ratelimit:{ip}:{minute_bucket}` with a 15-minute TTL. When the count exceeds the limit, the API returns `429 Too Many Requests` with a `Retry-After` header.

## Analytics schema

```sql
CREATE TABLE clicks (
    id          BIGSERIAL PRIMARY KEY,
    slug        TEXT NOT NULL,
    clicked_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
    country     TEXT,
    referrer    TEXT,
    device      TEXT,  -- mobile | desktop | bot
    user_agent  TEXT
);

CREATE INDEX clicks_slug_idx ON clicks (slug, clicked_at DESC);
```
