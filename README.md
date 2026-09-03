# Expense Tracker (FastAPI)
 Expense Tracker: FastAPI backend with SQLite, tests, Docker, and GitHub Actions CI.

## One-line summary
A RESTful Expense Tracker API built with FastAPI and SQLModel (SQLite) that supports user auth, CRUD for expenses & categories, CSV export, and monthly reporting — easy to run locally or in Docker for demos.

## Table of contents
- Project highlights
- Tech stack
- Data schema (clear sample data)
- API endpoints (method, path, request, response examples)
- Quick start (local & Docker)
- Tests & CI
- Sample curl commands
- Resume-ready bullets
- License & contribution

## Project highlights
- Authenticated users (JWT) can create/read/update/delete expenses.
- Categorize expenses and view monthly summaries and totals.
- CSV export for user expenses and endpoint for simple analytics (group by).
- Swagger UI (/docs) and built-in OpenAPI.
- Containerized (Docker) and CI via GitHub Actions running pytest.

## Tech stack
- Python 3.11+
- FastAPI, SQLModel (SQLAlchemy under the hood)
- SQLite by default (file-based); optional Postgres in Docker
- Pytest for tests
- Docker & docker-compose
- GitHub Actions for CI

## Data schema and sample data

Database tables (SQLModel / SQL/JSON schema):

- users
  - id: int (PK)
  - email: string (unique, indexed)
  - full_name: string | null
  - hashed_password: string
  - is_active: bool
  - created_at: datetime

- categories
  - id: int (PK)
  - name: string (unique per user optional)
  - color: string (hex, optional)
  - user_id: int (FK -> users.id)
  - created_at: datetime

- expenses
  - id: int (PK)
  - user_id: int (FK -> users.id)
  - category_id: int | null (FK -> categories.id)
  - amount: decimal (stored as numeric)
  - currency: string (e.g., "USD")
  - description: text | null
  - spent_at: date/datetime
  - created_at: datetime
  - updated_at: datetime

Sample CSV for import/export (expenses.csv):
date,amount,currency,category,description
2026-08-20,12.50,USD,Food,"Lunch at cafe"
2026-08-21,40.00,USD,Transport,"Monthly bus pass"

Sample JSON record (expense):
{
  "id": 12,
  "user_id": 3,
  "category_id": 2,
  "amount": 12.5,
  "currency": "USD",
  "description": "Lunch at cafe",
  "spent_at": "2026-08-20T12:00:00Z",
  "created_at": "2026-08-20T12:05:00Z",
  "updated_at": "2026-08-20T12:05:00Z"
}

Sample JSON (user registration):
{
  "email": "alice@example.com",
  "full_name": "Alice Example",
  "password": "P@ssw0rd!"
}

Sample JSON (login -> returns access token):
Request:
{
  "username": "alice@example.com",
  "password": "P@ssw0rd!"
}
Response:
{
  "access_token": "eyJhbGciOiJ...",
  "token_type": "bearer"
}

## API endpoints (main ones)

Auth:
- POST /auth/register
  - Body: { "email", "full_name?", "password" }
  - Response: created user (no password)
- POST /auth/login
  - Form: username (email), password
  - Response: { access_token, token_type }

Users:
- GET /users/me
  - Header: Authorization: Bearer <token>
  - Response: user profile

Categories:
- GET /categories
  - Returns list of user's categories
- POST /categories
  - Body: { "name", "color?" }
  - Creates category
- PUT /categories/{id}
  - Update category
- DELETE /categories/{id}

Expenses:
- GET /expenses
  - Query params: start_date, end_date, category_id, page, size
  - Returns paginated list
- POST /expenses
  - Body: { "amount", "currency", "category_id?", "description?", "spent_at" }
  - Creates an expense
- GET /expenses/{id}
  - Returns a single expense (user-scoped)
- PUT /expenses/{id}
  - Update expense
- DELETE /expenses/{id}

Reporting / export:
- GET /reports/monthly?year=2026&month=8
  - Returns totals grouped by category and overall total for given month
- GET /expenses/export?format=csv
  - Returns CSV file of user's expenses (respecting filters)

Errors follow FastAPI default: JSON problem detail with status codes (401 for unauthorized, 403 forbidden, 422 validation errors, etc).

## Quick start (local)

Prerequisites: Python 3.11+, pip

1. Clone
   git clone https://github.com/venkat67760-oss/expense-tracker-fastapi.git
   cd expense-tracker-fastapi

2. (Optional) Create virtualenv
   python -m venv .venv
   source .venv/bin/activate

3. Install
   pip install -r requirements.txt

4. Environment variables (example .env)
   SECRET_KEY=replace-with-secure-random-string
   ACCESS_TOKEN_EXPIRE_MINUTES=60
   DATABASE_URL=sqlite:///./dev.db

5. Run migrations (if used) or let SQLModel create tables:
   python -m app.db  # script that creates tables or uses alembic

6. Start dev server
   uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

7. Open interactive docs: http://localhost:8000/docs

## Quick start (Docker)

- Build & run:
  docker build -t expense-tracker:latest .
  docker run -e SECRET_KEY=... -p 8000:8000 expense-tracker:latest

Or using docker-compose (if provided)
  docker-compose up --build

## Tests & CI

Run tests locally:
  pytest -q

CI: GitHub Actions workflow runs pytest on push and pull requests. Tests use a temporary SQLite memory DB and TestClient.

## Sample curl commands

Register:
curl -X POST "http://localhost:8000/auth/register" -H "Content-Type: application/json" -d '{"email":"alice@example.com","password":"P@ssw0rd!","full_name":"Alice"}'

Login:
curl -X POST "http://localhost:8000/auth/login" -d "username=alice@example.com&password=P@ssw0rd!" -H "Content-Type: application/x-www-form-urlencoded"

Create expense:
curl -X POST "http://localhost:8000/expenses" -H "Authorization: Bearer <token>" -H "Content-Type: application/json" -d '{"amount": 12.5, "currency": "USD", "description":"Lunch", "spent_at":"2026-08-20T12:00:00Z"}'

Get monthly report:
curl "http://localhost:8000/reports/monthly?year=2026&month=8" -H "Authorization: Bearer <token>"

Export CSV:
curl "http://localhost:8000/expenses/export?format=csv" -H "Authorization: Bearer <token>" -o expenses.csv

## Resume-ready bullets (copy-paste)
- Designed and implemented a RESTful Expense Tracker using FastAPI, SQLModel (SQLite), and JWT authentication; features include CRUD operations, monthly reporting, CSV export, and OpenAPI docs.
- Built automated tests with pytest and configured GitHub Actions CI to run tests on every push; containerized app with Docker for easy deployment.
- Implemented data validation with Pydantic models, pagination for large result sets, and user-scoped resource permissions.

## Contribution & roadmap (short)
Planned enhancements:
- React frontend for a polished demo UI.
- Role-based permissions and multi-currency support with exchange rates.
- PostgreSQL production profile and Alembic migrations.

## License
MIT
