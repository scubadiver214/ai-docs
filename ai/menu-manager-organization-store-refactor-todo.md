# Menu Manager Organization And Store Refactor TODO

Track implementation work for splitting the mixed organization-entry API into separate Menu Manager organization and store surfaces.

## Decisions

- Remove the old `/admin/organization-entries/organization/{typeId}` endpoints; no compatibility shims are needed.
- Add separate Menu Manager routes: `/menumanager/organizations` and `/menumanager/stores`.
- Keep create and update endpoints returning response bodies.
- Store type id `MenuManagerOrganizationTypeIds.Store` remains internal to store service/repository/SQL code.
- Non-store organization requests may provide `TypeId`; validation must reject store type `MenuManagerOrganizationTypeIds.Store` and accept other non-store types.
- No database schema changes are planned.

## Route And Controller Split

- [x] Remove `OrganizationEntriesController`.
- [x] Add `MenuManagerOrganizationsController` with non-store organization GET/POST/PATCH endpoints.
- [x] Add `MenuManagerStoresController` with store GET/POST/PATCH endpoints.
- [x] Remove `typeId` from route templates.
- [x] Keep response metadata complete for 200, 400, 401, 403, 404, and 409 paths.

## Contracts

- [x] Add temporary MenuManager DTO aliases so code can compile with new public names during the physical contract move.
- [x] Create `Contracts/MenuManagerOrganizations` DTOs.
- [x] Create `Contracts/MenuManagerStores` DTOs.
- [x] Remove the combined `CreateOrganizationEntryRequest` DTO.
- [x] Rename non-store response/request names away from `OrganizationIsNotStore`.
- [x] Rename store response/request names away from `OrganizationIsStore`.
- [x] Ensure SwaggerSchema and validation attributes follow repository rules.

## Services

- [x] Add `IMenuManagerOrganizationService` / `MenuManagerOrganizationService`.
- [x] Add `IMenuManagerStoreService` / `MenuManagerStoreService`.
- [x] Move non-store lookup/create/update logic into the organization service. Completed by making `MenuManagerOrganizationService` the DI-facing implementation for organization operations while inheriting the shared behavior core.
- [x] Move store lookup/create/update/time-zone/migration/address logic into the store service. Completed by making `MenuManagerStoreService` the DI-facing implementation for store operations while inheriting the shared behavior core.
- [x] Remove controller-level branching on store versus organization.
- [x] Normalize public errors from franchise-only wording to organization/store wording where appropriate.

## Repositories

- [x] Add `IMenuManagerOrganizationRepository` / `MenuManagerOrganizationRepository`.
- [x] Add `IMenuManagerStoreRepository` / `MenuManagerStoreRepository`.
- [x] Preserve keyed MenuManager database injection.
- [x] Preserve `IScopedDependency` marker registration.
- [x] Preserve audit logging for all write paths.
- [x] Preserve inner exceptions when wrapping lower-level failures.

## SQL

- [x] Add `MenuManagerOrganizationSql` for non-store organization SQL.
- [x] Add `MenuManagerStoreSql` for store SQL.
- [x] Preserve parameterized `SqlCommandDefinition` use.
- [x] Preserve non-store type scoping and rejection of store type `MenuManagerOrganizationTypeIds.Store`.
- [x] Preserve exact store lookup by franchise/store and location.
- [x] Preserve organization and store audit snapshots.

## Middleware And Swagger

- [x] Retarget organization create request normalization middleware to `/menumanager/organizations`.
- [x] Update middleware tests for the new route.
- [x] Update Swagger static script references and tests.
- [x] Remove old inline handling for `/admin/organization-entries/organization/{typeId}` if no longer needed.

## Tests

- [x] Split controller tests into organization and store controller tests.
- [x] Split service tests into organization and store service tests.
- [x] Split repository tests into organization and store repository tests.
- [x] Split SQL coverage tests into organization and store SQL tests.
- [x] Rename store materialization tests.
- [x] Remove tests that only verify old route type branching.

## Docs And Cleanup

- [x] Update API README route examples.
- [x] Grep and remove stale public references to `organization-entries`.
- [x] Grep and remove stale implementation names after split is complete.

## Verification

