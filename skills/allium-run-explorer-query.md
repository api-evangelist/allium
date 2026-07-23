---
name: Run an Allium Explorer query and fetch results
description: Execute a saved Allium Explorer query asynchronously, poll for completion, and retrieve the result set.
api: openapi/allium-openapi-original.json
operations:
  - execute_query_async_api_v1_explorer_queries__query_id__run_async_post
  - get_query_run_status_api_v1_explorer_query_runs__run_id__status_get
  - get_query_run_results_api_v1_explorer_query_runs__run_id__results_get
  - cancel_query_run_api_v1_explorer_query_runs__run_id__cancel_post
---

# Run an Allium Explorer query and fetch results

Use this skill to run a saved Allium Explorer query against the blockchain data
warehouse and collect its results. All calls go to `https://api.allium.so` and must
carry the header `X-API-KEY: <your-api-key>` (get a key at
https://app.allium.so/settings/api-keys). Responses are JSON.

## Steps

1. **Kick off the run.** POST to `/api/v1/explorer/queries/{query_id}/run-async`
   (`execute_query_async_api_v1_explorer_queries__query_id__run_async_post`) with any
   query parameters in the body. The response returns a `run_id`.
   - For short queries you may instead call the synchronous
     `/api/v1/explorer/queries/{query_id}/run`
     (`execute_query_api_v1_explorer_queries__query_id__run_post`), which blocks and
     returns results directly.

2. **Poll status.** GET `/api/v1/explorer/query-runs/{run_id}/status`
   (`get_query_run_status_api_v1_explorer_query_runs__run_id__status_get`) until the
   status is a terminal state (success/failed). Back off between polls.

3. **Fetch results.** On success, GET
   `/api/v1/explorer/query-runs/{run_id}/results`
   (`get_query_run_results_api_v1_explorer_query_runs__run_id__results_get`) to retrieve
   the rows plus column metadata and run stats.

4. **Cancel if needed.** To abort a long-running query, POST
   `/api/v1/explorer/query-runs/{run_id}/cancel`
   (`cancel_query_run_api_v1_explorer_query_runs__run_id__cancel_post`).

## Rules

- Authenticate every request with the `X-API-KEY` header; a missing/invalid key is rejected.
- A malformed request returns `422 Validation Error` (`HTTPValidationError`,
  `detail[].loc/msg/type`) — read `detail` to fix the offending field. See
  `errors/allium-problem-types.yml`.
- There is no documented idempotency key; do not assume retries are deduplicated —
  prefer polling an existing `run_id` over re-issuing the run.
