# Store Buy/Sell Business Rules

## Purpose and scope

This document describes the backend business rules for the Store Buy/Sell workflow in the Menu Admin API.

It is intentionally API-focused, but it includes the database rules, stored procedures, triggers, and Hangfire behavior that determine API-visible outcomes.

This guide excludes UI behavior.

This guide separates:

- current enforced behavior from live code and schema artifacts
- known gaps, ambiguities, and stale-spec drift that should not be treated as settled business rules

## Source-of-truth order

When sources disagree, use this order:

1. Live API and workflow code
2. Live SQL builders and migrations
3. Supporting AI docs that explain database and environment behavior
4. Older StoreBuySell reimplementation specs and follow-up notes

Primary live sources:

- `Lce.MenuAdmin.Api/Controllers/MenuManagerStoreBuySellController.cs`
- `Lce.MenuAdmin.Api/Services/StoreBuySellService.cs`
- `Lce.MenuAdmin.Api/HostedServices/StoreBuySellJobIntentDispatcher.cs`
- `Lce.MenuAdmin.Core/Models/MenuManagerStoreBuySell/StoreBuySellRequestPayload.cs`
- `Lce.MenuAdmin.Core/Models/MenuManagerStoreBuySell/StoreBuySellJobIntent.cs`
- `Lce.MenuAdmin.Core/Processors/StoreBuySell/StoreBuySellProcessor.cs`
- `Lce.MenuAdmin.Core/Processors/StoreBuySell/StoreBuySellTransitionRules.cs`
- `Lce.MenuAdmin.Core/Processors/StoreBuySell/Steps/*.cs`
- `Lce.MenuAdmin.Core/Data/Repositories/StoreBuySellWorkflowRepository.cs`
- `Lce.MenuAdmin.Core/Data/Repositories/StoreBuySellMenuManagerRepository.cs`
- `Lce.MenuAdmin.Core/Data/Sql/StoreBuySellWorkflowSql.cs`
- `Lce.MenuAdmin.Core/Data/Sql/StoreBuySellMenuManagerSql.cs`
- `Lce.MenuAdmin.Migrations/Migrations/20260713001.CreateStoreBuySellSchema.cs`

Important drift to keep in mind:

- The live route is environment-scoped: `/api/v{version}/{environment}/menumanager/store-buy-sell`.
- Cancel is `POST /{requestId}/cancel`, not a `DELETE` endpoint.
- The live workflow columns are `requested_enqueue_utc` and `hangfire_enqueue_utc`.
- Request acceptance now uses durable dispatch intents and a background dispatcher instead of immediate Hangfire scheduling in `enqueue` or `schedule`.
- Step-level Menu Manager audit logging now exists in code, so older notes that describe all step audit as missing are stale at that level.

## Business summary

A Store Buy/Sell request moves one or more existing store organizations under a target organization.

The workflow may create the target organization first, depending on `IsNewOrganization`, then:

1. snapshots current organization state
2. updates store records and organization parentage
3. resets tax-sync state
4. refreshes hierarchy state
5. resets menu publication state
6. requests information-feed refresh

The API stores workflow state in the Menu Admin database and applies business mutations later against the environment-specific Menu Manager database.

## Current enforced API rules

### Base route and authorization

- All endpoints are under `/api/v{version}/{environment}/menumanager/store-buy-sell`.
- Read endpoints require `AuthPolicies.Permissions.MenuManagerStoreBuySellRead`.
- Mutating endpoints require `AuthPolicies.Permissions.MenuManagerStoreBuySellEdit`.
- Controller metadata advertises `401`, `403`, and `500` responses.

### Endpoint rules

