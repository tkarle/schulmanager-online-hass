# Changelog

## 1.0.9

- Keep Schulmanager sensor list and dictionary attributes stable when API data is missing, unavailable, or partially malformed.
- Reuse the last successful student data during temporary schedule, homework, grade, or exam API failures.
- Add explicit data status metadata for loaded, partial, stale, and not-yet-loaded student data.
- Add tests covering normal data, empty API responses, timeouts, authentication failures, partial fields, and last-successful-data fallback.
