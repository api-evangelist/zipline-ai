---
name: Monitor a Zipline workflow run
description: Authenticate the CLI, then poll a Zipline Hub workflow run's status and node execution timeline from automation.
api: openapi/zipline-ai-workflow-status-openapi.yml
operations: [getWorkflowStatus]
---

# Monitor a Zipline workflow run

Poll the status of a Zipline Hub workflow after it has been created — for scripts
and external systems that need to track a backfill/serving run to completion.

## Preconditions
- A `workflowId` (returned by the workflow start API or shown in the Hub UI).
- On auth-enabled deployments, a bearer token.

## Steps
1. **Get a token** (auth-enabled deployments only) — run `zipline auth get-access-token` to print a short-lived JWT. First-time CLI auth uses the device flow: `zipline auth login` (RFC 8628).
2. **Poll status** — call `getWorkflowStatus` (`GET /workflow/v2/{workflowId}/status`) with `Authorization: Bearer <token>`:
   ```bash
   curl -H "Authorization: Bearer $(zipline auth get-access-token)" \
     https://<host>/workflow/v2/<workflowId>/status
   ```
   Requires viewer access when role-based authorization is enabled.
3. **Interpret the response** — read the workflow-level `status` and walk `nodeExecutions[].stepRuns[]` for per-node progress and `jobTrackingInfo.jobUrl` links. Re-poll until terminal.

## Error handling
- `401` — missing/invalid token; re-authenticate.
- `403` — caller lacks viewer access.
- `503` — auth service (JWKS) unavailable; the deployment fails closed. Retry after it recovers.

See authentication/zipline-ai-authentication.yml and errors/zipline-ai-problem-types.yml.