- [x] Run `dotnet test --project .\Lce.MenuAdmin.Api.Tests\Lce.MenuAdmin.Api.Tests.csproj`.
- [x] Run `dotnet test --project .\Lce.MenuAdmin.Core.Tests\Lce.MenuAdmin.Core.Tests.csproj`.
- [x] Run `dotnet test --project .\Lce.MenuAdmin.Core.Data.Tests\Lce.MenuAdmin.Core.Data.Tests.csproj`.
- [x] Run `dotnet build`.
- [x] Confirm Swagger/routes no longer expose `/admin/organization-entries/organization/{typeId}`.

## Review Follow-Up Fixes

Address the code-review findings on the uncommitted split without losing logic.

- [x] Retire the transitional `IMenuManagerOrganizationStoreServiceCore` interface (fixes the double DI registration and the vestigial interface). The shared implementation is now an abstract helper base with no marker interface, so it is never DI-registered.
- [x] Keep `MenuManagerOrganizationService` / `MenuManagerStoreService` as separate concrete services over typed repositories; the shared `MenuManagerOrganizationStoreServiceCore` now contains only helper logic, so neither service leaks the other's public API surface.
- [x] Remove the `MenuManagerContractAliases.cs` bridge (API and Tests) and use the real DTO type names (`CreateMenuManagerOrganizationRequest`, `UpdateMenuManagerOrganizationRequest`, `CreateMenuManagerStoreRequest`, `MenuManagerOrganizationResponse`, `MenuManagerStoreResponse`) via normal file-level `using` directives.
- [x] Keep the data-layer `*Core` SQL/repository as the shared implementation (by design; not physically re-split) to avoid losing audit/migration/write-orchestration logic.
- [x] Re-run API, Core, and Core.Data test suites plus `dotnet build` (API 738, Core 463, Core.Data 332; build 0 warnings/0 errors).

## Controller-Scoped Cleanup

- [x] Remove store inline-field changes from `swagger-sse.js` and related Swagger UI tests/versioning.
- [x] Remove `IncidentNumberParameterOrderOperationFilter` and rely on request DTO `IncidentNumber` body binding instead of controller query parameters.
- [x] Remove `MenuManagerOrganizationCreateRequestNormalizationMiddleware` and the controller-scoped normalization attribute; organization create requests now use normal JSON model binding.
- [x] Remove redundant `incidentNumber` query parameters from organization/store create and update actions.
- [x] Split store GET into explicit `by-location-number`, `by-franchise-number`, and `by-organization-id` routes; remove the multiplexed root store GET route and store GET-by-organization-name behavior.
- [x] Update store controller tests and any Swagger/API assertions for the explicit store GET routes.
- [x] Re-run API tests and API build after the explicit store GET route split.
- [x] Split store PATCH into explicit `by-organization-name`, `by-location-number`, `by-franchise-number`/`by-store-number`, and `by-organization-id` routes; remove the multiplexed root store PATCH route.
- [x] Update store controller tests and any Swagger/API assertions for the explicit store PATCH routes.
- [x] Re-run API tests and API build after the explicit store PATCH route split.
- [x] Split organization PATCH into explicit `by-organization-name`/`by-type-id` and `by-organization-id` routes; remove the multiplexed root organization PATCH route.
- [x] Update organization controller tests and any Swagger/API assertions for the explicit organization PATCH routes.
- [x] Re-run API tests and API build after the explicit organization PATCH route split.
- [x] Re-run affected API tests and build.

## Physical Organization And Store Split

- [x] Split store and organization request models so store update paths use a store-owned request type.
- [x] Trace every candidate model field through controller, service, repository, SQL builders, SQL parameters/text, and tests before removing it.
- [x] Remove temporary mixed-core compatibility overloads after service core tests are split to store-owned request types.
- [x] Refactor services so organization and store implementations own their behavior and share only abstract-base logic.
	- [x] Move concrete organization/store service constructors to `IMenuManagerOrganizationRepository` and `IMenuManagerStoreRepository` instead of the mixed repository core interface.
	- [x] Convert shared service core to an abstract base inherited by concrete organization/store services.
	- [x] Move organization behavior into `MenuManagerOrganizationService` partial implementation.
	- [x] Move store behavior into `MenuManagerStoreService` partial implementation using `UpdateMenuManagerStoreRequest` directly on store update paths.
	- [x] Remove temporary typed-repository adapters from production service code after service behavior was physically split out of the mixed core.
