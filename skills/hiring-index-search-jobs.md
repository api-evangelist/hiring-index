---
name: hiring-index-search-jobs
description: Search live job postings by title, location, salary and posting age, then fetch full detail for a posting.
api: Hiring Index API
generated: '2026-09-14'
method: generated
source: Grounded in openapi/hiring-index-openapi.yaml operationIds and https://hiringindex.org/docs.
operations:
  - searchJobs
  - getJob
---

# Search live job postings

Find current openings across thirteen applicant-tracking systems behind one endpoint, then pull the full record for any one of them.

## Prerequisites

- A RapidAPI key. Send it on every request as `x-rapidapi-key`, with `x-rapidapi-host: hiringindex.p.rapidapi.com`.
- Base URL: `https://hiringindex.p.rapidapi.com`.

## Steps

1. **Search** (`searchJobs`, `POST /jobs/search`). Send a filter object. Every field is optional; an absent field is not applied. A key outside the vocabulary is rejected `422` with the list of valid keys, so fix the typo it names.
   ```json
   {"job_titles":["Data Engineer"],"cities":["Berlin"],"country_codes":["DE"],"salary":{"min":80000},"days_ago":30,"page":1,"limit":20}
   ```
   Keywords must be 3+ characters (the title index is a trigram index). `limit` goes up to 100.
2. **Read the page.** The response is `{ jobs[], total_count, company_count, page, limit, total_pages, meta }`. An empty `jobs` array with `total_count: 0` is a normal `200` - a filter that matched nothing, not a failure. A field the source did not state is absent from the object (never null, never ""), so test presence, e.g. `"salary" in job`.
3. **Page** by incrementing `page` until `page == total_pages`.
4. **Fetch detail** (`getJob`, `GET /jobs/{id}`) using a `_id` from the search response for the full posting, including `apply_url` where the source carries one.

## Errors and retries

- `400`/`422` `invalid_request`: a parameter was rejected; the message names it.
- `422` `keyword_too_common`: the keyword matches too much of the corpus (body carries `meta.estimated_matches`); use `job_titles` or a rarer keyword.
- `429`/`503`: wait the `Retry-After` header (or `retry_after_seconds` in the body) and retry. See conventions/hiring-index-conventions.yml.