| Endpoint | Current rule |
| --- | --- |
| `POST /enqueue` | Validates the request body, generates a new `Guid` request ID, inserts a workflow row with status `Queued` and `requested_enqueue_utc = UtcNow`, creates or refreshes a durable job-intent row in the same transaction, and returns `202 Accepted` with `QueueResponse`. No immediate Hangfire job is created on this request path. |
| `POST /schedule` | Validates the request body, generates a new `Guid` request ID, inserts a workflow row with status `Scheduled` and `requested_enqueue_utc = EnqueueAt`, creates or refreshes a durable job-intent row in the same transaction, and returns `202 Accepted` with `QueueResponse`. No immediate Hangfire job is created on this request path. |
| `GET /{requestId}` | Returns `200 OK` with `StoreBuySellRequest` when found in the route environment; otherwise `404 Not Found`. |
| `POST /{requestId}/cancel` | Uses environment-scoped cancel rules. Missing request returns `404 Not Found`. Requests outside allowed cancel states (`Queued`, `Scheduled`) return `409 Conflict`. Successful cancel transitions status to `Canceled`, removes matching durable intent rows, deletes the recorded Hangfire job when present, and returns `204 No Content`. |
| `POST /{requestId}/reschedule` | Uses environment-scoped reschedule rules. Missing request returns `404 Not Found`. Requests outside allowed reschedule states (`Queued`, `Scheduled`) return `409 Conflict`. On success, updates `requested_enqueue_utc`, resets Hangfire metadata, upserts a durable intent row, and returns `204 No Content`. Any previously stored Hangfire job ID is deleted only after the durable reschedule update succeeds. |
| `GET /` | Returns an environment-scoped paged request list. Optional filters are target `organizationId` and exact `status`. Invalid paging returns `400 Bad Request` with `Invalid paging parameters.` |
| `GET /{requestId}/step-logs` | Returns environment-scoped paged step logs for one request. Optional filter is exact `status`. Invalid paging returns `400 Bad Request` with `Invalid paging parameters.` |

### Paging rules

- The controller accepts only `page >= 1`.
- The controller accepts only `1 <= pageSize <= 100`.
- Request lists are ordered by `created_utc DESC`.
- Step logs are ordered by `step_number ASC`.

## Current enforced payload rules

### Top-level request payload

| Field | Current rule | Notes |
| --- | --- | --- |
| `OrganizationId` | Conditionally required. Required and must be `>= 1` when `IsNewOrganization = false`. Optional when `IsNewOrganization = true`; if supplied it must be `>= 1`. | For new-target requests with omitted target ID, workflow storage uses unresolved sentinel `0` until Step 2 assigns a real ID. |
| `ParentOrganizationId` | Required and must be `>= 1` when `IsNewOrganization` is `true`. | Used only when Step 2 creates a new organization. |
| `FranchiseNumber` | Required and must be `>= 1`. | Used by Step 2 for franchise creation semantics and by Step 3 when updating transferred stores. |
| `Name` | Required non-empty string when `IsNewOrganization` is `true`. | Used as target organization name and in list-query projections. |
| `Country` | Required non-empty string when `IsNewOrganization` is `true`. | Used only when Step 2 creates a new organization. |
| `IsNewOrganization` | Optional boolean, defaults to `false`. | Controls whether Step 2 runs or is skipped. |
| `Organizations` | Optional list, defaults to an empty list. | No explicit max-size validation exists. `StoreNumber` values in this list must be unique within one request. |

### Sub-organization payload items

| Field | Current rule | Notes |
| --- | --- | --- |
| `OrganizationId` | Required integer. | There is no explicit positive-range validation here. |
| `StoreNumber` | Required and must be `>= 1`. | Written to `organization_as_store`. |

### Additional enforced request validation

- Store numbers are validated for uniqueness inside each request payload regardless of `IsNewOrganization`.
- When `IsNewOrganization = false`, `OrganizationId` is required and must be greater than zero.
- When `IsNewOrganization = true`, `OrganizationId` may be omitted. If provided, it must be greater than zero.
- Step 2 target-organization creation resolves requested target ID from `payload.OrganizationId` only and does not fall back to workflow-row `request.OrganizationId`.
- When `IsNewOrganization = false`, requested store numbers are validated against existing direct child stores under the target `OrganizationId`.
- If one or more requested store numbers already exist under that target-organization scope, enqueue/schedule is rejected with `400 Bad Request`.

### Scheduling payload

- `ScheduleStoreBuySellRequestPayload` adds `EnqueueAt`.
- `EnqueueAt` has no explicit validation attribute and no explicit future-time rule in controller or service code.
- Service code rejects values that are not in the future.

## Workflow storage and state rules

### Workflow storage split

- Workflow tracking lives in the Menu Admin database.
- Durable Hangfire dispatch intents are stored in the Menu Admin database outbox table `store_buy_sell_job_intent`.
- Business mutations run later against the Menu Manager database resolved from the route environment.

### `store_buy_sell_request` row semantics

Each request is stored with these business fields:

- request ID
- target `environment_name`
- optional request `correlation_id`
- submitting user identity fields
- target `organization_id`
- full request `payload` as JSONB
- workflow `status`
- `current_step`
- `requested_enqueue_utc`
- `hangfire_job_id`
- `hangfire_enqueue_utc`
- `last_error`
- `version`
- `canceled_utc`
- `completed_utc`
- `hangfire_state`
- `hangfire_state_utc`

