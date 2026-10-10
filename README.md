# Employee Attendance & Analytics API

A FastAPI + MongoDB service implementing employee attendance management and analytics pipelines conforming strictly to `openapi.yaml` and business rules R1–R10.

## Architecture

The project follows a clean layered architecture self-contained within `app/main.py`:
```
api / app -> router -> middlewares -> controllers -> services -> repository -> models
```

- **Models:** Pydantic v2 schemas strictly validating inputs and epoch milliseconds.
- **Repositories:** MongoDB query execution and aggregation pipelines.
- **Services:** Pure domain logic for R1–R10 (IST timezone truncation, overnight shifts, 10-minute grace period, $\ge 30$ min overtime, Decimal HALF-UP rounding, Mon–Fri working days, and history audit trails).
- **Controllers & Middlewares:** Request handling, CORS, malformed JSON detection, and error responses.
- **Indexes:** Created idempotently on application startup, guaranteeing `IXSCAN` and zero `COLLSCAN`.

## Requirements & Local Setup

- Python 3.11+
- MongoDB 6.0+ (7.0 recommended)

```bash
# 1. Environment & Dependencies
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# 2. Configure Environment (optional, defaults to localhost:27017 and attendance_db)
export MONGO_URI="mongodb://localhost:27017"
export MONGO_DB="attendance_db"

# 3. Seed Sample Data
python sample_seed.py

# 4. Start the Application
uvicorn app.main:app --port 8000
```

## Health Check
```bash
curl http://localhost:8000/health
# Returns 200 {"status": "ok"} when MongoDB is reachable
```