- [x] Refactor repositories so organization and store implementations own their behavior and share only abstract-base logic.
- [x] Move organization/store repository read methods to direct SQL execution instead of delegating through the mixed repository core.
- [x] Move non-store organization repository create/update writes to `MenuManagerOrganizationRepository` with existing audit and transaction timing preserved.
- [x] Remove mixed repository core delegation from `MenuManagerOrganizationRepository`; remaining mixed repository core delegation is store write orchestration only.
- [x] Move store repository create/update orchestration to `MenuManagerStoreRepository` with migration, StoreOwner, unique-constraint, transaction, and four-table audit behavior preserved.
	- [x] Repoint existing store create/update orchestration tests to `MenuManagerStoreRepository` so behavior is covered before implementation is moved.
	- [x] Remove mixed repository core delegation from `MenuManagerStoreRepository`; store create/update orchestration now lives in the store repository.
	- [x] Remove `IScopedDependency` marker from the mixed repository core so it is not DI-registered.
- [x] Re-run Core and Core.Data tests after moving repository read methods to direct SQL execution.
- [x] Move real SQL builders into separate organization and store SQL files instead of delegating through a mixed core.
	- [x] Move organization read SQL builders into `MenuManagerOrganizationSql`.
	- [x] Move store read SQL builders into `MenuManagerStoreSql`.
	- [x] Move organization write SQL builders into `MenuManagerOrganizationSql` after organization repository write extraction.
	- [x] Move audit-sensitive write SQL builders after repository write orchestration is physically split.
		- [x] Move store create/update support SQL, StoreOwner SQL, required-column validation SQL, and store audit snapshot SQL into `MenuManagerStoreSql`.
		- [x] Move `StageStoreMigration` and `ExecuteStoreMigration` into `MenuManagerStoreSql`; API, Core, Core.Data tests and full build passed afterward.
- [x] Update mixed core tests into organization-focused and store-focused tests.
	- [x] Retarget SQL coverage tests for moved organization/store read SQL builders.
	- [x] Repoint existing store write orchestration tests to `MenuManagerStoreRepository`.
	- [x] Add a test-only concrete wrapper for the abstract shared service base so existing behavior coverage remains executable.
	- [x] Replace direct shared-core test construction with a test-only `MenuManagerOrganizationStoreServiceHarness` that composes the concrete organization/store services.
	- [x] Keep remaining mixed repository-core mocks test-only behind the harness while concrete services are exercised through typed repository adapters.
- [x] Re-run API, Core, Core.Data tests and builds after the physical split.
	- [x] Re-run API, Core, and Core.Data tests after request, service-constructor, repository-read, and SQL-read split slices.
	- [x] Run full build after request, service-constructor, repository-read, and SQL-read split slices; build passed with pre-existing CLI nullable warnings.
	- [x] Re-run API, Core, Core.Data tests and full build after moving store repository write orchestration and store support SQL.
	- [x] Re-run API, Core, Core.Data tests and full build after moving migration SQL into `MenuManagerStoreSql`.
	- [x] Re-run API, Core, Core.Data tests and full build after moving concrete service behavior out of the mixed service core.

### Physical Split Checkpoint Notes

- Store PATCH now uses `UpdateMenuManagerStoreRequest`; every retained field was traced through store service mapping into repository/SQL behavior before copying. `OrganizationTypeId` was not copied because store update routes still supply store type `MenuManagerOrganizationTypeIds.Store` internally.
- Organization/store service constructors now depend on typed repositories (`IMenuManagerOrganizationRepository` / `IMenuManagerStoreRepository`) instead of `IMenuManagerOrganizationStoreRepositoryCore`; production service behavior now lives in concrete organization/store partial implementations, and the shared service core contains only helper logic.
- Remaining service behavior tests that still use `IMenuManagerOrganizationStoreRepositoryCore` do so through a test-only harness that composes the concrete services; no production service code depends on the mixed repository core.
- Organization/store repository read methods now execute their own SQL builders directly. Non-store organization writes live in `MenuManagerOrganizationRepository`, and store writes live in `MenuManagerStoreRepository`.
- Organization/store read SQL builders now live in `MenuManagerOrganizationSql` and `MenuManagerStoreSql`. Organization write SQL, store write support SQL, and store migration SQL also live in their respective files. Remaining `MenuManagerOrganizationStoreSqlCore` references in these split SQL files are XML doc references only, not executable delegation.

