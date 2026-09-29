# PHANTOMS AI — Week 2

## Backend Engineering Capstone

A small backend integration project that connects an external API, a SQLite persistence layer, and a Flask webhook into one reproducible workflow.

**Core flow**

`OpenWeather API → SQLite → Webhook → SQLite`

## What I built

- External API client for OpenWeather
- SQLite persistence layer
- Flask webhook receiver
- API → database → webhook pipeline
- Automated tests for project structure and database schema
- Appointment-booking database design
- SQL JOIN and aggregation debugging

## Repository structure

```text
phantoms-ai-week2/
├── .github/
│   └── workflows/
│       └── tests.yml
├── data/
│   └── store.db              # generated locally, ignored by Git
├── src/
│   ├── api.py                # OpenWeather API client
│   ├── database.py           # SQLite storage layer
│   ├── pipeline.py           # API → DB → webhook pipeline
│   └── webhook.py            # Flask webhook receiver
├── tests/
│   └── test_project.py       # project and database checks
├── .env.example              # local configuration template
├── .gitignore
├── main.py
├── requirements.txt
├── db_design.md
└── debug_solution.sql
```

## Setup

### 1. Create a virtual environment

```bash
python -m venv .venv
```

Activate it on Linux/macOS:

```bash
source .venv/bin/activate
```

Activate it on Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Configure environment variables

Copy `.env.example` to `.env` and provide your local values.

Required variables:

- `OPENWEATHER_API_KEY`
- `WEBHOOK_URL`

Optional controls:

- `PIPELINE_CYCLES` — default: `3`
- `PIPELINE_INTERVAL` — default: `30` seconds

The application reads secrets from environment variables. Real credentials must never be committed.

## Run the webhook

Start the Flask receiver first:

```bash
python -m src.webhook
```

The local endpoint is:

```text
http://127.0.0.1:5000/webhook
```

For a public webhook endpoint, replace `WEBHOOK_URL` with the current endpoint.

## Run the pipeline

From the repository root:

```bash
python main.py
```

Each cycle:

1. Fetches weather data from OpenWeather
2. Stores the response in SQLite
3. Sends the JSON payload to the webhook
4. Stores the received payload in SQLite

## Run tests

```bash
pytest -q
```

The repository also includes a GitHub Actions workflow that runs the test suite automatically on pushes and pull requests targeting `main`.

## Database design

[`db_design.md`](db_design.md) documents a compact appointment-booking schema containing:

- Patients
- Doctors
- Appointments
- Appointment status
- Start and end times
- Preserved cancellation history

The design intentionally stays within the scope of a backend foundations exercise rather than attempting to model a complete medical platform.

## SQL debugging

[`debug_solution.sql`](debug_solution.sql) demonstrates two important SQL concepts:

1. Keeping a `LEFT JOIN` effective by placing the right-table filter in the `ON` clause.
2. Grouping by every selected non-aggregated column.

It also uses `COALESCE` so customers with no completed orders receive a total of zero.

## Engineering decisions

- Configuration and secrets are separated from source code.
- Generated SQLite data is ignored by Git.
- Database connections use context managers for reliable cleanup.
- Timestamps are stored in UTC.
- SQL writes use parameterized queries.
- HTTP requests use explicit timeouts.
- Runtime configuration is validated before the pipeline starts.
- The project is intentionally small so each backend component remains easy to inspect.

## Security considerations

This is a learning project, not a production webhook service.

The webhook currently accepts JSON without authentication or signature verification. A production implementation should add authentication, request validation, rate limiting, structured logging, and appropriate network controls.

Never commit API keys, webhook secrets, `.env` files, or other credentials.

## Learning direction

This project is part of my progression from software and backend foundations toward **cybersecurity, AI security, and eventually LLM Red Teaming**.

> Build it. Break it. Understand it. Document it.
