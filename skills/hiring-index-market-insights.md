---
name: hiring-index-market-insights
description: Aggregate a job-market slice in one call - salary percentiles, top employers, location and freshness splits - with the 202 retry loop.
api: Hiring Index API
generated: '2026-09-14'
method: generated
source: Grounded in openapi/hiring-index-openapi.yaml operationIds and https://hiringindex.org/docs.
operations:
  - jobInsights
---

# Market insights for a slice

Turn a search filter into the whole picture of that slice - computed server-side - instead of paging thousands of rows and building a spreadsheet.

## Prerequisites

- A RapidAPI key sent as `x-rapidapi-key`, with `x-rapidapi-host: hiringindex.p.rapidapi.com`.
- Base URL: `https://hiringindex.p.rapidapi.com`.

## Steps

1. **Aggregate** (`jobInsights`, `POST /jobs/insights`). Send the *same* filter object you would send to search - one filter vocabulary, not two.
   ```json
   {"job_titles":["Data Engineer"],"country_codes":["US"]}
   ```
2. **Handle 202.** A slice nobody has computed yet returns `202` with `Retry-After` (typically 30s) - not an error. Call again after the delay to get the full aggregates. Each call, including the `202`, counts as one request against the plan. `/jobs/insights` never returns `503`.
3. **Read the aggregates** from the `200`: `headline` (row_count, job_count, company_count, with_salary, with_posted_at, new_this_week), `salary` percentile slices per currency and period (percentiles need >= 30 disclosed postings in a group), `freshness` (median_days_live, pct_last_7_days, pct_over_60_days), `top_companies`, and the city / country / employment-type / remote / seniority / platform splits. `meta.computed_at` tells you how old the aggregate is.

## Retry loop

Retry `202`, `429` and `503`; wait the `Retry-After` header, else `retry_after_seconds` from the body. A short bounded loop is the honest shape - `202` promises the work has started, not that it finishes in the first delay. See rate-limits/hiring-index-rate-limits.yml.