Current write behavior:

- enqueue and schedule insert the row with `current_step = 0`
- enqueue and schedule persist unresolved target-organization ID as `organization_id = 0` when `IsNewOrganization = true` and top-level `OrganizationId` is omitted
- enqueue and schedule store the requested user identity directly on the row
- enqueue and schedule persist the ambient request correlation id when one is available
- enqueue and schedule also upsert a durable job intent in the same transaction
- Hangfire metadata (`hangfire_job_id` and `hangfire_enqueue_utc`) is written by the dispatcher when a durable intent is applied to the request row
- Step 2 assigns a concrete target organization ID when needed; after that step commits, processor backfills `store_buy_sell_request.organization_id` from sentinel `0` to the assigned value
- cancel writes `canceled_utc` and status `Canceled` only when current status is in the allowed cancel set
- cancel removes matching durable job intents for the request/environment
- reschedule updates `requested_enqueue_utc`, clears existing Hangfire metadata, and upserts a durable job intent when current status is in the allowed reschedule set
- processor status updates write `status`, `current_step`, `last_error`, and increment `version`

Current non-behavior:

- this workflow code does not set `completed_utc`
- this workflow code does not set `hangfire_state` or `hangfire_state_utc`

### Step-log row semantics

`store_buy_sell_request_step_log` stores one logical row per request and step number.

Rules:

- `(request_id, step_number)` is unique
- `Started`, `Succeeded`, `Failed`, and `SkippedNotNeeded` are written through upsert statements
- retries rewrite the same step-log row rather than creating a new history row for the same step number

### Durable outbox row semantics

`store_buy_sell_job_intent` stores one dispatch intent per workflow request (`request_id` is unique).

Rules:

- insert/schedule/reschedule paths create or refresh the intent row through upsert
- dispatcher reads intents in `created_utc ASC` order
- dispatcher increments `attempt_count` on dispatch or failure updates
- dispatcher stores the last dispatch error in `last_error` when dispatch/apply fails
- dispatcher deletes the intent after metadata is successfully applied to the request row
- dispatcher deletes intents for missing requests, environment mismatches, or terminal request states

### Request list projection rules

The paged request list is built only from the workflow JSON payload in Menu Admin. It does not join the Menu Manager database.

Current projection rules:

- `TargetOrganizationId` comes from `store_buy_sell_request.organization_id` with unresolved sentinel `0` mapped to unset (`null`) in API responses
- `TargetOrganizationName` comes from `payload.Name`
- `OrganizationId` on the list item comes from the first element of `payload.Organizations`
- `OrganizationName` on the list item is derived as zero-padded `payload.FranchiseNumber` + `-` + zero-padded first `payload.Organizations[].StoreNumber`
- `StoreCount` comes from `jsonb_array_length(payload.Organizations)`
- filters apply to target `organization_id` and exact request `status`

## Workflow execution rules

### Status model

| Status | Current meaning |
| --- | --- |
| `Queued` | Request was accepted for immediate execution. |
| `Scheduled` | Request was accepted for future execution. |
| `Running` | Processor is currently executing or retrying a step. |
| `StepFailed` | The current step failed and the processor rethrew the exception for Hangfire retry. |
| `Succeeded` | All steps completed successfully. |
| `Canceled` | Request was canceled through the cancel endpoint. |
| `SkippedNotNeeded` | Step-log-only value used when a step is intentionally skipped. |

### Processor guardrails

- Hangfire retries the processor up to 5 times with delays of 10, 30, 60, 120, and 300 seconds.
- Hangfire applies a 30-minute `DisableConcurrentExecution` lock.
- If the request does not exist when the processor starts, the job logs a warning and returns.
- If the request already has status `Succeeded`, the job logs that fact and returns.
- If the request already has status `Canceled`, the job logs that fact and returns.
- The processor executes only steps where `step.StepNumber > request.CurrentStep`, so retries resume from the next pending step.

### Durable dispatch and reconciliation rules

The API host includes an auto-hosted dispatcher (`StoreBuySellJobIntentDispatcher`) that continuously reconciles durable intents with Hangfire metadata.

Rules:

