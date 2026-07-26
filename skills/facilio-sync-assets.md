---
name: Sync assets with an external system
description: Incrementally read, create, and update Facilio assets to keep an external asset registry in sync.
api: openapi/facilio-openapi-original.yaml
operations: [getAssetMetadata, listAssets, getAsset, createAsset, updateAsset]
---

# Sync assets with an external system

## Auth
- `x-api-key` header (app `maintenance`) or OAuth2 bearer (app `developer`).
- Base URL: `https://{region}.facilioapis.com/{app_name}/api/v5`.

## Steps
1. `getAssetMetadata` (`GET /asset/metadata`) — map external fields to Facilio fields, `category`, and `type`; pass `includeAllowedValues=true` for picklists.
2. `listAssets` (`GET /asset`) — initial full pull, paging `page`/`pageSize` (max 200). Persist the high-water timestamp.
3. Incremental sync: `listAssets?changed={utcMillis}` — fetch only assets created/modified since the last run.
4. `createAsset` (`POST /asset`) — push new assets with `siteId` and `space`; capture returned `data.id` and store the external↔Facilio id mapping.
5. `updateAsset` (`PATCH /asset/{id}`) — apply field changes for existing mapped assets.
6. `getAsset` (`GET /asset/{id}`) — verify a specific record when reconciling conflicts.

## Rules
- Datetimes are milliseconds since epoch — send/receive `changed` accordingly.
- No idempotency-key header; guard `createAsset` with your stored id mapping to avoid duplicates.
- Errors: `{ success:false, error:{ code, message } }` — `VALIDATION_ERROR`, `UNAUTHORIZED`, `RECORD_NOT_FOUND`.
