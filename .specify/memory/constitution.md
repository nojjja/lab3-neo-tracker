<!--
Sync Impact Report
Version change: Unversioned template -> 1.0.0 (initial adoption)
Modified principles:
- PRINCIPLE_1_NAME -> GUI-Only NASA NEO Experience
- PRINCIPLE_2_NAME -> Resilient, Clear User Outcomes
- PRINCIPLE_3_NAME -> Mandatory Unit Tests
- PRINCIPLE_4_NAME -> Persistent Last-Success Cache
- PRINCIPLE_5_NAME -> Secret Hygiene and Minimal Dependencies
Added sections: Application Constraints; Development and Verification
Removed sections: None
Follow-up TODOs: None
-->

# NASA NEO Tracker Constitution

## Core Principles

### I. GUI-Only NASA NEO Experience
The product MUST be a desktop or web GUI, not a console application. It MUST let users
choose a date and display the near-Earth-object results for that date from NASA's NeoWs
API. This keeps the product focused on its intended users and data source.

### II. Resilient, Clear User Outcomes
Invalid input, network or API errors, and responses with no results MUST be handled without
terminating the application. The GUI MUST explain the outcome clearly and allow the user
to continue or correct their input. A zero-result response MUST be distinguished from an
error.

### III. Mandatory Unit Tests
Every change to application behavior MUST be covered by unit tests. Tests MUST include
relevant invalid-input, empty-result, and failure cases so expected problems remain safe
and understandable.

### IV. Persistent Last-Success Cache
Every successful NASA response MUST be cached on disk and remain available after an
application restart. A failed request MUST NOT erase the last successful cached response.
This preserves useful data through temporary service or connectivity problems.

### V. Secret Hygiene and Minimal Dependencies
The NASA API key MUST be loaded from a local `.env` file and MUST NOT be committed to
version control. The application MUST use only dependencies needed for its GUI, API
integration, persistence, and tests; additions MUST have a clear project need.

## Application Constraints

- The selected date is the basis of the NASA NeoWs request; input MUST be validated before
  sending a request.
- The `.env` file MUST be excluded from version control, and credentials MUST NOT be
  embedded in source code or committed configuration.
- Disk caching MUST preserve the latest successful response across process restarts.

## Development and Verification

- Unit tests are a required quality gate for application changes. Tests MUST verify the
  relevant normal and failure behavior, including the user-visible handling of expected
  errors.
- Changes MUST be reviewed against this constitution. Any proposed exception MUST be
  documented and approved as part of the change.
- Validation MUST use the repository's available test and build checks; changes MUST NOT
  be considered complete when required unit tests fail.

## Governance

This constitution governs product, design, implementation, and review decisions for NASA
NEO Tracker. Amendments MUST be made through a reviewed change that records the reason,
impact, and any required migration or follow-up work. Project changes MUST be checked for
compliance with these principles during review; any exception requires explicit approval
and a documented rationale.

Constitution versions use semantic versioning: MAJOR for backward-incompatible
governance changes or principle removals/redefinitions, MINOR for new principles or
material expansions, and PATCH for clarifications and non-semantic wording changes.
The version and amendment date MUST be updated with each approved amendment.

**Version**: 1.0.0 | **Ratified**: 2026-11-17 | **Last Amended**: 2026-11-17