- dispatcher polls pending intents in batches and loads the current workflow request
- when an intent has no `hangfire_job_id`, dispatcher creates either `Enqueue` or `Schedule` Hangfire jobs based on `requested_enqueue_utc`
- dispatcher records created Hangfire IDs on the intent first, then applies job metadata to the request row
- metadata apply succeeds when request state is still dispatch-applicable (`Queued`, `Scheduled`, or `Running`) and not canceled
- if metadata apply loses a race, dispatcher reloads request state: terminal states trigger intent cleanup, non-terminal mismatches record a dispatch failure for retry
- terminal request statuses (`Canceled`, `Succeeded`, `StepFailed`) trigger intent cleanup, and cleanup deletes any recorded Hangfire job ID

### Environment restoration rules

The route environment is part of the business contract, not a transport detail.

Rules:

1. The controller captures `{environment}` from the route.
2. The service persists that environment slug in the workflow row.
3. The Hangfire processor reloads the workflow row first.
4. The processor re-resolves the environment through `IEnvironmentRepository`.
5. If the environment is missing or deleted, processing fails closed before any Menu Manager connection is opened.
6. The processor sets `IEnvironmentContextAccessor.EnvironmentContext` before obtaining a Menu Manager connection.
7. The processor clears the environment context in a `finally` block.

### Per-step execution rules

For each pending step the processor:

1. writes step-log status `Started`
2. writes overall request status `Running`
3. evaluates `Skip(request)`
4. either writes `SkippedNotNeeded` or executes the step
5. writes step success when execution completes
6. advances `current_step`
7. commits the step transaction when one exists

If a step throws:

- the step transaction is rolled back when one exists
- the step log is written as `Failed` with `ex.ToString()`
- the request status becomes `StepFailed`
- `last_error` stores `ex.Message`
- the exception is rethrown so Hangfire can retry

## Step-by-step business rules

| Step | Transaction | Skip rule | Current database behavior |
| --- | --- | --- | --- |
| 1. Save Current Organization | Yes | Never | Calls `helper_organization_save_current()` and records audit entries representing the workflow scope in `organization_saved` and `organization_as_store_saved`. |
| 2. Create New Organization | Yes | Skipped when `!request.Payload.IsNewOrganization` | Inserts the target organization into `organization` using top-level payload fields. If the post-write audit snapshot cannot be read, the workflow still succeeds and writes a fallback audit payload. |
| 3. Update Organization As Store | Yes | Never | For each `Organizations` item, updates `organization_as_store.franchise_number` and `.store_number`, then updates `organization.parent_organization_id` and `organization.name` so the transferred organization becomes a store under the new target. |
| 4. Update Tax Rate Division | Yes | Never | For each `Organizations` item, sets `store_process_menu_tax_store_status.tax_rate_update_synced_on` to `1900-01-01` to force downstream tax-rate refresh behavior. |
| 5. Update Hierarchy And Create Franchise Menu | No | Never | Calls `organization_hierarchy_update_level()`, then `organization_hierarchy_update_ancestry()`, then reactivates inactive ancestor organizations with `ancestor_id > 50` for the transferred stores. Explicit application code does not create franchise-menu rows directly; any such side effects are inside the called database routines. |
| 6. Menu Republish | Yes | Never | Uses distinct store organization IDs, deactivates `organization_xref_service_location` rows for non-default service locations (`service_location_id NOT IN (1,2)`), sets `publish_menu = false`, sets `deliverect_ready = false`, clears `deliverect_channel_link_id`, calls `helper_delete_org_menu(<orgId>)` for each store, then calls `store_process_menu_store_reset(@organizationId)` and `store_process_dmb_store_reset(@organizationId)`. |
| 7. Trigger Information Feed | Yes | Never | For each `Organizations` item, upserts `store_process_info_feed` with `requested = true`, `requested_on = now()`, `requested_due_deactivation = false`, and `completed = false`. |

### Distinct-versus-duplicate behavior

- Steps 5 and 6 operate on distinct store organization IDs.
- Steps 3, 4, and 7 iterate the raw `Organizations` list and do not deduplicate it first.
- Duplicate `Organizations[].StoreNumber` values are rejected at request validation and do not reach processor execution.

If the payload contains duplicate store organization IDs, some steps will repeat work while others will not.

## Database-backed side effects that matter to the API

### Triggered information-feed side effects

The `menu-publisher` database attaches `store_process_info_feed_trigger_function` to:

- `organization_as_store`
- `organization_as_store_attribute`
- `organization_xref_service_location`

