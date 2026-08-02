---
name: trail-pull-task-reports
description: Pull completed/late/missed task data from the Trail Task Reports API and assemble a bespoke compliance report (task instances, checklists, record logs, comments).
api: trail:trail-task-reports-api
generated: '2026-07-21'
method: generated
source: openapi/trail_task_reports_v1.yaml
operations:
  - POST /public/task_reports/v1/task_instances
  - POST /public/task_reports/v1/checklists
  - POST /public/task_reports/v1/record_logs
  - POST /public/task_reports/v1/comments
---

# Pull task reports from Trail

Ground rules (from `conventions/trail-conventions.yml`):
- Authenticate every request with the `API_KEY` header (key generated in Company Settings > Task Reports API; Admin-level user required).
- Send `Content_Type: application/json` — the specs declare it as a required header parameter.
- Rate limit is 60 requests per 60 seconds across all endpoints: batch task IDs per request instead of calling per ID, and back off on HTTP 429.
- Report data is refreshed twice daily — do not poll more often expecting fresher data.

## Steps

1. **Query task instances** — `POST /public/task_reports/v1/task_instances` with your site/date filters. Collect `taskInstanceId` values plus status (completed on time / late / missed), `siteId`, `areaId`, `dueByDatetime`.
2. **Fetch checklists in bulk** — `POST /public/task_reports/v1/checklists` passing multiple task instance IDs at once (this is the documented way to stay inside the rate limit).
3. **Fetch record logs** — `POST /public/task_reports/v1/record_logs` for the same IDs; each `RecordLog` carries `hasError`, `score`, `scoreOutOf`, `percentageScore` and out-of-range records for exception reporting.
4. **Fetch comments** — `POST /public/task_reports/v1/comments`; comments include `attachmentUrls` (photos) and link back via `checkId` / `recordLogRowIndex`.
5. **Join locally** — join on `taskInstanceId` (see `data-model/trail-data-model.yml`); corrective actions raised from a task appear as `actionTaskInstanceIds`.

## Errors

All errors use the `errors[]` envelope (`title`/`detail`/`status` — see `errors/trail-problem-types.yml`): `401` means the API_KEY header is missing/invalid; `422` means the request body failed validation; `429` means you exceeded 60 req/60s — wait and retry.
