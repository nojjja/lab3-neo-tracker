# External Contract: NASA NeoWs Feed

## Request

- **Method**: `GET`
- **Endpoint**: `https://api.nasa.gov/neo/rest/v1/feed`
- **Query parameters**:
  - `start_date`: selected date in `YYYY-MM-DD` form.
  - `end_date`: same selected date.
  - `api_key`: value supplied by `NASA_API_KEY` from the process environment or `.env`.
- **Timeout**: Use a finite Requests timeout of `(3, 10)` seconds for
  `(connect, read)`. The read timeout is not an overall response deadline.
- The API key MUST NOT be included in logs, error text, or committed source/configuration.

## Response Mapping

The response is JSON containing `element_count` and a `near_earth_objects` mapping whose
date key identifies that day's array of asteroid objects.

For each asteroid, the parser reads:

- `name`
- `is_potentially_hazardous_asteroid`
- `estimated_diameter.meters.estimated_diameter_min`
- `estimated_diameter.meters.estimated_diameter_max`
- `close_approach_data[0].relative_velocity.kilometers_per_second`
- `close_approach_data[0].miss_distance.kilometers`

NASA numeric values may be strings. The parser converts valid numeric values to numbers,
converts diameter values from meters to kilometers, and preserves the other measurements
in their specified units. Missing fields are represented as unavailable. Invalid JSON or
an invalid required structure produces `ParseError`.

If `element_count` is zero, the requested date key is absent, or its object list is empty,
the response is treated as no data. No-data responses do not replace the cached
non-empty response.

## Error Mapping

- Connection and timeout failures -> `NetworkError`.
- HTTP 429 -> `RateLimitError`.
- Other unsuccessful HTTP status -> `ApiError`.
- Invalid JSON or malformed required response structure -> `ParseError`.

The service may use the last valid cached non-empty response for any of these failures.
The UI receives a clear warning and the cached data date when fallback is used.
