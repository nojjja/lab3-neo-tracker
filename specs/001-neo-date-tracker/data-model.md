# Data Model: NASA NEO Tracker Date Lookup

## Entities

### Requested Date

- **Value**: Calendar date in strict `YYYY-MM-DD` form.
- **Validation**: Must parse as a real calendar date; empty, malformed, and impossible
  dates are rejected before any HTTP request.
- **Request mapping**: The selected date is sent as both `start_date` and `end_date`.

### Asteroid

One near-Earth object returned for the requested date.

| Field | Type | Source | Rules |
|---|---|---|---|
| `name` | string, optional | `name` | Missing name is displayed as unavailable. |
| `diameter_min_km` | number, optional | `estimated_diameter.meters.estimated_diameter_min` | Convert numeric string from meters to kilometers. |
| `diameter_max_km` | number, optional | `estimated_diameter.meters.estimated_diameter_max` | Convert numeric string from meters to kilometers. |
| `velocity_km_s` | number, optional | `close_approach_data[0].relative_velocity.kilometers_per_second` | Convert numeric string to a number. |
| `miss_distance_km` | number, optional | `close_approach_data[0].miss_distance.kilometers` | Convert numeric string to a number. |
| `is_potentially_hazardous` | boolean, optional | `is_potentially_hazardous_asteroid` | Display as yes/no; show unavailable if absent. |

Missing nested fields are tolerated and represented as unavailable values. A present,
non-empty numeric value that cannot be converted is a parse failure. The UI shows
available asteroid entries without inventing missing measurements.

### NASA Feed Response

- **Selected day**: Object list under `near_earth_objects[requested_date]`.
- **Empty state**: `element_count == 0`, a missing selected-day key, or an empty selected
  day list is treated as no data.
- **Malformed state**: Invalid JSON or an invalid required top-level structure raises a
  parser error.
- **Relationships**: One requested date has zero or more asteroid records.

### Cached Response

The last successful non-empty result set, persisted as `cache/last_response.json`.

| Field | Type | Rules |
|---|---|---|
| `requested_date` | string | Valid `YYYY-MM-DD` date associated with the results. |
| `fetched_at` | string | ISO 8601 timestamp with timezone, recorded when the response is received. |
| `source` | string | NASA NeoWs Feed source. |
| `asteroids` | array of Asteroid | Must contain at least one result for a successful cache entry. |

An empty result or failed request does not replace this record. Missing or malformed cache
files are treated as unavailable cache data, surfaced as a warning, and do not crash the
application.

### Lookup Result

The service result returned to the GUI.

| Field | Type | Meaning |
|---|---|---|
| `status` | enum | `fresh`, `cached_fallback`, `no_data`, or `invalid`. |
| `requested_date` | string | Date the user asked to look up, if valid. |
| `data_date` | string, optional | Date associated with displayed fresh or cached data. |
| `asteroids` | array of Asteroid | Results to display; empty when none are available. |
| `source` | string, optional | Data source for displayed results. |
| `fetched_at` | string, optional | Original response timestamp for displayed results. |
| `warning` | string, optional | User-facing reason for fallback, missing data, or invalid input. |

`fresh` means a non-empty response for the requested date was parsed and returned;
successful cache persistence is reported separately through `warning` if it fails.
`cached_fallback` means cached non-empty results are shown for a different date.
`no_data` means no usable results or cache are available. `invalid` is a validation
guard state and never causes an HTTP request.

## State Transitions

```text
Input
├── invalid -> invalid result; no request
└── valid
    ├── non-empty response -> fresh result -> persist cache
    ├── empty response -> cached_fallback if available, otherwise no_data
    └── network/API/rate-limit/parse failure
        -> cached_fallback if available, otherwise no_data
```

Cache load failures are non-fatal and add a user-visible warning. A successful response
that cannot be persisted may still be displayed as fresh data, but the result must warn
that it could not be saved for future offline use.
