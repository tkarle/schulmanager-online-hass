# Changelog

## 1.1.2

- Fix multi-school accounts never getting HA devices created (`UnboundLocalError` in `async_setup_entry`, silently swallowed).
- Fix the `get_grades` request to use the current `moduleName`/`endpointName` API format instead of the legacy `method`/`data` shape that the backend appears to have dropped (unverified against a live account - grades is opt-in).
- Remove a dead, shadowed duplicate of `get_class_hours` in `api.py`.
- Remove a dead, unreachable duplicate of `async_unload_entry` in `__init__.py`.
- Fix `diagnostics.py` reading a config-entry key (`api`) that was never stored, which made the service/multi-school diagnostics sections always empty.
- Make `unique_id` for per-student sensors and calendars derive from the stable `student_id` instead of the display name, avoiding collisions between same-named students and orphaned entities on name changes. Includes a one-time migration (config entry version 3) that rewrites existing entities' `unique_id` in place so history and customizations are preserved.
- Use Home Assistant's configured time zone (`dt_util.now()`) instead of the host's local time for "today"/"current lesson" calculations in the coordinator.
- Cache the institution-wide class-hours (period-to-time) grid once per refresh cycle instead of re-fetching it per student.
- Cache resolved institution display info (name/city/address) in the config entry so `async_setup_entry` doesn't re-authenticate and re-fetch it on every Home Assistant start.
- De-duplicate the institution-name/address-building logic and the per-student API-instance lookup into shared helpers (`institution_utils.py`, `student_utils.py`).
- Use `.get(...)` defensively when reading primary-teacher fields to avoid a possible `KeyError`.

## 1.0.10

- Retry schedule requests once when Schulmanager returns authentication or nested API status 401/403.
- Refresh the authentication and bundle-version cache before retrying rejected requests.
- Handle lessons without start times or class-hour numbers during sorting (Issue #9).

## 1.0.9

- Keep Schulmanager sensor list and dictionary attributes stable when API data is missing, unavailable, or partially malformed.
- Reuse the last successful student data during temporary schedule, homework, grade, or exam API failures.
- Add explicit data status metadata for loaded, partial, stale, and not-yet-loaded student data.
- Add tests covering normal data, empty API responses, timeouts, authentication failures, partial fields, and last-successful-data fallback.
