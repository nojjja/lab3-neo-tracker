# Application Contract: Lookup Service and GUI

## Date Validation

`validate_date(value)` returns a clear validation message for an empty value, a value that
does not match `YYYY-MM-DD`, or a string that is not a real calendar date. Valid input
returns no error. The GUI uses this result to keep **Get data** disabled until the date is
valid. The service validates again before starting a request.

## Lookup Service

`lookup(date)` returns a result with:

- `status`: `fresh`, `cached_fallback`, `no_data`, or `invalid`.
- `requested_date`: validated requested date, when valid.
- `data_date`: date of results being displayed, when available.
- `asteroids`: list of parsed asteroid records.
- `source` and `fetched_at`: metadata for the displayed response, when available.
- `warning`: clear user-facing message when using fallback or no result is available.

The `invalid` status is an additional guard for direct service calls; normal GUI use keeps
invalid dates from reaching the lookup action. It never triggers a network request.

Missing or empty `NASA_API_KEY` is a configuration error. It returns cached fallback data
when available or a clear `no_data` message otherwise; it never sends an unauthenticated
request.

For a valid date:

1. A non-empty response that parses successfully is saved and returned as `fresh`.
2. An empty response returns the cached non-empty day as `cached_fallback` when available,
   otherwise `no_data`.
3. Network, API, rate-limit, and parsing failures return cached data as
   `cached_fallback` when available, otherwise `no_data`.
4. A failed cache read is reported to the user as a warning and does not crash the
   application. A cache write failure is also surfaced even if fresh results can be shown.

## GUI Behavior

- The interface is graphical and contains a `YYYY-MM-DD` date field, **Get data** button,
  results list, source/timestamp metadata, and a warning area for fallback/error states.
- The button is disabled for invalid dates and while a lookup is active.
- Network work runs in a background worker. It places its result or typed error on a
  thread-safe queue; the GUI event loop drains the queue via `after()` and performs all
  widget updates on the GUI event thread.
- Each displayed asteroid includes its name when present, diameter range, velocity, miss
  distance, and hazardous status. Missing measurements are visibly marked unavailable.
- A fallback view identifies the cached date and retains the original response timestamp.
