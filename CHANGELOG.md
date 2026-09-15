# Changelog

## 1.0.10

- Retry schedule requests once when Schulmanager returns authentication or nested API status 401/403.
- Refresh the authentication and bundle-version cache before retrying rejected requests.
- Handle lessons without start times or class-hour numbers during sorting (Issue #9).

## 1.0.9

- Keep Schulmanager sensor list and dictionary attributes stable when API data is missing, unavailable, or partially malformed.
- Reuse the last successful student data during temporary schedule, homework, grade, or exam API failures.
- Add explicit data status metadata for loaded, partial, stale, and not-yet-loaded student data.
- Add tests covering normal data, empty API responses, timeouts, authentication failures, partial fields, and last-successful-data fallback.
