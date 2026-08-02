---
name: Fetch online features for a model
description: Look up online GroupBy/Join features from the Zipline (Chronon) Fetcher service for one or more entity keys.
api: openapi/zipline-ai-fetcher-openapi.yml
operations: [listJoins, getJoinSchema, fetchJoin, fetchGroupBy]
---

# Fetch online features for a model

Use the Zipline Fetcher service to retrieve precomputed online features at serving
time. The Fetcher reads from the online KV store and the metadata uploaded during
deploy, so only GroupBys/Joins marked `online=True` are fetchable.

## Preconditions
- The Fetcher service base URL (local sandbox default `http://localhost:9000`, or your deployment host).
- The GroupBy/Join must be deployed and its metadata uploaded online.

## Steps
1. **Discover online joins** — call `listJoins` (`GET /v1/joins`) to see joins marked `online=True`.
2. **Inspect the schema** — call `getJoinSchema` (`GET /v1/join/{name}/schema`) to learn the entity keys and feature fields. URL-encode names containing path separators.
3. **Fetch features** — call `fetchJoin` (`POST /v1/fetch/join/{name}`) with a JSON array of entity-key maps, one map per lookup:
   ```json
   [{"user_id": "5"}, {"user_id": "7"}]
   ```
   For a single GroupBy instead of a join, use `fetchGroupBy` (`POST /v1/fetch/groupby/{name}`).
4. **Read results in order** — the response `results[]` preserves request order. For each entry check `status`:
   - `"Success"` → read `features`.
   - `"Failure"` → read the `error` field for that lookup and handle it.
   A top-level `errors[]` array means the whole request failed (e.g. invalid JSON) — fix and retry.

## Notes
- Batch many lookups in one request array rather than calling once per key.
- There is no pagination and no idempotency key; fetches are reads (see conventions/zipline-ai-conventions.yml, errors/zipline-ai-problem-types.yml).
