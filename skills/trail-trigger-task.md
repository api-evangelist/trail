---
name: trail-trigger-task
description: Automatically create a task in Trail from an external system (sensor alert, HR tool, maintenance event) using the Task Instances API or the Evo API.
api: trail:trail-task-instances-api
generated: '2026-07-21'
method: generated
source: openapi/trail_task_instances_v1.yaml, openapi/trail_evo_api_v1.yaml
operations:
  - POST /public/task_instances/v1/task_instances
  - GET /public/task_instances/v1/task_instances/{id}
  - getSites
  - getTaskTemplates
  - getTasks
  - createTask
---

# Trigger a task in Trail

Two routes exist; pick based on the credential you hold:

## Route A — Task Instances API (API_KEY header)

1. **Resolve targets** — `GET /public/sites/v1/list` and `GET /public/task_templates/v1/list` to map site names and template names to integer IDs (see `data-model/trail-data-model.yml`).
2. **Create the task instance** — `POST /public/task_instances/v1/task_instances` with `Content_Type: application/json`; a `201` returns the created instance.
3. **Verify** — `GET /public/task_instances/v1/task_instances/{id}` to confirm status.

## Route B — Evo API (OAuth2)

1. **Authorize** — OAuth2 authorization-code flow against Access Identity (`/connect/authorize`, token at `/api/evo_api/oauth/token`) requesting the `access.trail.api` scope (plus `openid profile email`) — see `scopes/trail-scopes.yml`.
2. **Resolve targets** — real operationIds `getSites` (`GET /evo_api/sites`) and `getTaskTemplates` (`GET /evo_api/task_templates`).
3. **Create** — `createTask` (`POST /evo_api/tasks`); a `201` confirms creation.
4. **Confirm** — `getTasks` (`GET /evo_api/tasks`) filtered by `location_id`, `task_template_id` and `date`.

## Constraints

- Rate limit: 60 requests/60s shared across endpoints (`rate-limits/trail-rate-limits.yml`).
- No idempotency key mechanism exists (`conventions/trail-conventions.yml`) — deduplicate triggers on your side before POSTing, or you will create duplicate tasks.
- `401` = bad/missing credential; `422` = body failed validation; `429` = throttled (errors envelope: `errors/trail-problem-types.yml`).
