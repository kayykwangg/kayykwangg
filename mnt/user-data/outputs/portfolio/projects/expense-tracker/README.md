# Expense Tracker

Full-stack personal finance app with a Django REST backend and React + Recharts frontend.

## Features

- **Transaction CRUD** with categories and tags
- **Recurring transactions** (weekly / monthly / yearly)
- **Budget alerts** — notified when you exceed a category limit
- **CSV import** — paste exports from your bank
- **Analytics** — monthly breakdown, trend charts, category pie charts
- **Dark mode** support

## Stack

| Layer | Tech |
|---|---|
| Backend | Django 5 + Django REST Framework |
| Database | PostgreSQL |
| Frontend | React + TypeScript |
| Charts | Recharts |
| Styling | Tailwind CSS |
| Auth | JWT via djangorestframework-simplejwt |

## API highlights

```
GET    /api/transactions/          # paginated, filterable by date & category
POST   /api/transactions/          # create transaction
GET    /api/transactions/summary/  # monthly aggregates for charts
POST   /api/transactions/import/   # CSV upload
GET    /api/budgets/               # budget limits per category
PUT    /api/budgets/{id}/          # update a budget limit
```

## CSV import format

```csv
date,description,amount,category
2026-01-15,Coffee,4.50,Food & Drink
2026-01-16,Spotify,9.99,Subscriptions
```

## Running locally

```bash
# Backend
cd backend
pip install -r requirements.txt
cp .env.example .env
python manage.py migrate
python manage.py runserver

# Frontend
cd frontend
npm install
npm run dev
```

## Key implementation notes

**Recurring transactions** are generated lazily: on each request to `/api/transactions/`, the backend checks if any recurring rules have un-generated instances up to today, creates them, then returns the full list. This avoids a scheduled task dependency for simple deployments.

**Budget alerts** are computed at query time and returned as a `budget_status` field on each category summary — the frontend shows a warning badge when `spent / limit > 0.9`.
