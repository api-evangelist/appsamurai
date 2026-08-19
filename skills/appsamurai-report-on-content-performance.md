---
name: appsamurai-report-on-content-performance
description: Pull daily, weekly, monthly and total performance roll-ups for Storyly story groups and individual stories over a date window.
api: Appsamurai Storyly External API
spec: openapi/appsamurai-storyly-external-api-openapi.json
base_url: https://api.storyly.io
operations:
  - getStoryGroupStats
  - getStoryStats
  - listInstances
  - getStoryGroups
generated: '2026-08-13'
method: generated
source: openapi/appsamurai-storyly-external-api-openapi.json + data-model/appsamurai-data-model.yml
---

# Report on content performance

Storyly returns analytics as pre-bucketed roll-ups attached to the content
entity, not as a separate metrics resource. Two operations cover it.

## Steps

1. **Scope the request.** Both stats operations take a **required** window:
   `ts_start` and `ts_end`, both `YYYY-MM-DD`. Every other filter is optional,
   but if you supply none you will get everything the token can see.

2. **Story-group level.** `getStoryGroupStats` —
   `GET /external/story-group/stats?ts_start=2026-08-01&ts_end=2026-08-13`.
   Optional narrowing: `instance_id`, `story_group_id`.
   Each row carries `id`, `name`, `icon`, `instance_id`, `segments`, `status`,
   the scheduling window (`ts_start`, `ts_end`, `ts_created`) and four buckets:
   `daily-stats`, `weekly-stats`, `monthly-stats`, `total-stats`.

3. **Story level.** `getStoryStats` —
   `GET /external/story/stats?ts_start=…&ts_end=…`.
   Optional narrowing: `app_id`, `instance_id`, `story_group_id`, `story_id` —
   this is the one operation on the API that lets you filter at any level of the
   tree, so prefer it when you need a cross-cutting read.
   Same four buckets, plus `media`, `outlink`, `thumbnail` and `settings`.

4. **Resolve names if you need them.** The stats rows give you ids and a `name`,
   but to map back to the tree use `listInstances` (requires `app_id`) and
   `getStoryGroups` (requires `instance_id`).

## Rules

- **The bucket keys are hyphenated** — `daily-stats`, not `daily_stats` — while
  every other field on the API is snake_case. Deserialize accordingly; this is
  the most common parsing mistake against this API.
- **No pagination on stats.** Unlike the list operations, neither stats operation
  accepts `limit` or `offset`. A wide window on a large account returns
  everything in one response.
- The specification declares only a `200` for these operations. In practice a
  missing `Authorization` header returns **400** `TokenNotFound` and an invalid
  token returns **401** `InvalidToken` — handle both.
- No rate-limit headers are returned. If you are backfilling a long history,
  chunk the window yourself rather than relying on the API to tell you to slow
  down.
