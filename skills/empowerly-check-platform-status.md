---
name: empowerly-check-platform-status
description: >-
  Check whether the Empowerly platform is up, which components are affected, and whether there is an
  open incident or an active maintenance window — using the public, unauthenticated Empowerly status
  API. Use this before telling a student or counselor that a portal problem is on their end.
api: Empowerly Status API
base_url: https://status.empowerly.com/api/v2
authentication: none
operations:
  - getStatusSummary
  - getStatus
  - listComponents
  - listUnresolvedIncidents
  - listActiveScheduledMaintenances
generated: '2026-08-12'
method: generated
source: openapi/empowerly-status-api-openapi.yml
---

# Check Empowerly platform status

## When to use

A student or counselor reports that app.empowerly.com is failing, or you want to confirm platform
health before escalating. This skill answers "is it them or is it Empowerly?" in one call.

## Steps

1. **Get everything in one call.** Run `getStatusSummary` — `GET https://status.empowerly.com/api/v2/summary.json`.
   Send no credentials; none are accepted. This single response carries the page, every component,
   all unresolved incidents, all scheduled maintenances, and the overall indicator.

2. **Read the roll-up.** `status.indicator` is one of `none`, `minor`, `major`, `critical`,
   `maintenance`. `none` with `status.description` of "All Systems Operational" means Empowerly is
   reporting no problem — so the issue is most likely local to the user.

3. **Check the component.** Look through `components[]` for `Empowerly Web Portal`
   (id `h2qv5hy8rsbp`). Its `status` is one of `operational`, `degraded_performance`,
   `partial_outage`, `major_outage`, `under_maintenance`. Anything other than `operational` is a
   confirmed platform-side problem.

4. **Check for an open incident.** If `incidents[]` is non-empty, report `name`, `impact`,
   `status` and `shortlink` for each. Incident `status` walks
   `investigating → identified → monitoring → resolved → postmortem`.

5. **Check for planned work.** If `scheduled_maintenances[]` is non-empty, report `name`,
   `scheduled_for` and `scheduled_until`. A window with `status: in_progress` explains degradation
   that is expected and will end.

6. **If you only need liveness**, call `getStatus` — `GET /status.json` — instead. It returns the
   same indicator in about 235 bytes.

7. **If you need only the component roster**, call `listComponents` — `GET /components.json`.

## Rules

- **Do not send credentials.** The API is anonymous and CORS-open (`Access-Control-Allow-Origin: *`).
- **Do not poll faster than once per 10 seconds.** `Cache-Control: max-age=10, s-maxage=10,
  stale-while-revalidate=20` means a faster poll returns the same cached bytes.
- **Use conditional requests.** Store the weak `ETag` and send `If-None-Match`; handle `304`.
- **Do not JSON-parse an error body.** Unknown paths under `/api/v2` return an **HTML** document,
  not `application/problem+json`. Branch on the HTTP status and `Content-Type`.
- **Expect no rate-limit headers.** There are no `X-RateLimit-*`, `RateLimit-*` or `Retry-After`
  headers to read. On a network failure, back off on your own schedule.
- **Quote the trace id when escalating.** Every response carries `atl-traceid` and `atl-request-id`.

## The gap you must state out loud

The status page tracks **only** the web portal. `api.empowerly.com` — the production API host behind
app.empowerly.com — has **no component of its own**. An all-clear on this page therefore does not
rule out an API-layer failure. Say so rather than reporting a clean bill of health as conclusive.

## Do not attempt

There is no Empowerly API for students, counselors, sessions, college lists, essays or applications.
Nothing in the counseling product is programmatically reachable. Do not invent an endpoint, a token
flow, or a base URL for it.
