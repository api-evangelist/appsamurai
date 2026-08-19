---
name: appsamurai-publish-a-story
description: Publish a scheduled story into a Storyly placement by walking the account tree from app to instance to story group to story.
api: Appsamurai Storyly External API
spec: openapi/appsamurai-storyly-external-api-openapi.json
base_url: https://api.storyly.io
operations:
  - listApps
  - listInstances
  - createStoryGroup
  - createStory
  - getStories
generated: '2026-08-13'
method: generated
source: openapi/appsamurai-storyly-external-api-openapi.json + data-model/appsamurai-data-model.yml
---

# Publish a story

Storyly's content tree is strict: **app → instance → story group → story**. Every
list operation demands its parent id, so there is no shortcut — you walk down.

## Before you start

- Get a Storyly JWT. Send it as `Authorization: Bearer <JWT>` on every call.
  There is no token endpoint in this API; the credential is issued out of band.
- Note the two failure modes so you can tell them apart: a **missing**
  `Authorization` header returns **400** with `error.code: TokenNotFound`, and an
  **invalid** token returns **401** with `error.code: InvalidToken`. Neither is
  declared in the specification.
- `createStoryGroup` and `createStory` take **`multipart/form-data`**, not JSON,
  because they accept a binary `file` upload. Nested values (`settings`,
  `segments`) are sent as JSON-encoded **strings** inside form fields.

## Steps

1. **Find the app.** `listApps` — `GET /external/app`.
   Optional `limit` (1–100) and `offset` (≥0). Read `data[].id`.
   Nothing here scopes to the caller other than the token, so the account is
   whatever the JWT resolves to.

2. **Find the placement instance.** `listInstances` —
   `GET /external/instance?app_id={app.id}`.
   `app_id` is **required**; you cannot enumerate instances account-wide.
   Read `data[].id` as `instance_id`.

3. **Create the story group.** `createStoryGroup` —
   `POST /external/story-group` as `multipart/form-data`.
   Required: `instance_id`, `title` (max 255 chars), `status`.
   Optional: `file` (cover image) or `media_url`, `ts_start` / `ts_end`
   (ISO-8601, e.g. `2026-09-01T00:00:00Z`) for the scheduling window,
   `segments` and `settings` as JSON strings.
   Response is `{"status":"ok","message":"Story Group successfully created"}` —
   it does **not** return the new id, so step 4 has to look it up.

4. **Read back the id.** `getStoryGroups` —
   `GET /external/story-group?instance_id={instance_id}`.
   Match on `title`. This round-trip is required because create returns no id.

5. **Create the story.** `createStory` — `POST /external/story` as
   `multipart/form-data`.
   Required: `story_group_id`, plus `file` or `media_url`.
   Optional: `title`, `name`, `type`, `deep_link`, `sort_order`, `status`,
   `ts_start` / `ts_end`.

6. **Verify.** `getStories` — `GET /external/story?story_group_id={id}`.
   Results come back in a top-level `data` array with no total count and no
   cursor, so page with `limit`/`offset` until you get a short page.

## Rules

- **No idempotency.** There is no `Idempotency-Key` header and no client-supplied
  key on any create operation. A retried `createStoryGroup` creates a **second**
  story group. Read back with `getStoryGroups` before retrying, never after a
  blind retry.
- **No rate-limit signal.** The API returns no `RateLimit-*` or `Retry-After`
  headers and publishes no limits. Pace your own writes; you will get no warning.
- **`limit` maxes at 100.** Stated only in the parameter description text.
- Quote `error.request` from any error body when contacting support — it is the
  only correlation id this API exposes, and it appears on errors only.
