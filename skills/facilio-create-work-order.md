---
name: Create and track a maintenance work order
description: Create a work order in Facilio, attach context, add a comment, and track it to closure.
api: openapi/facilio-openapi-original.yaml
operations: [getProfile, getWorkOrderMetadata, createWorkOrder, getWorkOrder, updateWorkOrder, addWorkOrderComment, listWorkOrders]
---

# Create and track a maintenance work order

Use the Facilio REST API v5 to raise and manage a corrective/preventive work order.

## Auth
- API key in the `x-api-key` header (app `maintenance`), or an OAuth2 bearer token (app `developer`).
- Base URL: `https://{region}.facilioapis.com/{app_name}/api/v5` (region default `us`).
- Verify credentials with `getProfile` (`GET /profile`) before writing.

## Steps
1. `getWorkOrderMetadata` (`GET /workorder/metadata`) — read allowed values for `status`, `priority`, `category`, `type`, and required fields before constructing the body.
2. `createWorkOrder` (`POST /workorder`) — send `subject`, `siteId`, and any of `priority`, `category`, `type`, `vendor`, `tenant`, `client`. Response envelope is `{ success, data }`; capture `data.id`.
3. `addWorkOrderComment` (`POST /workorder/{id}/comments`) — record context/instructions on the record.
4. `getWorkOrder` (`GET /workorder/{id}`) — confirm the created record.
5. `updateWorkOrder` (`PATCH /workorder/{id}`) — advance `status` (e.g. to a closed state) as work progresses.
6. `listWorkOrders` (`GET /workorder`) — filter with `search`, `view`, or `changed` (delta sync); page with `page`/`pageSize` (max 200), add `?count=true` for totals.

## Rules
- No idempotency-key header exists — do not blindly retry `createWorkOrder`; on a network timeout, `listWorkOrders?search=` to check whether the record was created before retrying.
- On failure the body is `{ success:false, error:{ code, message } }`. Handle `VALIDATION_ERROR` (400), `UNAUTHORIZED` (401), `RECORD_NOT_FOUND` (404). See errors/facilio-problem-types.yml.
- Datetimes are milliseconds since epoch.
