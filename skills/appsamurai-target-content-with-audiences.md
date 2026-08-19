---
name: appsamurai-target-content-with-audiences
description: Target Storyly story groups at specific users by creating segment labels and audiences, including audiences built from an interaction with a story component.
api: Appsamurai Storyly External API
spec: openapi/appsamurai-storyly-external-api-openapi.json
base_url: https://api.storyly.io
operations:
  - listSegments
  - createSegment
  - deleteSegment
  - listAudiences
  - createAudience
  - updateAudience
  - updateStoryGroup
generated: '2026-08-13'
method: generated
source: openapi/appsamurai-storyly-external-api-openapi.json + data-model/appsamurai-data-model.yml
---

# Target content with audiences and segments

Storyly has two targeting primitives and they behave very differently.

- A **segment** is a free string `label`. It has no numeric id — it is created,
  listed and deleted **by label**.
- An **audience** is a real entity with an `id` and an `audience_identifier`, and
  it can be built from a user's *interaction* with a story component.

## Steps

### Segments — label-based targeting

1. **List what exists.** `listSegments` — `GET /external/segment`
   (`limit` 1–100, `offset` ≥0). Each row is
   `{account_id, label, is_used}`. `is_used` tells you whether any story group
   currently references the label, which is your safety check before deleting.

2. **Create a label.** `createSegment` — `POST /external/segment` with
   **`application/json`** body `{"label": "vip-customers"}`.
   Note this is the *only* create operation on the API that takes JSON; every
   other one takes `multipart/form-data`.

3. **Attach it to content.** `updateStoryGroup` —
   `PATCH /external/story-group` as `multipart/form-data`, with `id` and
   `segments` as a JSON-encoded string.

4. **Remove a label.** `deleteSegment` —
   `DELETE /external/segment?label=vip-customers`.
   Deletion is **by label, not by id**. Check `is_used` first: nothing stops you
   deleting a label that live story groups still target.

### Audiences — behavioural targeting

5. **List audiences.** `listAudiences` — `GET /external/audience`.
   Each row carries `audience_identifier`, `properties`, `type` and
   `story_group_list` — the back-reference telling you which story groups the
   audience is already attached to.

6. **Create an audience.** `createAudience` — `POST /external/audience` as
   `multipart/form-data`. The interesting form is the interaction-derived one:
   supply `app_id`, `story_group_id`, `story_id`, `component_id`,
   `interactive_type` and `activity` to build "everyone who answered this poll
   this way". A simpler audience takes `name` and `audience_identifier`.

7. **Rename or re-key.** `updateAudience` — `PATCH /external/audience` as
   `multipart/form-data` with `id`, and any of `name`, `audience_identifier`.

## Rules

- **Audiences cannot be deleted.** There is no `deleteAudience` operation. Plan
  your naming, because every audience you create through the API is permanent as
  far as the API is concerned.
- **Segments cannot be updated.** There is no update operation — to rename a
  label you create the new one, re-point the story groups with
  `updateStoryGroup`, then `deleteSegment` the old one.
- **No idempotency key.** A retried `createSegment` with the same label, or a
  retried `createAudience`, is not deduplicated for you. `listSegments` first.
- Client-side targeting is the other half of this: the Placement SDKs expose
  "Targeting with Labels" and "Targeting with Audience" against exactly these
  objects. See components/appsamurai-components.yml.