## Review Hardening Cleanup

Track the implementation work from the uncommitted-code review. Warnings and analyzer findings are treated as failing work unless there is a narrow, documented false-positive or test-intent reason.

- [x] Add a tracked checklist for review hardening work.
- [x] Capture a warning-as-error baseline before cleanup.
- [x] Fix dead assignments, unused usings, redundant casts, and nullable-pattern analyzer noise in the split services/repositories.
- [x] Apply minor cosmetic cleanup to `UpdateMenuManagerStoreRequest` without changing wire names or store-owned behavior.
- [x] Remove production pragma suppressions where usage can be corrected.
- [x] Replace leaky transitional repository write signatures with typed organization/store command records so `CancellationToken` can remain last.
- [x] Delete old mixed repository and SQL core types after tests are retargeted to typed repositories/SQL builders.
- [x] Remove test-only mixed-core adapters after service tests mock typed repositories directly.
- [x] Remove triplicated repository/service orchestration by deleting the obsolete mixed core; remaining organization/store write orchestration stays in typed repositories to preserve domain-specific behavior.
- [x] Preserve controller-level behavior and metadata for explicit Menu Manager organization/store routes.
- [x] Preserve SQL-level behavior for parameterized commands, type scoping, exact store lookups, audit snapshots, StoreOwner support, required-column validation, and migration execution.
- [x] Keep Menu Manager organization type ids centralized in `MenuManagerOrganizationTypeIds` and avoid reintroducing inline numeric literals in code, SQL assertions, or docs.
- [x] Run API, Core, and Core.Data tests plus Debug/Release warning-as-error builds.
- [x] Grep for remaining pragmas, stale mixed-core symbols, and old `organization-entries` route references; document any intentional retained suppressions.

### Review Hardening Checkpoint Notes

- Debug and Release builds pass with `-warnaserror` after the cleanup.
- API, Core, and Core.Data tests pass after deleting the obsolete mixed repository/SQL/service-core test harnesses (API 661, Core 463, Core.Data 332).
- `MenuManagerOrganizationStoreSqlCore`, `MenuManagerOrganizationStoreRepositoryCore`, `IMenuManagerOrganizationStoreRepositoryCore`, and `MenuManagerOrganizationStoreServiceCore` no longer exist in C# code.
- Organization/store repositories now expose typed create/update command records so cancellation tokens remain last and store-only fields do not leak through organization write contracts.
- Store and organization SQL builders remain split; all executable SQL tests use `MenuManagerOrganizationSql` or `MenuManagerStoreSql` directly.
- Remaining `#pragma warning disable` entries are intentional and narrow: CS8601 null-injection tests, CA2201 base-exception behavior test, and CA1711 domain-name false positives for `MenuManagerPermission` records.
- Remaining `organization-entries` references are historical TODO context only; no C# or route implementation references remain.

## Review Fixes, Factory Migration, And Coverage Hardening

Track the follow-up implementation from the 2026-07-08 current-branch review and coverage pass.

- [x] Fix invalid Menu Manager organization/store client input so documented 400 paths no longer fall through to 500.
- [x] Fix out-of-band audit timing consistency for no-caller-transaction organization/store update paths.
- [x] Fix store-scoped service error messages that still use franchise wording.
- [x] Make NGCV migration date boundary semantics explicit and cover yesterday/today/tomorrow/sentinel behavior.
- [x] Replace created-organization readback lookup with deterministic insert-returning SQL.
- [x] Migrate `MenuManagerOrganizationRepository` to `IMenuManagerDbConnectionFactory`.
- [x] Migrate `MenuManagerStoreRepository` to `IMenuManagerDbConnectionFactory`.
- [x] Add focused tests for `MenuManagerOrganizationTypeOptionResponse`.
- [x] Add focused tests for `MenuManagerOrganizationService` behavior.
- [x] Add focused tests for `MenuManagerOrganizationStoreServiceHelpers` behavior.
- [x] Add focused tests for `MenuManagerStoreService` behavior.
- [x] Add focused tests for `OrganizationTimeZoneRecord`.
- [x] Run API, Core, Core.Data tests and build after all fixes.