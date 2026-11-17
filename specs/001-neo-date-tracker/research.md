# Research: NASA NEO Tracker Date Lookup

## NASA NeoWs Feed

**Decision**: Use `GET https://api.nasa.gov/neo/rest/v1/feed` with the selected
`YYYY-MM-DD` value as both `start_date` and `end_date`, plus `api_key`.

**Rationale**: This produces the one-day feed required by the feature and avoids fetching
unrequested dates. The endpoint returns `element_count` and asteroid arrays under
`near_earth_objects`, keyed by date. NASA's listing permits a range of up to seven days;
the application always requests one day. The normal response includes the required
hazard flag, diameter estimates, velocity, and miss distance.

**Alternatives considered**:

- A multi-day request was rejected because the user selects one date.
- `detailed=true` was rejected because its additional orbital details are outside scope.
- JPL's close-approach API was rejected because the required fields and date lookup are
  already available in NeoWs.

**Cache semantics**: A successful application lookup is a valid, parsed response with
one or more asteroid results. An empty upstream result is the feature's no-data outcome,
not a successful cacheable lookup, and therefore does not replace the last successful
non-empty cache. This follows the user's explicit cache design and preserves the
constitution's last-success fallback behavior.

**Response and error decisions**:

- Treat `element_count == 0`, a missing selected date, or an empty selected-day array as
  no data.
- Parse the user-specified `estimated_diameter.meters` min/max, first
  `close_approach_data` entry's kilometers-per-second velocity and kilometer miss
  distance. Convert numeric strings to numbers; convert diameter meters to kilometers
  for display.
- Map HTTP 429 to `RateLimitError`, other unsuccessful HTTP responses to `ApiError`, and
  connection/timeouts to `NetworkError`. NASA's gateway documents 403 for key/access
  problems and 429 for rate limits. Do not retry automatically; show a clear message and
  use cached results if available.
- The NASA feed is called over HTTPS. Do not include `detailed=true` or send date ranges
  longer than needed.

## Python GUI and Background Work

**Decision**: Run the blocking `requests` call on a worker thread. Pass its typed result
or error through `queue.Queue`; drain the queue with the Tk/CustomTkinter `after()` loop
and perform every widget update on the GUI thread.

**Rationale**: Tk event handlers that perform long-running work block input and painting.
Using a queue avoids cross-thread widget access and requires no additional runtime
dependency.

**Alternatives considered**:

- Calling `requests` directly from a button handler was rejected because the UI would
  freeze while the request waits.
- Updating CustomTkinter widgets from the worker was rejected because Tk calls should be
  serialized on the UI event thread.
- A thread pool is unnecessary for the single active lookup; one worker is sufficient.

## HTTP Timeouts and Error Mapping

**Decision**: Set Requests timeout to `(3, 10)` seconds for connect and read respectively.
Translate expected Requests exceptions to the specified typed errors. Treat the read
timeout as an inactivity limit, not a total download deadline.

**Rationale**: Requests can wait indefinitely when no timeout is set. Finite limits ensure
the service can reach its error/fallback path. The app has no strict external API latency
SLA; these values are starting defaults and can be adjusted if real usage warrants it.

**Alternatives considered**:

- An unbounded request was rejected because it can stall the worker indefinitely.
- Automatic retries/backoff were rejected for the initial feature: a bounded request and
  cached fallback avoid added latency and complexity, and rate-limit responses are shown
  clearly instead.

## API-Key Configuration

**Decision**: Load `.env` with `python-dotenv`'s default non-overriding behavior and read
`NASA_API_KEY` from the process environment. Validate that the final key is present before
making a request. Provide only a placeholder in `.env.example`.

**Rationale**: An environment variable already set by the user or execution environment
must take precedence over a local development file. The `.env` file and runtime cache
remain excluded from version control.

**Alternatives considered**:

- Allowing `.env` to override a process-level key was rejected because it can replace
  deployment configuration unexpectedly.
- Embedding a key in source or a packaged default was rejected because it exposes
  credentials.

## Test Strategy

**Decision**: Use pytest fixtures for representative, empty, and malformed feed payloads;
replace the HTTP call with `monkeypatch`/fake responses in API-client tests. Use temporary
directories for cache tests and environment patching for key-configuration tests.

**Rationale**: Unit tests remain deterministic and do not depend on NASA availability or
consume API quota. Pytest and the standard library provide the required mocking and
temporary-file facilities without another dependency.

**Alternatives considered**:

- Live NASA calls in the unit-test suite were rejected because they are network-dependent,
  rate-limited, and nondeterministic.
- Adding a third-party HTTP mocking package was rejected because pytest monkeypatching is
  sufficient for this small client.

## References

- [NASA NeoWs Feed endpoint](https://api.nasa.gov/neo/rest/v1/feed)
- [NASA API gateway](https://api.nasa.gov/)
- [NASA API documentation listing](https://github.com/nasa/api-docs/blob/gh-pages/assets/json/apis.json)
- [api.data.gov developer manual](https://api.data.gov/docs/developer-manual/)
- [Python Tkinter threading model](https://docs.python.org/3/library/tkinter.html#threading-model)
- [Python queue documentation](https://docs.python.org/3/library/queue.html)
- [CustomTkinter documentation](https://customtkinter.tomschimansky.com/documentation/)
- [Requests timeouts](https://requests.readthedocs.io/en/latest/user/quickstart/#timeouts)
- [Requests exceptions](https://requests.readthedocs.io/en/latest/api/#exceptions)
- [python-dotenv documentation](https://saurabh-kumar.com/python-dotenv/)
- [pytest monkeypatch documentation](https://docs.pytest.org/en/stable/how-to/monkeypatch.html)