That means Store Buy/Sell can affect information-feed processing in two ways:

- indirectly through Step 3 and Step 6 updates that hit trigger-backed tables
- directly through Step 7, which upserts `store_process_info_feed`

### Stored-procedure dependencies

The workflow depends on database routines whose side effects are broader than the C# code alone shows:

- `helper_organization_save_current()`
- `organization_hierarchy_update_level()`
- `organization_hierarchy_update_ancestry()`
- `helper_delete_org_menu(<orgId>)`
- `store_process_menu_store_reset(@organizationId)`
- `store_process_dmb_store_reset(@organizationId)`

These routines are part of the business behavior of Store Buy/Sell even though their internals live in the database rather than the API codebase.

## Current audit and logging rules

### Controller and processor logs

- The controller logs and rethrows enqueue, schedule, get, cancel, and reschedule failures.
- The processor logs request-not-found, already-succeeded, step start, step completion, step failure, and final success.

### Menu Manager mutation audit

Current code writes application-driven audit events for Menu Manager mutations in all seven business steps.

Current audited targets include:

- `organization_saved`
- `organization_as_store_saved`
- `organization`
- `organization_as_store`
- `store_process_menu_tax_store_status`
- `organization_xref_ancestor`
- `organization_xref_service_location`
- `store_process_menu_action_store`
- `store_process_dmb_action_store`
- `store_process_info_feed`

This means the older follow-up note describing all step-level audit logging as missing is no longer accurate.

### Remaining audit gap

Current code does not emit application audit rows for the Menu Admin workflow lifecycle itself.

Missing workflow-level audit currently includes:

- insert of `store_buy_sell_request`
- status changes on `store_buy_sell_request`
- cancel and reschedule metadata changes
- step-log writes in `store_buy_sell_request_step_log`

## Known gaps, ambiguities, and stale-spec drift

The items below should not be treated as settled business rules.

### 1. `IsNewOrganization` only has technical meaning in code

Current enforced meaning:

- `true` means Step 2 runs
- `false` means Step 2 is skipped

The broader business meaning of when a request should be considered a "new organization" is not defined in current API code.

### 2. `EnqueueAt` lacks API validation

The schedule request does not currently enforce:

- future time
- non-default timestamp
- minimum scheduling window

Any stronger scheduling rule would be normative, not currently enforced.

### 3. Step 5 is intentionally non-transactional

This is a real business behavior choice with operational consequences.

If Step 5 fails partway through, prior Step 5 database side effects are not rolled back by the processor.

### 4. Duplicate store organization IDs can still repeat step-level work

Duplicate `Organizations[].StoreNumber` values are rejected before processing, but duplicate `Organizations[].OrganizationId` values are still accepted.

Because steps 3, 4, and 7 iterate the raw `Organizations` list, duplicate organization IDs can still repeat step-level updates.

### 5. Sub-organization ID validation is weaker than the other numeric fields

`SubOrganization.OrganizationId` is marked required but does not currently have a positive-range constraint like the other numeric identifiers.

### 6. Workflow completion fields are not populated by this code

`completed_utc`, `hangfire_state`, and `hangfire_state_utc` exist in the model and schema, but the Store Buy/Sell workflow code shown above does not populate them.

### 7. List summaries depend on the first source organization in the payload

The request list uses only the first `Organizations` element for source organization identity and name.

If payload ordering is not meaningful, the summary can be incomplete or misleading for multi-store transfers.

### 8. Older StoreBuySell specs and follow-ups contain drift

Known examples:

- older docs describe non-environment-scoped routes
- older docs use outdated column names
- older follow-up notes describe step-level audit logging as entirely missing

For current behavior, rely on live code first.

## Practical interpretation for future work

When changing Store Buy/Sell behavior, preserve these invariants unless the business rules are intentionally being changed:

- environment slug must be persisted and restored before any Menu Manager connection is used
- request and step status changes must remain resumable across Hangfire retries
- request persistence and durable dispatch-intent persistence must remain atomic for enqueue/schedule/reschedule writes
- Step 2 must stay coupled to `IsNewOrganization`
- Step 5 must be treated as a non-transactional boundary unless intentionally redesigned
- cancel/reschedule allowed-status rules must stay centralized and consistently enforced in service, SQL guards, and dispatcher-apply paths
- republish must continue to clear non-default service-location publish state and reset downstream menu-processing queues
- information-feed refresh must continue to be requested both through direct upsert behavior and any relevant trigger-backed table updates
