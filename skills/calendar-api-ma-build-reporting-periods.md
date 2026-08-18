---
name: build-reporting-periods
description: >-
  Produce gap-free, non-overlapping [start_date, end_date] business-day bounds for a Moroccan month,
  quarter, semester or year — the intervals you drop straight into a SQL BETWEEN or an orchestrator's
  reporting window.
api: Calendar API (calendrier marocain)
generated: '2026-08-18'
method: generated
source: openapi/calendar-api-ma-calendar-api-openapi.yml + https://calendar-api.ma/about.html
operations:
  - ApiV1BdaysSpanMonthSpanMonth
  - ApiV1BdaysSpanQuarterSpanQuarter
  - ApiV1BdaysSpanSemesterSpanSemester
  - ApiV1BdaysSpanYearSpanYear
---

# Build Moroccan reporting periods (CalSpan)

Base URL `https://calendar-api.ma`, header `X-API-KEY: <key>`.

`CalSpan` is this API's distinctive primitive. It returns the **open business-day bounds** of a calendar
period rather than its raw calendar bounds, and consecutive spans are designed to chain without gap or
overlap. The provider built it for Moroccan OPCVM (mutual fund) reporting, where a missing or double-counted
day is a reconciliation break.

## Steps

1. Pick the granularity:
   - `GET /api/v1/bdays/span/month?year=2025&month=7` — `ApiV1BdaysSpanMonthSpanMonth`
   - `GET /api/v1/bdays/span/quarter?year=2025&quarter=3` — `ApiV1BdaysSpanQuarterSpanQuarter`
   - `GET /api/v1/bdays/span/semester?year=2026&semester=1` — `ApiV1BdaysSpanSemesterSpanSemester`
   - `GET /api/v1/bdays/span/year?year=2025` — `ApiV1BdaysSpanYearSpanYear`

2. Read the `CalSpan`: `start_date`, `end_date`, `year`, `semester`, `quarter`, `month`, `mode_calendar`
   (`CalMode`: `D` or `W`) and `country_code`. All eight are required on the response.

3. Use `[start_date, end_date]` directly as an inclusive SQL `BETWEEN` range, or as the window of an
   Airflow/Dagster run. To build a chained series, call the same endpoint once per period and lay the spans
   end to end — they are built to abut.

4. **Expect the start bound to precede the naive calendar start.** The provider's own published example for
   Q3 2025 returns `start_date=2025-06-30`, `end_date=2025-09-30`: the span opens on the last open business
   day *before* the period so that the interval closes cleanly against the previous one. Do not "correct"
   this to 1 July — it is the point of the primitive.

## Rules and gotchas

- Spans inherit the holiday model: while a religious holiday inside the period is still `Estimated`, the
  bounds assume it lands on the estimated date. Re-derive the span after confirmation if the report is
  regulatory.
- All four operations are safe GETs. No pagination, no idempotency key, no published rate limit.
- Errors: `{"status_code", "detail", "extra"}`, `400` declared, `401` on a missing key.
