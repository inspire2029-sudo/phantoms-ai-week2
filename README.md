# PHANTOMS AI Team — Week 2 Capstone

This repository contains the Week 2 Backend Foundations capstone and the mini integration work.

## Project goal

The project connects the Week 2 backend pieces into one simple flow:

**External API → SQLite database → Webhook**

The same project also documents an appointment booking database design and debugs a SQL aggregation query.

## Repository structure

```text
phantoms-ai-week2/
├── data/
│   └── store.db              # generated SQLite database
├── src/
│   ├── api.py                # OpenWeather API client
│   ├── database.py           # SQLite storage layer
│   ├── pipeline.py           # API → DB → webhook pipeline
│   └── webhook.py            # Flask webhook receiver
├── tests/
│   └── test_project.py       # basic structure and database checks
├── main.py                   # pipeline entry point
├── requirements.txt
├── db_design.md              # appointment booking schema
└── debug_solution.sql        # corrected SQL query and explanation
```

## Setup

Create a virtual environment if desired, then install dependencies:

```bash
pip install -r requirements.txt
```

Set the required environment variables.

### OpenWeather API key

```bash
export OPENWEATHER_API_KEY="YOUR_OPENWEATHER_API_KEY"
```

On Windows PowerShell:

```powershell
$env:OPENWEATHER_API_KEY="YOUR_OPENWEATHER_API_KEY"
```

### Webhook URL

Start the Flask webhook receiver first:

```bash
python -m src.webhook
```

Then set:

```bash
export WEBHOOK_URL="http://127.0.0.1:5000/webhook"
```

For a public webhook such as an ngrok endpoint, use the current endpoint instead. Public ngrok URLs can change when a session ends.

## Run the integrated pipeline

From the repository root:

```bash
python main.py
```

By default it runs 3 cycles with a 30-second interval.

Optional controls:

```bash
export PIPELINE_CYCLES="3"
export PIPELINE_INTERVAL="30"
```

The pipeline performs:

1. Fetch weather data from OpenWeather.
2. Store the API response in SQLite table `api_logs`.
3. Send the same JSON payload to the webhook.
4. The webhook stores the received payload in `webhook_logs`.

## Run the tests

```bash
pytest -q
```

The tests verify the required project structure and the SQLite tables used by the integration.

## W2D5 database design

`db_design.md` documents an appointment booking system with:

- multiple patients
- multiple doctors
- one patient and one doctor per appointment
- appointment start and end times
- explicit appointment status
- preserved cancelled appointments

Cancelled bookings are retained by changing their status rather than deleting the appointment row. This keeps historical booking data available.

## W2D5 SQL debugging

`debug_solution.sql` fixes two issues in the supplied query:

1. A condition on the right-side table was placed in `WHERE`, which removes customers without matching completed orders. The condition is moved into the `LEFT JOIN ... ON` clause.
2. `customer_name` is selected but was not included in `GROUP BY`. The corrected query groups by both customer ID and customer name.

`COALESCE(SUM(...), 0)` is used so customers with no completed orders receive a total of zero.

## Engineering decisions

- Secrets are read from environment variables and are not committed to the repository.
- SQLite is used because the capstone needs a lightweight local database.
- The database layer is separated from the API, webhook, and pipeline logic.
- The webhook accepts JSON through POST only.
- Database writes use parameterized SQL.
- The pipeline fails explicitly when required configuration is missing or an HTTP request fails.
- The appointment schema is intentionally small and does not attempt to model authentication, payments, prescriptions, or full medical records.

## Source references

The task research also used the supplied database and SQL JOIN tutorials:

- https://www.youtube.com/watch?v=qCIFuoN32cM
- https://www.youtube.com/watch?v=8grUQO38J6A

## Security note

Do not commit an actual API key, webhook secret, or other credential. Use environment variables or a local secret manager instead.
