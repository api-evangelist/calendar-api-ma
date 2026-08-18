---
name: schedule-on-business-days
description: >-
  Move a date to the next or previous open Moroccan business day, count or list the business days between
  two dates, and enumerate the working days of a month or year. Use when scheduling jobs, dating
  settlements, or filling a working-day calendar for Morocco.
api: Calendar API (calendrier marocain)
generated: '2026-08-18'
method: generated
source: openapi/calendar-api-ma-calendar-api-openapi.yml + https://calendar-api.ma/bdays-api.html
operations:
  - ApiV1BdaysNextBdaysNext
  - ApiV1BdaysPreviousBdaysPrevious
  - ApiV1BdaysCountCountBdays
  - ApiV1BdaysBetweenDatesBetween
  - ApiV1BdaysYearBdaysYear
  - ApiV1BdaysYearMonthBdaysMonth
---

# Schedule on Moroccan open business days

Base URL `https://calendar-api.ma`, header `X-API-KEY: <key>`.

**What counts as an open business day here:** not a Saturday, not a Sunday, and not an `Official` holiday of
any type. The provider deliberately treats both weekend days as non-working even though only Sunday is
Morocco's official day off, because the economy runs Monday–Friday. Know this before reconciling against
another calendar.

## Steps

1. **Roll a date forward.** `GET /api/v1/bdays/next?date=2025-11-17` (`ApiV1BdaysNextBdaysNext`) returns
   `NextDate {date, next_date}`. Roll backward with `GET /api/v1/bdays/previous?date=2025-11-19`
   (`ApiV1BdaysPreviousBdaysPrevious`) returning `PreviousDate {date, previous_date}`. Both fields are
   required on the response.

2. **Count working days across a range.** `GET /api/v1/bdays/count?start=&end=`
   (`ApiV1BdaysCountCountBdays`) returns `DaysCount {start_date, end_date, count, freq}`. Both bounds are
   inclusive.

3. **List them instead of counting.** `GET /api/v1/bdays/between?start=&end=`
   (`ApiV1BdaysBetweenDatesBetween`) returns a `SerieDatesEng {ref, min_date, max_date, freq, nitems,
   serie}` — `serie` is the full array of dates. There is **no pagination anywhere in this API**: the whole
   series comes back in one response, so bound the range yourself rather than expecting a cursor.

4. **Enumerate a calendar period.** `GET /api/v1/bdays/{year}` (`ApiV1BdaysYearBdaysYear`) or
   `GET /api/v1/bdays/{year}/{month}` (`ApiV1BdaysYearMonthBdaysMonth`), both returning `SerieDatesEng`.
   The `ref` field carries a readable series label such as `bdays-2025-06`.

## Rules and gotchas

- **Estimated holidays leak into business-day maths.** While a religious holiday is still `Estimated`, the
  business-day calculation assumes it will be confirmed on the estimated date. If a job cannot tolerate a
  one-day shift, verify the surrounding holidays with the `check-moroccan-holiday` skill and re-run the
  calculation once they flip to `Official`.
- All operations are GET and safe; retries are safe by method. There is no idempotency key because there is
  nothing to make idempotent.
- No rate limits are published and no `RateLimit-*` or `Retry-After` header is returned — throttle yourself.
- Errors are `{"status_code", "detail", "extra"}` JSON, not problem+json. `400` is declared on every
  operation; `401` on a missing key.
