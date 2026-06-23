# TaskFlow API

A production-grade task management REST API built with FastAPI, PostgreSQL, Redis, and Celery.

## Features

- **Async FastAPI** with Pydantic v2 validation
- **JWT authentication** with refresh token rotation
- **Role-based access control** (admin / member)
- **Background jobs** via Celery + Redis (email notifications, cleanup)
- **Rate limiting** per user and per IP
- **OpenAPI docs** auto-generated at `/docs`
- **90%+ test coverage** with Pytest + httpx
- **Docker Compose** for one-command local setup

## Stack

| Layer | Tech |
|---|---|
| Framework | FastAPI 0.111 |
| Database | PostgreSQL 16 + SQLAlchemy 2 + Alembic |
| Cache / Queue | Redis 7 |
| Task queue | Celery 5 |
| Auth | python-jose, passlib[bcrypt] |
| Tests | Pytest, httpx, factory-boy |
| Deploy | Docker Compose |

## Quick start

```bash
git clone https://github.com/alexmorgan-dev/taskflow-api
cd taskflow-api
cp .env.example .env
docker compose up --build
```

API available at `http://localhost:8000` · Docs at `http://localhost:8000/docs`

## Running tests

```bash
docker compose exec api pytest --cov=app --cov-report=term-missing
```

## Project structure

```
app/
├── api/
│   ├── routes/
│   │   ├── auth.py       # /auth/register, /auth/login, /auth/refresh
│   │   ├── tasks.py      # CRUD + status transitions
│   │   └── users.py      # Admin user management
│   └── deps.py           # FastAPI dependencies (get_db, get_current_user)
├── core/
│   ├── config.py         # Settings via pydantic-settings
│   ├── security.py       # JWT helpers
│   └── rate_limit.py     # Redis-backed rate limiter
├── models/               # SQLAlchemy ORM models
├── schemas/              # Pydantic request/response schemas
├── crud/                 # DB access layer
├── worker/
│   └── tasks.py          # Celery tasks
└── tests/
    ├── conftest.py
    ├── test_auth.py
    └── test_tasks.py
```

## Key design decisions

**Async throughout** — all DB calls use SQLAlchemy 2's async session to keep the event loop unblocked under load.

**Separation of schema and model** — Pydantic schemas never inherit from ORM models, keeping serialisation logic independent of persistence.

**Repository pattern** — CRUD functions live in `app/crud/`, so routes stay thin and tests can swap out the DB layer easily.
