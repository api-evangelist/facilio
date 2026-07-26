---
name: Triage a tenant service request
description: List and inspect incoming service requests, then convert one into an actionable work order.
api: openapi/facilio-openapi-original.yaml
operations: [listServiceRequests, getServiceRequest, getServiceRequestMetadata, updateServiceRequest, createWorkOrder]
---

# Triage a tenant service request

## Auth
- `x-api-key` header (app `maintenance`) or OAuth2 bearer (app `developer`).
- Base URL: `https://{region}.facilioapis.com/{app_name}/api/v5`.

## Steps
1. `listServiceRequests` (`GET /servicerequest`) — pull the queue; filter with `search` or delta-sync `changed`; page with `page`/`pageSize`.
2. `getServiceRequestMetadata` (`GET /servicerequest/metadata`) — resolve allowed `status` values and fields.
3. `getServiceRequest` (`GET /servicerequest/{id}`) — read the full request, including `site`.
4. `updateServiceRequest` (`PATCH /servicerequest/{id}`) — set triage `status`/priority.
5. `createWorkOrder` (`POST /workorder`) — raise the fulfilling work order, carrying over `siteId`, `subject`, and `priority`. Cross-link back to the service request in the work-order body.

## Rules
- Uniform envelope `{ success, data }`; errors `{ success:false, error:{ code, message } }`.
- Use `select` to fetch only needed fields and `expand` to inline lookups on list calls.
- No idempotency key — de-dupe created work orders by `search` before retrying.
