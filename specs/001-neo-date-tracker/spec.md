# Feature Specification: NASA NEO Tracker Date Lookup

**Feature Branch**: `001-neo-date-tracker` (planned; not created)

**Created**: 2026-11-17

**Status**: Draft

**Input**: User description: "A graphical application NASA NEO Tracker that shows near-Earth asteroids passing Earth on a date chosen by the user. The user enters a date (YYYY-MM-DD) and presses a Get data button. The app shows a list of asteroids with diameter, velocity, miss distance from Earth and a flag whether the asteroid is potentially hazardous. It also shows the data source (NASA NeoWs API) and the timestamp of the response. The button is disabled until the date is valid. Invalid input (empty field, letters, wrong format, non-existent dates like 2026-02-31) shows a clear error and no request is sent. If NASA returns no data or an empty list for the date, the app shows the last successfully cached day and warns the user. If there is no network, the app does not crash, shows a clear message and offers the last cached data with its date. The last successful response is cached locally and survives an app restart."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Look Up Asteroids for a Date (Priority: P1)

As a user, I want to choose a calendar date and retrieve its near-Earth asteroid
information so I can review the objects and their close-approach details.

**Why this priority**: Retrieving and understanding the requested day's data is the
application's primary purpose.

**Independent Test**: Enter a valid date with available results and select **Get data**.
Verify that each asteroid and its required details, the source, and the response timestamp
are shown.

**Acceptance Scenarios**:

1. **Given** a valid date with asteroid results, **When** the user selects **Get data**,
   **Then** the app shows the results for that date with diameter, velocity, miss distance,
   and potentially-hazardous status for each asteroid.
2. **Given** results are displayed, **When** the user reviews the result details,
   **Then** the app identifies NASA NeoWs as the source and shows when the response was
   received.

---

### User Story 2 - Validate a Date and Handle No Results (Priority: P2)

As a user, I want invalid dates rejected before lookup and no-result dates explained so I
can correct input or understand why the selected day's data is unavailable.

**Why this priority**: Preventing invalid lookups and clearly handling empty results avoids
confusing or misleading data.

**Independent Test**: Try an empty value, letters, a wrongly formatted date, and an
impossible date; verify the button stays disabled, a clear validation message is shown, and
no request is made. Then try a valid date with no results and verify the cached-day
fallback or no-cache message.

**Acceptance Scenarios**:

1. **Given** the date is empty, malformed, or not a real calendar date, **When** the user
   enters or edits it, **Then** the app shows a clear validation message, keeps **Get data**
   disabled, and sends no request.
2. **Given** a valid selected date has no data or an empty result list and a cached day is
   available, **When** the lookup completes, **Then** the app warns that the selected date
   has no results and shows the cached day's results and date.
3. **Given** a valid selected date has no data and no cached day exists, **When** the
   lookup completes, **Then** the app explains that there are no results and no cached data
   is available.

---

### User Story 3 - Continue with Cached Data (Priority: P3)

As a user, I want the last successful data to remain available after a connection problem
or an application restart so I can still refer to the most recently available asteroid
information.

**Why this priority**: Persistent fallback data makes the app useful during temporary
network outages and across sessions.

**Independent Test**: Complete a successful lookup, restart the app, then make the network
unavailable and request a valid date. Verify the app remains open, explains the failure,
and offers the cached results with their date.

**Acceptance Scenarios**:

1. **Given** a successful lookup has been cached, **When** the app is restarted, **Then**
   the cached results and their date remain available.
2. **Given** the network is unavailable and cached data exists, **When** the user requests
   a valid date, **Then** the app shows a clear connection error and offers the cached
   results with their date, source, and original response timestamp.
3. **Given** the network is unavailable and no cached data exists, **When** the user
   requests a valid date, **Then** the app shows a clear connection error and explains
   that no cached data is available without closing unexpectedly.

### Edge Cases

- The date is empty, contains letters, uses a format other than YYYY-MM-DD, or represents
  an impossible date such as 2026-02-31.
- A valid date returns no asteroid entries, or the service indicates that no data is
  available for that date.
- The service cannot be reached and there is no cached response to offer.
- A cached response is shown for a date different from the one requested; the app clearly
  identifies the cached date and warns that it is fallback data.
- The application is restarted after a successful lookup; the cached date, results, and
  original response timestamp remain identifiable.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The application MUST provide a graphical interface for selecting a date and
  viewing near-Earth asteroid data; it MUST NOT require a console interface.
- **FR-002**: The date field MUST accept real calendar dates in YYYY-MM-DD format.
- **FR-003**: **Get data** MUST remain disabled until the entered date is valid. Empty,
  malformed, and impossible dates MUST produce a clear validation message and MUST NOT
  initiate a data request.
- **FR-004**: For a valid date, the application MUST retrieve the near-Earth asteroid
  results for that date from NASA NeoWs.
- **FR-005**: For every returned asteroid, the application MUST show its diameter,
  velocity, miss distance from Earth, and whether it is potentially hazardous. Each
  measurement MUST include its unit.
- **FR-006**: The application MUST identify NASA NeoWs as the data source and show the
  response-received timestamp with its time zone.
- **FR-007**: When the selected date has no data or an empty result list, the application
  MUST warn the user and show the most recent cached day with results when one is
  available. If no such cached day exists, it MUST explain that no cached results are
  available.
- **FR-008**: When a request fails because the network or service is unavailable, the
  application MUST remain open, show a clear error, and offer the most recent cached
  results and their date when available. If none are available, it MUST say so.
- **FR-009**: The application MUST cache the latest successful non-empty response locally
  and retain it across restarts. The most recent cached day containing asteroid results
  MUST remain available for fallback.
- **FR-010**: An empty result MUST NOT replace the most recent cached day containing
  asteroid results. A failed request MUST NOT replace the latest successful response or
  the cached day used for fallback.

### Key Entities

- **Date Lookup**: The user-selected calendar date and the result or status returned for
  that date.
- **Asteroid Result**: A near-Earth object with diameter, velocity, miss distance, and
  potentially-hazardous status.
- **Cached Response**: The latest successful non-empty response for a requested date,
  including its result list and response timestamp.
- **Cached Day**: The date, asteroid results, source, and original response timestamp of
  the most recent successful lookup with results, retained for fallback.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In every valid-date lookup with results, users can see all four required
  asteroid details, their units, the data source, and the response timestamp.
- **SC-002**: For each invalid-input case—empty, letters, wrong format, or impossible
  date—the button is disabled, a clear error is visible, and zero data requests are sent.
- **SC-003**: In both empty-result and network-failure cases, the app shows the latest
  cached day with results and a warning when a cache exists; when no cache exists, it
  clearly states that no fallback data is available.
- **SC-004**: After a successful lookup and restart, the cached date and its results
  remain available in every persistence verification.

## Assumptions

- The application is a desktop or web GUI, consistent with the project constitution.
- A date is valid if it is an actual calendar date in YYYY-MM-DD format; availability of
  data for that date is determined by the service response.
- Diameter is shown in kilometers, velocity in kilometers per second, and miss distance
  in kilometers. Hazard status is shown as a clear yes/no value.
- The response timestamp represents when the response was received and includes a time
  zone. Cached data retains its original timestamp.
- An empty-result response is a no-data outcome, not a successful cacheable result; it
  does not replace the most recent cached day containing asteroid results used for
  fallback.
- Access to NASA NeoWs and the required API credentials is available for online lookups;
  credential handling follows the project constitution.
