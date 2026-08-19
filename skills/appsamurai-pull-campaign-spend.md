---
name: appsamurai-pull-campaign-spend
description: Pull AppSamurai campaign spend reporting for a date range, filtered by campaign, bundle, platform or country, with the daily spend series.
api: Appsamurai Campaign Spend API
spec: openapi/appsamurai-campaign-spend-api-openapi.yml
base_url: http://api.appsamurai.com
operations:
  - getCampaignSpend
generated: '2026-08-13'
method: generated
source: >-
  openapi/appsamurai-campaign-spend-api-openapi.yml +
  https://help.appsamurai.com/en/articles/11105087-appsamurai-campaign-spend-api
---

# Pull campaign spend

The AppSamurai advertising platform exposes exactly one reporting operation:
`getCampaignSpend`. It is a single authenticated GET.

## Before you start

- The credential is an **account-scoped API key issued by your AppSamurai account
  manager**. There is no self-service issuance and no rotation mechanism.
- The key goes in the **path**, not a header:
  `/api/customer-pull/spent/{api_key}`.
- **Treat the URL itself as a secret.** Because the key is a path segment it
  lands in proxy logs, browser history and referrer headers. The documentation
  publishes the base as `http://api.appsamurai.com` — force `https://` yourself;
  the host does serve TLS and 301s from port 80.
- **Health check first.** On 2026-08-13 `api.appsamurai.com` returned
  `{"code":500,"message":"Internal Server Error"}` for every path including the
  documented endpoint and the host root. If you get a 500 for everything, the
  problem is not your request — the endpoint may only route from allow-listed
  customer networks.

## Steps

1. **Call it.**
   `GET https://api.appsamurai.com/api/customer-pull/spent/{api_key}`

2. **Filter with query parameters**, all optional:
   - `start_date`, `end_date` — `YYYY-MM-DD`. Omit `end_date` and you get the
     **last 30 days**.
   - `campaign_id`, `campaign_name`, `bundle_id`
   - `platform` — `ios` or `play`
   - `country` — ISO 3166-1 alpha-2, e.g. `US`, `GB`

3. **Read the response.** Each record carries `campaign_id`, `bundle_id`,
   `app_title`, `platform`, `cpi_bid`, `status`, `campaign_name`, plus:
   - `event_goals[]` — `{event_name, cpi_bid, event_token}` per milestone
   - `campaign_names` — an object of campaign names keyed by update date
   - `countries[]` — targeted countries
   - `daily_spend` — an object of `date → amount`, which is the series you
     actually want for reporting

## Rules

- **Read-only.** One GET, no writes, so nothing to make idempotent — but also no
  way to reconcile or correct anything through the API.
- **No pagination.** Narrow with the filters or take the whole result set.
- **No rate limits are published** and no `RateLimit-*` headers are returned.
- Errors are bare HTTP status codes with no documented body: `400` invalid date
  format, `401` bad or missing key, `404` no matching data, `500` server error.
  Note that `404` means "no data matched your filters", not "wrong URL" — do not
  treat it as a fatal integration error.
- There is no versioning: the path carries no version segment and no deprecation
  or Sunset policy is published, so a breaking change would arrive without notice.
