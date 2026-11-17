---
description: "Implementation tasks for NASA NEO Tracker date lookup"
---

# Tasks: NASA NEO Tracker Date Lookup

**Input**: Design documents from `specs/001-neo-date-tracker/`

**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/`,
and `quickstart.md`.

**Tests**: Unit tests are required by the project constitution. For each story, write
tests before implementation and verify the new tests fail for the expected reason.
All HTTP tests use mocks; no unit test calls NASA.

**Organization**: Tasks are grouped by user story so each increment can be validated
independently.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Tasks use different files and can proceed in parallel after their dependencies.
- **[Story]**: Story label maps to the priorities in `spec.md`.
- Every task names the file or path to create, modify, or validate.

## Path Conventions

- Python package: `src/neo_tracker/`
- Tests and fixtures: `tests/`
- Application entry point and project files: repository root

## Phase 1: Setup

**Purpose**: Create the project and test scaffolding required by all stories.

- [ ] T001 [P] Add `customtkinter`, `requests`, `python-dotenv`, and `pytest` dependencies to `requirements.txt`.
- [ ] T002 [P] Add a placeholder `NASA_API_KEY` entry to `.env.example` and ensure `.gitignore` excludes `.env` and `cache/`.
- [ ] T003 [P] Create the `src/neo_tracker/` package with `src/neo_tracker/__init__.py` and the root `main.py` launcher that exposes `src/` for direct execution.
- [ ] T004 [P] Configure `tests/conftest.py` to expose `src/` for pytest imports and shared test setup.
- [ ] T005 [P] Add representative, empty, and malformed NASA response fixtures at `tests/fixtures/neows_sample.json`, `tests/fixtures/neows_empty.json`, and `tests/fixtures/neows_malformed.json`.

---

## Phase 2: Foundational

**Purpose**: Establish strict date validation before any user story can issue requests.

- [ ] T006 [P] Write failing date-validation unit cases in `tests/unit/test_validator.py` for empty input, letters, wrong format, impossible dates, and a valid date.
- [ ] T007 Implement `src/neo_tracker/validator.py` to enforce `data-model.md` `Requested Date.Value` and `Requested Date.Validation`, and map the valid value using `Requested Date.Request mapping`.

**Checkpoint**: Setup and validation foundation are ready; no invalid date can reach an HTTP request.

---

## Phase 3: User Story 1 - Look Up Asteroids for a Date (Priority: P1) 🎯 MVP

**Goal**: Retrieve a selected day's NASA NeoWs results and show the asteroid details,
source, timestamp, and persisted successful response.

**Independent Test**: With a mocked non-empty NASA response, look up a valid date and
verify that each asteroid's measurements, units, hazardous flag, source, response
timestamp, and saved cache entry are correct.

### Tests for User Story 1

- [ ] T008 [P] [US1] Write failing parser tests in `tests/unit/test_parser.py` for numeric-string conversion, diameter units, missing fields, and malformed JSON.
- [ ] T009 [P] [US1] Write failing request and error-mapping tests in `tests/unit/test_api_client.py`, mocking `requests` and asserting the endpoint, same-day parameters, API key, and timeout.
- [ ] T010 [P] [US1] Write failing JSON persistence tests in `tests/unit/test_cache.py` for cache round-trip, timestamp/source fields, and non-empty records.
- [ ] T011 [P] [US1] Write a failing fresh-lookup test in `tests/unit/test_service.py` for a valid date, parsed results, and cache save.
- [ ] T012 [P] [US1] Write a failing successful-results presentation test in `tests/unit/test_gui.py` using mocked view/root objects rather than a real network or display.

### Implementation for User Story 1

- [ ] T013 [P] [US1] Implement the `Asteroid` dataclass and NASA response parsing in `src/neo_tracker/parser.py`, satisfying `data-model.md` `Asteroid.name`, `Asteroid.diameter_min_km`, `Asteroid.diameter_max_km`, `Asteroid.velocity_km_s`, `Asteroid.miss_distance_km`, `Asteroid.is_potentially_hazardous`, `NASA Feed Response.Selected day`, `NASA Feed Response.Empty state`, `NASA Feed Response.Malformed state`, and `NASA Feed Response.Relationships`.
- [ ] T014 [P] [US1] Implement the NASA GET request and typed configuration/network/API/rate-limit errors in `src/neo_tracker/api_client.py` according to `contracts/nasa-neows-feed.md`; load `NASA_API_KEY` without logging or exposing its value.
- [ ] T015 [US1] Implement JSON persistence and cache validation in `src/neo_tracker/cache.py`, enforcing `data-model.md` `Cached Response.requested_date`, `Cached Response.fetched_at`, `Cached Response.source`, and `Cached Response.asteroids`; missing or corrupt files must produce an explicit cache warning without crashing.
- [ ] T016 [US1] Implement the fresh lookup path in `src/neo_tracker/service.py`, coordinating validation, API client, parser, and cache and returning all `Lookup Result.status`, `Lookup Result.requested_date`, `Lookup Result.data_date`, `Lookup Result.asteroids`, `Lookup Result.source`, `Lookup Result.fetched_at`, and `Lookup Result.warning` fields defined in `data-model.md`.
- [ ] T017 [US1] Implement the CustomTkinter date form and fresh-result display in `src/neo_tracker/gui.py`; use a worker thread plus queue/`after()` so HTTP work never blocks the GUI thread.
- [ ] T018 [US1] Connect the root `main.py` launcher to the GUI entry point in `src/neo_tracker/gui.py`.

**Checkpoint**: User Story 1 can retrieve and display a mocked fresh response and persist it without live-network tests.

---

## Phase 4: User Story 2 - Validate a Date and Handle No Results (Priority: P2)

**Goal**: Keep invalid dates from issuing requests and explain empty-result outcomes,
showing the last cached day when available.

**Independent Test**: Try empty, alphabetic, malformed, and impossible dates and verify
the button stays disabled with a clear message and no request. Then return an empty
response with and without a seeded cache and verify the fallback or no-cache message.

### Tests for User Story 2

- [ ] T019 [P] [US2] Add failing empty-response tests to `tests/unit/test_service.py` for cached fallback, warning text, no-cache `no_data`, and preservation of the prior cache.
- [ ] T020 [P] [US2] Add failing GUI tests to `tests/unit/test_gui.py` for invalid-input messages, disabled button state, zero requests, and empty-result fallback presentation.

### Implementation for User Story 2

- [ ] T021 [P] [US2] Extend `src/neo_tracker/service.py` so `element_count == 0` or an empty selected-day list returns cached fallback when available, otherwise `no_data`, without overwriting the successful cache.
- [ ] T022 [P] [US2] Update `src/neo_tracker/gui.py` to show clear date-validation messages, keep **Get data** disabled until valid, prevent invalid requests, and identify the cached date in empty-result fallback mode.

**Checkpoint**: User Story 2 is independently verifiable with mocked empty responses and a seeded cache.

---

## Phase 5: User Story 3 - Continue with Cached Data (Priority: P3)

**Goal**: Preserve the last successful non-empty result across restarts and offer it after
network, NASA API, rate-limit, or parsing failures.

**Independent Test**: Seed a successful cache, restart the service/cache objects, simulate
each request failure, and verify cached results, their date, source, and original
timestamp are offered with a clear warning. With no cache, verify a clear error state and
that the app remains open.

### Tests for User Story 3

- [ ] T023 [P] [US3] Add failing service tests in `tests/unit/test_service.py` for `NetworkError`, `ApiError`, `RateLimitError`, and `ParseError`, with and without a cached result.
- [ ] T024 [P] [US3] Add failing GUI tests in `tests/unit/test_gui.py` for worker completion, fallback warnings, cached date/timestamp display, and no-cache error presentation.

### Implementation for User Story 3

- [ ] T025 [US3] Extend `src/neo_tracker/service.py` to catch only the defined API/parser/configuration errors, return cached fallback when available, retain the prior cache on failure, and surface cache read/write warnings.
- [ ] T026 [US3] Update `src/neo_tracker/gui.py` to render network/API/parse failure messages and cached fallback metadata on the GUI thread without terminating the application.

**Checkpoint**: User Story 3 works across a service restart and all expected offline/error outcomes.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Document operation and verify the complete feature against its runnable guide.

- [ ] T027 [P] Document installation, API-key setup, GUI launch, and the single-command test run in `README.md`.
- [ ] T028 [P] Run `python -m pytest` for `tests/` and validate the runnable scenarios in `specs/001-neo-date-tracker/quickstart.md`; fix any failures in the relevant source or test files.
- [ ] T029 [P] Verify `.gitignore`, `.env.example`, and `cache/` contain no committed API key or runtime cache data.

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies; all setup tasks may run in parallel.
- **Foundational (Phase 2)**: Depends on Setup; date validation blocks all story work.
- **User Story 1 (Phase 3)**: Depends on the foundation; parser and API client can be implemented in parallel, then cache, service, GUI, and launcher follow their listed dependencies.
- **User Story 2 (Phase 4)**: Depends on User Story 1 because it extends the existing service and GUI flows.
- **User Story 3 (Phase 5)**: Depends on User Story 2 because it extends the shared service and GUI error/fallback handling.
- **Polish (Phase 6)**: Depends on all three user stories.

### User Story Completion Order

```text
Setup -> Foundation -> US1 (P1) -> US2 (P2) -> US3 (P3) -> Polish
```

### Parallel Opportunities

- **Setup**: T001-T005 operate on separate dependency, config, entry-point, test-setup, and fixture files.
- **Foundation**: T006 is the test prerequisite for T007.
- **US1**: T008-T012 can be authored in parallel. T013 and T014 can be implemented in parallel; cache follows parser, service follows parser/API/cache/validator, and GUI follows service.
- **US2**: T019 and T020 can be authored in parallel. T021 and T022 touch separate service and GUI files and can proceed in parallel after the tests.
- **US3**: T023 and T024 can be authored in parallel. T025 and T026 touch separate service and GUI files and can proceed in parallel after the tests.
- **Polish**: T027-T029 touch documentation, test execution, and secret/cache checks and can proceed in parallel after all stories.

## Parallel Examples

### User Story 1

```text
Parallel tests: T008 parser tests; T009 API-client tests; T010 cache tests;
                T011 service tests; T012 GUI tests
