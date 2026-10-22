# Changelog

## 1.2.2

- Keep calendar lessons when `subject` is null or empty by falling back to `subject_name`, then `Lesson`.
- Keep lessons without a class-hour number, including AGs, at their actual start/end times and omit the unavailable class-hour description.
- Add offline calendar regression tests for subject fallbacks, unnumbered lessons, API class times, substitutions, cancellations, highlighting, range filtering, and replacement/cancellation de-duplication. Fixtures contain synthetic data only.

## 1.2.1

- Rewrite the README to document the implemented sensors, events and cancellations, calendars, options, bundled schedule card, services, and dashboard examples.
- Fix the lookahead option so the configured 1–4 week range is used for schedule requests.

## 1.2.0

- Include school events such as Franziskustag in schedule sensors, with event titles and occupied class periods exposed as attributes.
- Count cancelled lessons and replacement events by occupied period so they do not inflate the active lesson count.
- Preserve raw cancellation and event entries for detailed schedule views.
- Keep cancelled lessons and school events out of the detected lesson-change list, and do not classify cancellations as substitutions.

## 1.1.3

- Fix config entry migration crashing on startup (`AttributeError: version cannot be changed directly, use async_update_entry instead`) on recent Home Assistant Core versions (confirmed on 2026.9.2). The 1.1.2 migration to version 3 (unique_id fix) set `config_entry.version` directly instead of going through `hass.config_entries.async_update_entry(...)`.

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
