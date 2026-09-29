# Schulmanager Online for Home Assistant

[![Latest release](https://img.shields.io/github/v/release/tkarle/schulmanager-online-hass)](https://github.com/tkarle/schulmanager-online-hass/releases)
[![HACS custom repository](https://img.shields.io/badge/HACS-Custom-orange.svg)](https://hacs.xyz/)
[![License](https://img.shields.io/github/license/tkarle/schulmanager-online-hass)](LICENSE)

A custom Home Assistant integration for [Schulmanager Online](https://login.schulmanager-online.de/). It provides timetable, change, homework and exam sensors, calendars, and a bundled Lovelace timetable card. Multiple students and accounts with access to multiple schools are supported.

## Features

### Timetable sensors

Each student gets sensors for:

- Current lesson and next lesson
- Lessons today, tomorrow, and on the next school day
- Lessons this week and next week
- Changes today and detected timetable changes

Schedule sensors include structured `lessons` attributes with date, period, time, subject, room, teacher, lesson type, and substitution details where the API provides them. Subject names are also exposed as `subject_sanitized` for templates and voice announcements. API failures are marked with `data_status` and `last_error`; the integration keeps the last successful data when an update fails.

The **Lessons Next Week** sensor reports active lessons, school events, and cancelled periods separately. If a school event replaces cancelled lessons, the event and cancellation entries remain available in the `lessons` attribute, while counts are de-duplicated by date and period. For example:

```text
6 lessons, 5 subjects; Franziskustag (6 periods); 6 cancelled
```

Its attributes include `total_lessons`, `scheduled_periods`, `event_periods`, `cancelled_periods`, `lessons_by_day`, `scheduled_periods_by_day`, `events`, `cancelled_lessons`, and the complete `lessons` list. Event lesson rows use `type: event` and expose the event name as `subject` and `event_title`. Cancelled rows use `type: cancelledLesson` and preserve original subject, room, and teacher values when available.

### Homework, exams, and grades

- Optional homework sensors: due today, due tomorrow, overdue, and upcoming in the next seven days.
- Optional exam sensors: today, this week, next week, and the next 30 days.
- Optional experimental grade sensors: an overall average and one average per subject, when grade data is available.
- A schedule calendar is created for each student. Homework and exam calendars can be enabled in the integration options.

Calendar lesson entries use the times supplied by the school timetable when available. Fallback times are estimates for installations where the API does not return a period time. Cancelled lessons and their replacement entries are grouped by day and period to avoid duplicate calendar events.

### Lovelace timetable card

The integration bundles `schulmanager-schedule-card`, with weekly matrix, weekly list, daily list, and compact views. Home Assistant copies it to `/local/` and registers it when the integration starts. The card is experimental and uses legacy entity-name discovery; the examples below use standard Home Assistant cards and the sensor attributes directly.

The examples below use placeholder entity IDs. Select the actual IDs created for your student under **Settings → Devices & services → Schulmanager Online**.

## Installation

### HACS

1. In HACS, open **Integrations → ⋮ → Custom repositories**.
2. Add `https://github.com/tkarle/schulmanager-online-hass` and choose **Integration**.
3. Download **Schulmanager Online** and restart Home Assistant.
4. Open **Settings → Devices & services → Add integration**, search for **Schulmanager Online**, and sign in.

### Manual

Copy `custom_components/schulmanager_online` from a release into `<config>/custom_components/schulmanager_online`, then restart Home Assistant and add the integration from **Settings → Devices & services**.

## Setup and options

Sign in with the email address and password for your Schulmanager Online account. The integration discovers the students associated with the account. For accounts with access to more than one school, school information is discovered during setup.

In the integration's **Configure** dialog, you can set:

| Option | Description | Default |
| --- | --- | --- |
| Weeks to look ahead | Timetable range, from 1 to 4 weeks | 2 |
| Include homework | Create homework sensors and calendar | On |
| Include exams/tests | Create exam sensors and calendar | On |
| Include letters/messages | Fetch letter data | On |
| Include grades | Fetch grades and create average sensors | Off |
| Highlight schedule changes | Highlight substitutions and changes in the timetable calendar | On |
| Show cancelled lessons without highlighting | Keep cancellation entries visible when highlighting is disabled | Off |

The integration polls Schulmanager Online every 15 minutes. Timetable sensor calculations use Home Assistant's configured local time zone.

## Dashboard examples

### Schedule summary

```yaml
type: entities
title: School
entities:
  - entity: sensor.current_lesson_your_student
    name: Current lesson
  - entity: sensor.next_lesson_your_student
    name: Next lesson
  - entity: sensor.lessons_today_your_student
    name: Lessons today
  - entity: sensor.lessons_changes_today_your_student
    name: Changes today
```

### Next-week event and cancellation details

```yaml
type: entities
title: Next week
entities:
  - entity: sensor.lessons_next_week_your_student
    name: Lessons, events, and cancellations
    secondary_info: last-changed
```

To build a grid or a custom card, use the sensor's `lessons` attribute. Each row contains a `date`, `class_hour`, `time`, `type`, and display fields. Event rows contain `event_title`; cancelled lessons retain `original_subject`, `original_room`, and `original_teacher` where available. The separate `events` attribute groups an event by title and date and lists its periods.

For example, a template can show the event periods without counting the cancelled originals again:

```jinja
{% set events = state_attr('sensor.lessons_next_week_your_student', 'events') or [] %}
{% for event in events %}
{{ event.title }} on {{ event.date }}: periods {{ event.periods | join(', ') }}
{% endfor %}
```

### Calendar

```yaml
type: calendar
initial_view: listWeek
entities:
  - calendar.lessons_your_student
  - calendar.homework_your_student
  - calendar.exams_your_student
```

Homework and exam calendar entities only exist when their options are enabled.

## Automation example

Notify when the state of the next-week summary changes:

```yaml
alias: School schedule update
triggers:
  - trigger: state
    entity_id: sensor.lessons_next_week_your_student
actions:
  - action: notify.mobile_app_phone
    data:
      title: School timetable update
      message: >-
        {{ states('sensor.lessons_next_week_your_student') }}
mode: single
```

For detailed changes, read the `changes` attribute from the **Changes Detected** sensor. Each item includes a date, period, change type, previous and current lesson data, and a description. Explicit cancellations and school events are represented in the timetable rather than being reported as field edits.

## Services

The integration provides these `schulmanager_online` services:

- `refresh`: request an immediate data refresh; limited to once every five minutes.
- `clear_cache`: clear cached authentication tokens so the next update authenticates again.
- `clear_debug`: remove debug dump files created by the integration.

## Troubleshooting

1. Confirm the integration is loaded under **Settings → Devices & services** and that its student device has entities.
2. Check **Developer tools → States** for each sensor's `data_status` and `last_error` attributes.
3. Use the `schulmanager_online.refresh` service to request an update.
4. If authentication fails, check the account credentials and try `schulmanager_online.clear_cache`.
5. For detailed logs, add this to `configuration.yaml` and restart Home Assistant:

   ```yaml
   logger:
     logs:
       custom_components.schulmanager_online: debug
   ```

Please remove credentials and personal student information before attaching logs or diagnostics to an issue.

## Development and support

- [Issues](https://github.com/tkarle/schulmanager-online-hass/issues)
- [Releases](https://github.com/tkarle/schulmanager-online-hass/releases)
- [Sensor attribute reference](documentation/Sensors_Documentation.md)
- [API troubleshooting](documentation/Schedule_API_Troubleshooting.md)

This project is not affiliated with Schulmanager Online. See [LICENSE](LICENSE) for license terms.