Parallel implementation: T013 parser; T014 API client
Then: T015 cache -> T016 service -> T017 GUI -> T018 launcher
```

### User Story 2

```text
Parallel tests: T019 service empty-result tests; T020 GUI validation/fallback tests
Parallel implementation after tests: T021 service; T022 GUI
```

### User Story 3

```text
Parallel tests: T023 service failure/fallback tests; T024 GUI offline/fallback tests
Parallel implementation after tests: T025 service; T026 GUI
```

## Implementation Strategy

### MVP First (User Story 1)

1. Complete Setup and Foundation.
2. Complete User Story 1 and run its focused mocked unit tests.
3. Demonstrate a valid-date lookup with asteroid details, source, timestamp, and disk cache.
4. Stop for MVP validation before adding empty-result and offline fallback behavior.

### Incremental Delivery

1. Setup and date-validation foundation.
2. User Story 1: successful date lookup and result display (MVP).
3. User Story 2: invalid-input prevention and empty-result fallback.
4. User Story 3: persistent offline and error fallback.
5. Polish documentation and validate `quickstart.md`.

## Notes

- All application behavior is covered by unit tests as required by the constitution.
- Mock NASA HTTP calls; do not make real network calls from tests.
- Tests precede implementation within each story.
- `[P]` tasks use separate files and have no dependency on incomplete tasks.
