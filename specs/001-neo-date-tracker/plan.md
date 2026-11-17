# Implementation Plan: NASA NEO Tracker Date Lookup

**Branch**: `001-neo-date-tracker` (planned) | **Date**: 2026-11-17 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `specs/001-neo-date-tracker/spec.md` and the
user-provided technical stack and architecture.

## Summary

Build a Python desktop GUI for looking up near-Earth objects for a user-selected date
through NASA NeoWs. Separate the GUI, date validation, HTTP client, response parser,
service orchestration, and JSON cache. Run blocking HTTP work away from the GUI event
loop. Valid non-empty results are cached locally; empty results and failures use the
last cached non-empty day when available. Unit tests use fixtures and mocked HTTP calls,
with no real network access.

## Technical Context

**Language/Version**: Python 3.11+

**Primary Dependencies**: customtkinter, requests, python-dotenv; pytest for tests.

**Storage**: `cache/last_response.json` for the latest successful non-empty result;
`.env` for the local NASA API key. Both `cache/` and `.env` are local-only and MUST be
excluded from version control.

**Testing**: pytest unit tests with fixture JSON and mocked `requests`; no live NASA calls.

**Target Platform**: Desktop platforms supported by Python and customtkinter; validate
locally on Windows.

**Project Type**: Desktop GUI application.

**Performance Goals**: Keep the GUI responsive while a request is in progress. HTTP
requests use a finite `(connect=3s, read=10s)` timeout; no external-service latency SLA
is assumed. The read timeout is not an overall download deadline.

**Constraints**: API requests use NASA NeoWs Feed with the selected date as both
`start_date` and `end_date`. The API key comes from `NASA_API_KEY` in the environment or
`.env`; `.env` MUST stay untracked. No request is made for invalid dates. Handle network,
service, rate-limit, parse, missing-cache, and corrupted-cache conditions without
terminating the application. Do not overwrite cached non-empty data with an empty result
or a failed request. Keep runtime dependencies to the user-specified stack.

**Scale/Scope**: One user, one selected calendar day per lookup, one GUI window, and a
local cache containing the latest successful non-empty day.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **GUI-only product**: PASS. The application is a customtkinter desktop GUI with no
  console workflow.
- **Resilient user outcomes**: PASS. Expected validation, network, API, rate-limit,
  parsing, and cache problems are represented as clear UI states; no expected failure
  terminates the application.
- **Mandatory unit tests**: PASS. pytest tests cover validation, parsing, service
  outcomes, cache persistence, and failures using fixtures and mocked HTTP calls.
- **Persistent last-success cache**: PASS. Persist non-empty successful results and retain
  them across restart; empty results and failed requests do not replace the fallback.
- **Secret hygiene and minimal dependencies**: PASS. Read the API key from environment or
  `.env`, ignore `.env`, provide only a placeholder `.env.example`, and use only the
  specified runtime/test dependencies.

No constitutional exceptions are required. For this plan, a successful cached lookup
means a valid response containing asteroid results; an empty result is the specified
no-data outcome and preserves the previous successful cache.

## Project Structure

### Documentation (this feature)

```text
specs/001-neo-date-tracker/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   ├── nasa-neows-feed.md
│   └── service-result.md
└── tasks.md                 # Created by /speckit-tasks
```

### Source Code (repository root)

```text
main.py
src/
└── neo_tracker/
    ├── __init__.py
    ├── api_client.py
    ├── cache.py
    ├── gui.py
    ├── parser.py
    ├── service.py
    └── validator.py
tests/
├── conftest.py
├── fixtures/
│   ├── neows_sample.json
│   ├── neows_empty.json
│   └── neows_malformed.json
└── unit/
    ├── test_api_client.py
    ├── test_cache.py
    ├── test_gui.py
    ├── test_parser.py
    ├── test_service.py
    └── test_validator.py
requirements.txt
README.md
.env.example
.gitignore
```

**Structure Decision**: Use the user-specified `src/neo_tracker` package with a root
`main.py`, and pytest unit tests under `tests/unit`. The launcher and test setup expose
the `src` package path so the documented `python main.py` and `python -m pytest`
commands work without adding a packaging dependency. Keep domain parsing and date
validation separate from GUI code; the service coordinates API, parser, and cache.

## Post-Design Constitution Check

- GUI-only surface and background request processing: PASS.
- Expected invalid-input, network, API, rate-limit, parse, and cache failures have explicit
  user-facing outcomes without expected application termination: PASS.
- Unit tests cover the domain, service, cache, API client, and GUI state without live
  network access: PASS.
- Successful non-empty results persist across restart; empty results and failures retain
  the last successful cached day: PASS.
- API credentials and local cache data are excluded from version control, and the plan
  adds no dependencies beyond the user-specified set: PASS.

No constitutional exceptions are required.

## Complexity Tracking

No constitutional violations or additional architectural projects are planned.
