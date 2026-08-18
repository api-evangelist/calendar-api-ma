---
name: check-moroccan-holiday
description: >-
  Determine whether a given date is a Moroccan public holiday, and act correctly on whether that holiday is
  Official or still Estimated. Use before scheduling, settling, or dating anything against the Moroccan
  calendar.
api: Calendar API (calendrier marocain)
generated: '2026-08-18'
method: generated
source: openapi/calendar-api-ma-calendar-api-openapi.yml + https://calendar-api.ma/holidays-api.html
operations:
  - ApiV1HolidaysIsHolidayIsHoliday
  - ApiV1HolidaysYearHolidaysYear
  - ApiV1HolidaysHolidays
---

# Check whether a date is a Moroccan holiday

Base URL `https://calendar-api.ma`. Every call needs the header `X-API-KEY: <key>`; keys come from
https://calendar-api.ma/console/ (free account, up to 5 keys). Without the header the API returns
`401 {"status_code":401,"detail":"Auth Header = \`X-API-KEY\` not found in request header"}`.

## Steps

1. **Ask about one date.** `GET /api/v1/holidays/is-holiday?date=2026-11-06`
   (operationId `ApiV1HolidaysIsHolidayIsHoliday`). The response is an `IsHoliday`:
   `date`, `is_holiday`, `description`, `holiday_type`, `status`, `country_code` — all required, so you can
   read them without null checks.

2. **Read `status` before you act, not just `is_holiday`.** `status` is `Official` or `Estimated`.
   - `Official` — confirmed. Safe to make an irreversible decision on.
   - `Estimated` — a religious holiday whose date was computed astronomically and has not yet been
     confirmed by moon sighting. It can move by a day. Do not settle, post, or close a period on it.
   National and exceptional holidays are always `Official`; only religious ones are ever `Estimated`.

3. **If the answer is Estimated and the decision cannot tolerate a one-day shift, poll.**
   The provider's documented pattern is an hourly re-check of the same call until `status` flips to
   `Official`. No webhook or event stream exists — polling is the only change-notification mechanism.
   Note that the API publishes no rate limits at all, so size the loop conservatively and back off on error.

4. **Need the whole year instead of one date?** `GET /api/v1/holidays/{year}`
   (`ApiV1HolidaysYearHolidaysYear`) returns an array of `Holiday`. Narrow it with the optional
   `holiday_type` (`Religious` | `National` | `Exceptional`, or `ND` to disable the filter), `day`,
   `month`, and `description` query parameters. `description` is a case-insensitive wildcard search that
   respects accents — `trô` matches *Fête du trône*.

5. **Need the standing reference list across years?** `GET /api/v1/holidays` (`ApiV1HolidaysHolidays`),
   same filters. Note the dataset uses SCD Type 2 history: a holiday is simply absent for years in which it
   was not defined. Religious holidays are qualified from 2006 onward.

## Rules and gotchas

- Only `description` is required on a `Holiday`; `day`, `month` and `date` are optional, because a religious
  holiday has no fixed Gregorian date until confirmed. Handle their absence.
- The spec's date parameters use ISO `YYYY-MM-DD`. The Python SDK's `to_date()` helper takes `DD/MM/YYYY`.
- `ND` is the provider's sentinel across every enum, meaning "not defined / disable this filter".
- Errors are **not** RFC 9457. The envelope is `{"status_code": int, "detail": string, "extra": object|null}`
  with `application/json`. `detail` is free prose with no stable code — branch on the HTTP status, never on
  the message text. The spec declares `400` on this operation; `401` is returned live but undeclared.
- The API returns no request-id or correlation header, so there is nothing to quote in a support ticket
  beyond the request itself. Support is support@calendar-api.ma.
