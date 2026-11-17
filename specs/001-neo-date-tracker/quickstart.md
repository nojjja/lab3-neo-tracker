# Quickstart: NASA NEO Tracker Date Lookup

## Prerequisites

- Python 3.11 or newer.
- A NASA API key for live lookups.
- Network access for live NASA NeoWs requests.

## Install

From the repository root, create and activate a virtual environment, then install the
declared dependencies:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

```powershell
Copy-Item .env.example .env
```

Set `NASA_API_KEY` in `.env` to the personal key. The `.env` file is ignored by version
control and MUST NOT be committed. An already configured process environment variable
takes precedence over the `.env` value.

## Run the GUI

```powershell
python main.py
```

Enter a real date in `YYYY-MM-DD` form and select **Get data**. Verify that results show
diameter, velocity, miss distance, hazardous status, NASA NeoWs as the source, and the
response timestamp.

## Run Automated Tests

```powershell
python -m pytest
```

Expected result: all unit tests pass without making real network requests. API-client
tests use mocked HTTP responses; parser, service, date validation, and cache tests use
local fixtures or temporary files.

## End-to-End Validation Scenarios

1. Enter an empty value, letters, a malformed date, and an impossible date such as
   `2026-02-31`. For each, confirm the button is disabled, an explanatory validation
   message is visible, and no request is sent.
2. With a configured key and network, look up a date with results and verify every listed
   asteroid's fields, units, source, and response timestamp.
3. After a successful lookup, request a date for which NASA returns no results. Confirm
   the last cached day is shown with a warning and the cached date is visible.
4. Restart after a successful lookup, then make the network unavailable and request a
   valid date. Confirm the application stays open and offers the cached results, date, and
   original timestamp. If no cache is present, confirm a clear no-cache message instead.
