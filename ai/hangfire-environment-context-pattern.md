# Hangfire Environment Context Pattern

## Purpose
Use this pattern when an HTTP request targets an environment-aware Menu Manager database, but the actual work runs later in a Hangfire background job.

## Problem
Environment-aware request handling uses `IEnvironmentContextAccessor` to store the resolved environment in `AsyncLocal`. That works inside the HTTP pipeline, but the context does not flow automatically into a later Hangfire worker execution.

## Required Pattern
1. Capture the environment slug from the request route at the controller boundary.
2. Pass the environment slug explicitly through the service layer when creating the workflow request.
3. Persist the environment slug in the workflow record stored in the Menu Admin database.
4. When the Hangfire processor starts, load the workflow request first.
5. Re-resolve the persisted environment slug through `IEnvironmentRepository`.
6. Reject missing or deleted environments so execution fails closed.
7. Set `IEnvironmentContextAccessor.EnvironmentContext` before any `IMenuManagerDbConnectionFactory` call.
8. Clear `IEnvironmentContextAccessor.EnvironmentContext` in a `finally` block after execution.

## Store Buy/Sell Reference Implementation
The store buy/sell workflow now implements this pattern. Use these files as the copyable reference set:
- `Lce.MenuAdmin.Api/Controllers/MenuManagerStoreBuySellController.cs`
- `Lce.MenuAdmin.Api/Services/StoreBuySellService.cs`
- `Lce.MenuAdmin.Core/Data/Repositories/StoreBuySellWorkflowRepository.cs`
- `Lce.MenuAdmin.Core/Data/Sql/StoreBuySellWorkflowSql.cs`
- `Lce.MenuAdmin.Core/Processors/StoreBuySell/StoreBuySellProcessor.cs`
- `Lce.MenuAdmin.Core/Factories/MenuManagerDbConnectionFactory.cs`
- `Lce.MenuAdmin.Core/Infrastructure/EnvironmentContextAccessor.cs`
- `Lce.MenuAdmin.Migrations/Migrations/20260713001.CreateStoreBuySellSchema.cs`

## Concrete Flow
1. `MenuManagerStoreBuySellController` receives `{environment}` from the route.
2. `StoreBuySellService.EnqueueAsync(...)` and `ScheduleAsync(...)` accept that route value explicitly.
3. `StoreBuySellWorkflowRepository.InsertAsync(...)` persists the environment slug in `store_buy_sell_request.environment_name`.
4. Hangfire later invokes `StoreBuySellProcessor.RunAsync(...)` with only the request ID.
5. The processor loads the workflow request from Menu Admin storage.
6. The processor re-resolves `request.EnvironmentName` through `IEnvironmentRepository.GetByNameAsync(...)`.
7. If the environment is missing or deleted, the processor throws and the job fails closed.
8. If the environment resolves, the processor sets `IEnvironmentContextAccessor.EnvironmentContext` before any Menu Manager connection is requested.
9. `IMenuManagerDbConnectionFactory.GetConnection()` then resolves the correct keyed Menu Manager connection from the restored environment context.
10. The processor clears the environment context in a `finally` block so worker state does not leak across jobs.

## Why Persist The Slug Instead Of Connection Metadata
- The environment registry remains the source of truth for connection mapping.
- Jobs pick up the current environment configuration at execution time.
- The workflow record stores a stable business identifier instead of infrastructure details.
- This keeps background-job behavior aligned with the HTTP pipeline's environment resolution model.

## Rules
- Persist the environment slug, not resolved connection metadata.
- Resolve environment data again when the Hangfire job runs so the current environment registry remains authoritative.
- Do not let a Hangfire processor touch Menu Manager repositories or connections before `EnvironmentContext` is restored.
- Fail closed when the stored environment is invalid.
- Keep controller to service to repository flow explicit; do not rely on ambient `AsyncLocal` state during request creation.

## Checklist For Future Hangfire Workflows
1. Add a required environment slug field to the workflow record or durable job payload.
2. Pass the route environment explicitly from controller to service when creating the job.
3. Re-resolve the environment inside the Hangfire processor before any environment-aware repository or connection factory is used.
4. Throw on missing or deleted environments.
5. Clear `IEnvironmentContextAccessor.EnvironmentContext` in a `finally` block.
6. Add focused tests that prove persistence, rehydration-before-connection, and cleanup-after-execution.
