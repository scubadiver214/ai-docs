# Menu Manager Organizations and Stores Business Rules

This document captures the current backend API business rules for manipulating Menu Manager organizations and stores. The backend service and repository layers are the source of truth. Frontend mock behavior and store-transfer scheduling behavior are outside this document's scope.

## Scope and Terms

- Organization endpoints manage non-store `organization` rows for create, update, and type-scoped name lookup. The organization read-by-id path returns the repository match for the supplied id and does not apply a store-type filter in SQL.
- Store endpoints manage store organizations and related store tables. Store organization type `MenuManagerOrganizationTypeIds.Store` is applied internally by store endpoints.
- Franchise organization lookup uses organization type `MenuManagerOrganizationTypeIds.Franchise`.
- The current API mutation surface is create and update. Reads are included where lookup behavior affects mutation targeting.
- Routes are environment-scoped under `api/v{version}/{environment}/menumanager`.

## Source of Truth

| Area | Source |
| --- | --- |
| Organization routes and controller validation | [MenuManagerOrganizationsController.cs](../../Lce.MenuAdmin.Api/Controllers/MenuManagerOrganizationsController.cs) |
| Store routes and controller validation | [MenuManagerStoresController.cs](../../Lce.MenuAdmin.Api/Controllers/MenuManagerStoresController.cs) |
| Organization service rules | [MenuManagerOrganizationService.Organizations.cs](../../Lce.MenuAdmin.Api/Services/MenuManagerOrganizationService.Organizations.cs) |
| Store service rules | [MenuManagerStoreService.Stores.cs](../../Lce.MenuAdmin.Api/Services/MenuManagerStoreService.Stores.cs) |
| Shared parsing and conflict helpers | [MenuManagerOrganizationStoreServiceHelpers.cs](../../Lce.MenuAdmin.Api/Services/MenuManagerOrganizationStoreServiceHelpers.cs) |
| Organization request contracts | [CreateMenuManagerOrganizationRequest.cs](../../Lce.MenuAdmin.Api/Contracts/MenuManagerOrganizations/CreateMenuManagerOrganizationRequest.cs), [UpdateMenuManagerOrganizationRequest.cs](../../Lce.MenuAdmin.Api/Contracts/MenuManagerOrganizations/UpdateMenuManagerOrganizationRequest.cs) |
| Store request contracts | [CreateMenuManagerStoreRequest.cs](../../Lce.MenuAdmin.Api/Contracts/MenuManagerStores/CreateMenuManagerStoreRequest.cs), [UpdateMenuManagerStoreRequest.cs](../../Lce.MenuAdmin.Api/Contracts/MenuManagerStores/UpdateMenuManagerStoreRequest.cs) |
| Store defaults | [MenuManagerStoreDefaults.cs](../../Lce.MenuAdmin.Core/Constants/MenuManagerStoreDefaults.cs) |
| Organization persistence and audit | [MenuManagerOrganizationRepository.cs](../../Lce.MenuAdmin.Core/Data/Repositories/MenuManagerOrganizationRepository.cs) |
| Store persistence, orchestration, and audit | [MenuManagerStoreRepository.cs](../../Lce.MenuAdmin.Core/Data/Repositories/MenuManagerStoreRepository.cs) |
| Audit policy | [menu-manager-logged-actions-audit-spec.md](../specs/menu-manager-logged-actions-audit-spec.md) |
| Database catalog | [database-schema.md](database-schema.md) |

## Shared Rules

### Canonical Organization Type IDs

The codebase centralizes Menu Manager organization type ids in [Lce.MenuAdmin.Core/Constants/MenuManagerOrganizationTypeIds.cs](../../Lce.MenuAdmin.Core/Constants/MenuManagerOrganizationTypeIds.cs). Use those named constants in code, SQL builders, and tests instead of reintroducing numeric literals.

Current canonical values:
- See [Lce.MenuAdmin.Core/Constants/MenuManagerOrganizationTypeIds.cs](../../Lce.MenuAdmin.Core/Constants/MenuManagerOrganizationTypeIds.cs) for the full current list.
- This document references named constants (for example `MenuManagerOrganizationTypeIds.Store`) instead of restating numeric values.

When adding or updating Menu Manager organization/store logic, prefer `MenuManagerOrganizationTypeIds.*` over inline magic numbers. If a doc or test needs to mention the value for explanation, pair it with the named constant.

### Authorization

| Operation | Policy |
| --- | --- |
| Organization reads | `AuthPolicies.Permissions.MenuManagerRead` |
| Store reads | `AuthPolicies.Permissions.MenuManagerRead` |
| Organization create/update | `AuthPolicies.Permissions.MenuManagerEdit` |
| Store create/update | `AuthPolicies.Permissions.MenuManagerEdit` |

Protected endpoints declare body-less `401` and `403` responses at the controller level. Validation failures produce `400`; missing targets produce `404`; duplicate or unchanged data conflicts produce `409`.

### Environment Scoping

Every organization and store route includes `{environment}`. The environment route value is documented and required by the controllers, but business logic for organization and store manipulation is implemented in the service and repository layers rather than branching directly on the route value.

### Type Boundaries

- Store organization type `MenuManagerOrganizationTypeIds.Store` is reserved for store endpoints.
- Organization endpoints reject store type `MenuManagerOrganizationTypeIds.Store` on create, lookup by name/type, and update.
- Organization read-by-id has no type input and does not reject store type `MenuManagerOrganizationTypeIds.Store` before returning a match.
- Store endpoints hide the type id from callers and force store type `MenuManagerOrganizationTypeIds.Store` internally.
- Franchise lookups use organization type `MenuManagerOrganizationTypeIds.Franchise` internally.
- Store update and create fail if store type `MenuManagerOrganizationTypeIds.Store` is unavailable or is not marked as a store type.

### Positive Numeric Inputs

Route identifiers and required numeric body identifiers must be greater than zero unless the contract explicitly allows zero.

- `OrganizationId` on create may be `null`, `0`, or a positive value. `null` and `0` mean the next highest unused organization identifier is assigned.
- `OrganizationId` less than zero is rejected.
- Store create requires `LocationNumber` to be supplied; repository required-column validation decides whether the final persisted store is valid.

### Country Resolution

Store mutation requests derive country from `parentOrganizationId`.

- Store POST does not accept `country`; the parent organization's country resolves through the existing organization lookup.
- Store PATCH does not accept `country` or `rootCountryId`.
- When PATCH supplies `parentOrganizationId`, its country resolves through the parent lookup and existing root-country lookup. The derived root-country id and country are persisted internally.
- When PATCH omits `parentOrganizationId`, existing country and root-country linkage remain unchanged.
- A legacy JSON `country` property is rejected with `400`; unrelated unmapped JSON properties retain existing extension-data behavior.
- If the parent organization or its root-country option does not exist, the request is rejected with `400`.
- Store response country fields remain available on read endpoints.

### Name and Number Parsing

Organization and store name lookups have special numeric parsing rules.

| Input shape | Behavior |
| --- | --- |
| Organization name is digits only, including leading zeroes | Treated as a franchise-number search scoped to the supplied organization type. |
| Organization name contains non-digit text | Treated as a type-scoped organization-name search. |
| Store organization name parses as `digits-digits` | Treated as exact franchise/store lookup for store update by organization name. |
| Store organization name does not parse as `digits-digits` | Treated as store organization-name lookup for update by organization name. |

Name-based update operations must resolve exactly one target organization id. Zero matches produce `404`. More than one match produces `400` and requires callers to update by organization id instead.

### Collision Detection

Exact organization-name duplicates are rejected. The services also check embedded numeric tokens of at least three digits in organization names and reject whole-token number collisions in the same organization type.

Examples of enforced collision categories:

- Organization create/update rejects an exact organization-name duplicate within the effective organization type.
- Organization create/update rejects a collision when an embedded number token in the requested name matches an existing organization's number token in the same type.
- Store create rejects an exact store organization-name duplicate.
- Store create rejects an existing franchise/store number pair.
- Store update rejects store-name collisions when renaming.
- Store repository unique-constraint wrappers convert duplicate Deliverect location id, location number, or franchise/store number failures into `409` conflicts.

### Incident Numbers

`IncidentNumber` is accepted on create and update requests with a maximum length of 100 characters. It is recorded on the audit event only. It is never used in Menu Manager write SQL.

### Audit Rules

All organization and store write paths log audit events through application code.

- Trigger-based auditing is not used.
- Audit calls are made after successful data mutation.
- Audit failures are best effort: failures are logged by the audit path and do not fail the main business operation.
- Organization and store mutations log against schema `public`, table `organization`.
- Store create/update audit payloads flatten related store tables into the audit data so one audit event represents the store change.

## Organization Rules

Organization endpoints are rooted at:

```text
api/v{version}/{environment}/menumanager/organizations
```

### Read by Organization Id

| Rule | Value |
| --- | --- |
| Route | `GET by-organization-id/{organizationId:int}` |
| Authorization | `MenuManagerRead` |
| Lookup | Exact organization id; no store-type filter is applied in SQL. |
| Validation | `organizationId` must be greater than zero. |
| Success | `200` with `MenuManagerOrganizationResponse`. |
| Missing target | `404` with `Franchise not found.` from the service. |

The endpoint is part of the organization surface, but the current repository lookup does not reject store type `MenuManagerOrganizationTypeIds.Store` for this read path. Store details should still be read through the stores endpoint.

### Read by Organization Name and Type

| Rule | Value |
| --- | --- |
| Route | `GET by-organization-name/{organizationName}/by-type-id/{typeId:int}` |
| Authorization | `MenuManagerRead` |
| Lookup | Numeric names use franchise-number paging; non-numeric names use type-scoped name paging. |
| Paging | `page` defaults to `1`; `pageSize` defaults to `50`. |
| Validation | `organizationName` must be provided; `typeId` must be greater than zero and cannot be store type `MenuManagerOrganizationTypeIds.Store`. |
| Success | `200` with `PagedResult<MenuManagerOrganizationResponse>`. |
| No matches | `404` with `Franchise not found.` from the service. |

Paging metadata is returned from the database-side paged result and preserved in the API response.

### Read by Franchise Number

| Rule | Value |
| --- | --- |
| Route | `GET by-franchise-number/{franchiseNumber:int}` |
| Authorization | `MenuManagerRead` |
| Lookup | Franchise number search using fixed organization type `MenuManagerOrganizationTypeIds.Franchise`. |
| Validation | `franchiseNumber` must be greater than zero. |
| Success | `200` with `MenuManagerOrganizationResponse`. |
| No matches | `404` with `Franchise not found.` from the service. |

The franchise-number match is digit-bounded and resolves a single organization.

### Create Organization

| Rule | Value |
| --- | --- |
| Route | `POST` |
| Authorization | `MenuManagerEdit` |
| Body | `CreateMenuManagerOrganizationRequest` JSON. |
| Success | `200` with `MenuManagerOrganizationResponse`. |
| Duplicate | `409`. |
| Validation failure | `400`. |

Create rules:

- `OrganizationId` may be `null`, `0`, or positive. Positive ids are checked for duplicates before insert.
- `ParentOrganizationId` is required and must be greater than zero.
- `Country` is required, limited to 50 characters, and must resolve to a root country.
- `OrganizationName` is required, limited to 100 characters, and cannot be blank or whitespace.
- `OrganizationTypeId` is required, must be greater than zero, must exist, and must not be store type `MenuManagerOrganizationTypeIds.Store`.
- Store organization types are rejected with guidance to use the stores endpoint.
- Exact name and embedded-number collisions for the requested type are rejected.
- `HideFromMenuManager` defaults to `false` when omitted.
- `FranConnectManaged` defaults through database behavior when omitted; the contract describes the default as `true`.
- `IncidentNumber` is audit-only.

The repository inserts the organization and logs an insert audit event for `public.organization`.

### Update Organization by Organization Id

| Rule | Value |
| --- | --- |
| Route | `PATCH by-organization-id/{organizationId:int}` |
| Authorization | `MenuManagerEdit` |
| Body | `UpdateMenuManagerOrganizationRequest` JSON. |
| Success | `200` with `MenuManagerOrganizationResponse`. |
| Missing target | `404`. |
| Duplicate values | `409`. |
| Validation failure | `400`. |

Update rules:

- `organizationId` must be greater than zero.
- The current row must exist.
- The current row must not be store type `MenuManagerOrganizationTypeIds.Store`.
- All request fields are optional.
- Supplied `Active` updates the organization active state.
- Supplied `ParentOrganizationId` and `OrganizationTypeId` must be greater than zero.
- Supplied `OrganizationName` cannot be blank or whitespace.
- Supplied `Country` must resolve to a root country.
- Supplied `OrganizationTypeId` must exist and must not be a store type.
- If `OrganizationName` is supplied, collision detection runs against the effective type id. The effective type id is the supplied type id when present, otherwise the current type id.
- If `ParentOrganizationId` is supplied, the parent organization must exist and the response hierarchy level is recomputed as parent hierarchy level plus one.
- PATCH no-op payloads are accepted; omitted and unchanged supplied values resolve to current persisted values.
- The repository updates the organization, refreshes hierarchy data, and logs an update audit event for `public.organization`.

The service response is assembled from current data plus supplied changes; the repository is responsible for persistence and audit.

### Update Organization by Organization Name and Type

| Rule | Value |
| --- | --- |
| Route | `PATCH by-organization-name/{organizationName}/by-type-id/{typeId:int}` |
| Authorization | `MenuManagerEdit` |
| Body | `UpdateMenuManagerOrganizationRequest` JSON. |
| Target resolution | Name/type lookup resolves to exactly one organization id, then uses the same rules as update by id. |

Additional rules:

- `organizationName` must be provided.
- `typeId` must be greater than zero and cannot be store type `MenuManagerOrganizationTypeIds.Store`.
- Numeric names are treated as franchise-number lookups scoped to `typeId`.
- Non-numeric names are treated as organization-name lookups scoped to `typeId`.
- Zero matches produce `404`.
- Multiple matches produce `400` and require update by organization id.

## Store Rules

Store endpoints are rooted at:

```text
api/v{version}/{environment}/menumanager/stores
```

### Read by Location Number

| Rule | Value |
| --- | --- |
| Route | `GET by-location-number/{locationNumber:int}` |
| Authorization | `MenuManagerRead` |
| Lookup | Exact location-number match. |
| Validation | `locationNumber` must be greater than zero. |
| Success | `200` with `MenuManagerStoreResponse`. |
| Missing target | `404` with `Store not found.` |

Location-number lookup is exact. For example, `12689` targets only location `12689`.

### Read by Franchise and Store Number

| Rule | Value |
| --- | --- |
| Route | `GET by-franchise-number/{franchiseNumber:int}/by-store-number/{storeNumber:int}` |
| Authorization | `MenuManagerRead` |
| Lookup | Exact franchise/store number pair. |
| Validation | Both values must be supplied and greater than zero. |
| Success | `200` with `MenuManagerStoreResponse`. |
| Missing target | `404` with `Store not found.` |

### Read by Organization Id

| Rule | Value |
| --- | --- |
| Route | `GET by-organization-id/{organizationId:int}` |
| Authorization | `MenuManagerRead` |
| Lookup | Exact store organization id. |
| Validation | `organizationId` must be greater than zero. |
| Success | `200` with `MenuManagerStoreResponse`. |
| Missing target | `404` with `Store not found.` |

### Create Store

| Rule | Value |
| --- | --- |
| Route | `POST` |
| Authorization | `MenuManagerEdit` |
| Body | `CreateMenuManagerStoreRequest` JSON. |
| Success | `200` with persisted `MenuManagerStoreResponse`. |
| Duplicate | `409`. |
| Validation failure | `400`. |

Create rules:

- `OrganizationId` may be `null`, `0`, or positive. Positive ids are checked for existing stores before insert.
- `ParentOrganizationId` is required and must be greater than zero.
- `Country` is required, limited to 50 characters, and must resolve to a root country.
- Parent organization must exist and its country must match the resolved store country.
- If the parent organization name contains any digit, the parent organization name must contain that franchise number as a whole token.
- `OrganizationName` is required, limited to 100 characters, and cannot be blank or whitespace.
- Store organization type `MenuManagerOrganizationTypeIds.Store` is forced internally and must exist as a store type.
- `LocationNumber` is required for store creation.
- `NgcvMenuMigrationDate` cannot be before the current UTC date.
- If `NgcvMenuMigrationDate` is omitted, it defaults to `2100-01-01`, representing migration not scheduled.
- `StoreTimeZoneId` must resolve to a time zone for the store's root country.
- If any store address field is supplied on create, all of `MainDivision`, `Address1`, `City`, `Province`, and `PostalCode` must be supplied and nonblank.
- Exact store organization-name duplicates are rejected.
- Existing franchise/store number pairs are rejected.
- `HideFromMenuManager` defaults to `false` when omitted.
- `FranConnectManaged` defaults through database behavior when omitted; the contract describes the default as `true`.
- `IncidentNumber` is audit-only.

Create orchestration rules:

- Store metadata is written across `organization`, `organization_as_store`, `organization_as_store_attribute`, and optionally `organization_as_store_address`.
- `ShowCalories` is applied to `dmb_show_calories`, `online_show_calories_on_menu`, and `online_show_calories_on_cart`.
- `IsLoadBalance` is persisted consistently across store metadata tables.
- `ngcv_menu_migrated` is initialized as `false`.
- `ngcv_menu_migration_staged` is derived from whether the migration date differs from the not-scheduled sentinel.
- If the migration date is not the sentinel, migration staging is executed.
- If the migration date is the current UTC date, migration execution is also run.
- Required store columns are validated after orchestration; missing values produce `400` with the missing column list.
- The response is read back from persisted store data.

Default StoreOwner creation rules:

- No default StoreOwner is created when another store already exists for the franchise.
- No default StoreOwner is created when the parent is a sub-franchise of the company.
- No default StoreOwner is created when a menu-root default language cannot be resolved.
- When eligible, a default user is created with user id `<franchise padded to four digits>0000`, first name `Franchise`, last name equal to the franchise number, group id `StoreOwner`, and the resolved default language.

The repository logs an insert audit event for `public.organization` with flattened store data and any created StoreOwner data.

### Update Store by Organization Id

| Rule | Value |
| --- | --- |
| Route | `PATCH by-organization-id/{organizationId:int}` |
| Authorization | `MenuManagerEdit` |
| Body | `UpdateMenuManagerStoreRequest` JSON. |
| Success | `200` with persisted `MenuManagerStoreResponse`. |
| Missing target | `404`. |
| Duplicate or unchanged values | `409`. |
| Validation failure | `400`. |

Update rules:

- `organizationId` must be greater than zero.
- The target store must exist.
- All request fields are optional.
- Supplied `ParentOrganizationId` must be greater than zero.
- Supplied `OrganizationName` cannot be blank or whitespace.
- Supplied `NgcvMenuMigrationDate` cannot be before the current UTC date.
- Supplied `Country` must resolve to a root country.
- Supplied `ParentOrganizationId` must exist and match the effective store country.
- If parent organization or franchise number changes, the effective parent/franchise combination must pass the parent-name/franchise-token rule.
- Store organization type `MenuManagerOrganizationTypeIds.Store` must exist and be marked as a store type.
- Supplied `OrganizationName` is checked for exact and embedded-number store collisions.
- Supplied `StoreTimeZoneId` must resolve to a time zone for the effective root country.
- The request is rejected with `409` if any supplied field already matches current data.
- Store menu time-zone offsets are recalculated when a time zone is supplied.
- If a migration date is supplied and is not the not-scheduled sentinel, migration staging is executed.
- If the supplied migration date is the current UTC date, migration execution is also run.
- The response is read back from persisted store data.

Store address update behavior:

- Address fields are optional on update.
- Supplying any address field causes the store address row to be upserted.
- Update does not require the complete address set that create requires.
- Supplied address fields that already match current data trigger the shared unchanged-value conflict rule.

The repository logs an update audit event for `public.organization` with flattened store data.

### Update Store by Organization Name

| Rule | Value |
| --- | --- |
| Route | `PATCH by-organization-name/{organizationName}` |
| Authorization | `MenuManagerEdit` |
| Body | `UpdateMenuManagerStoreRequest` JSON. |
| Target resolution | Organization name resolves to exactly one store id, then uses the same rules as update by id. |

Additional rules:

- `organizationName` must be provided.
- Values that parse as `digits-digits` are treated as exact franchise/store lookup.
- Other values are treated as store organization-name lookup.
- Zero matches produce `404`.
- Multiple matches produce `400` and require update by organization id.

### Update Store by Location Number

| Rule | Value |
| --- | --- |
| Route | `PATCH by-location-number/{locationNumber:int}` |
| Authorization | `MenuManagerEdit` |
| Body | `UpdateMenuManagerStoreRequest` JSON. |
| Target resolution | Exact location-number lookup, then update by resolved organization id. |

Additional rules:

- `locationNumber` must be greater than zero.
- Missing target produces `404`.
- After target resolution, update by organization id rules apply.

### Update Store by Franchise and Store Number

| Rule | Value |
| --- | --- |
| Route | `PATCH by-franchise-number/{franchiseNumber:int}/by-store-number/{storeNumber:int}` |
| Authorization | `MenuManagerEdit` |
| Body | `UpdateMenuManagerStoreRequest` JSON. |
| Target resolution | Exact franchise/store lookup, then update by resolved organization id. |

Additional rules:

- Both `franchiseNumber` and `storeNumber` must be supplied and greater than zero.
- Missing target produces `404`.
- After target resolution, update by organization id rules apply.

## Field-Level Rules

### Organization Create Fields

| Field | Rule |
| --- | --- |
| `OrganizationId` | Optional; `null` or `0` auto-assigns the next highest unused id; positive values must not already exist; negative values rejected. |
| `ParentOrganizationId` | Required and greater than zero. |
| `Country` | Required; max length 50; must resolve to root country. |
| `OrganizationName` | Required; max length 100; cannot be blank; checked for exact and embedded-number collisions. |
| `OrganizationTypeId` | Required; greater than zero; must exist; cannot be store type. |
| `HideFromMenuManager` | Optional; defaults to `false` when omitted. |
| `FranConnectManaged` | Optional; database default applies when omitted. |
| `IncidentNumber` | Optional; max length 100; audit-only. |

### Organization Update Fields

| Field | Rule |
| --- | --- |
| `ParentOrganizationId` | Optional; when supplied, greater than zero and must exist; response hierarchy level is recomputed from parent. |
| `Country` | Optional; max length 50; when supplied, must resolve to root country. |
| `OrganizationName` | Optional; max length 100; when supplied, cannot be blank and is checked for collisions. |
| `OrganizationTypeId` | Optional; greater than zero; must exist; cannot be store type. |
| `HideFromMenuManager` | Optional; unchanged supplied values are rejected. |
| `FranConnectManaged` | Optional; unchanged supplied values are rejected. |
| `IncidentNumber` | Optional; max length 100; audit-only. |

### Store Identity and Relationship Fields

| Field | Create Rule | Update Rule |
| --- | --- | --- |
| `OrganizationId` | Optional; `null` or `0` auto-assigns; positive values must not already exist; negative values rejected. | Targeted by route, not body. |
| `ParentOrganizationId` | Required; greater than zero; parent must exist; parent country must match store country. | Forbidden. |
| `Country` | Required; max length 50; must resolve to root country. | Forbidden and is used in parent/time-zone validation. |
| `OrganizationName` | Required; max length 100; cannot be blank; exact duplicates rejected. | Forbidden. |
| `FranchiseNumber` | Required; if parent name contains digits, parent name must contain the franchise number as a whole token. | Forbidden. |
| `StoreNumber` | Required; used with franchise number for duplicate detection when both are supplied. | Forbidden. |
| `LocationNumber` | Required by service on create. | Forbidden. |
| `Active` | Not accepted on create request. | Optional; updates store active state. |

### Store Time Zone and Migration Fields

| Field | Rule |
| --- | --- |
| `StoreTimeZoneId` | Required; greater than zero; must resolve to a time zone valid for the effective root country; persisted as Windows name, Unix name, and UTC offset. |
| `NgcvMenuMigrationDate` | Optional; cannot be before current UTC date. |
| Omitted create migration date | Defaults to `2100-01-01`, meaning not scheduled. |
| Migration staged flag | Derived internally; callers do not supply it. |
| Migration execution | If supplied date is not the sentinel, staging runs; if the date is current UTC date, execution also runs. |

### Store Integration and Feature Fields

| Field | Rule |
| --- | --- |
| `DeliverectLocationId` | Optional; duplicate database values are converted to conflict responses. |
| `DeliverectBasePriceApplicable` | Optional; persisted on the store row. |
| `IsMike` | Optional; persisted on store attribute data. |
| `DmbStore` | Optional; persisted on store attribute data. |
| `ShowCalories` | Optional; applied to DMB calories and both online calories flags. |
| `DmbShowSodium` | Optional; persisted on store attribute data. |
| `IsLoadBalance` | Optional; persisted consistently across store metadata. |
| `IsNonTraditional` | Optional; persisted on store attribute data. |
| `ParticipateInDigitalHnr` | Optional; persisted on store attribute data. |
| `ParticipateInDigitalHnrNoScan` | Optional; persisted on store attribute data. |
| `DmbDriveThroughStore` | Optional; persisted on store attribute data. |

### Store Address Fields

| Field Set | Rule |
| --- | --- |
| `MainDivision`, `Address1`, `City`, `Province`, `PostalCode` on create | If any field is supplied, all must be supplied and nonblank. |
| Same fields on update | Each field is optional; supplying any address field upserts the address row. |
| Address `Country` | Derived from the resolved root country when store country changes. |
| Unchanged address field values | Rejected through the shared unchanged-value conflict rule when supplied. |

### Store Visibility and Audit Fields

| Field | Rule |
| --- | --- |
| `HideFromMenuManager` | Optional; create defaults to `false` when omitted. |
| `FranConnectManaged` | Optional; database default applies when omitted. |
| `IncidentNumber` | Optional; max length 100; audit-only. |

## Persistence and Audit Details

### Organization Persistence

Organization create and update write to `organization`. Update operations refresh hierarchy state after successful changes.

Relevant database objects from the schema catalog include:

- `organization`
- `organization_type`
- `organization_xref_ancestor`
- `organization_xref_children`
- `organization_as_menu_root`

### Store Persistence

Store create and update coordinate multiple physical tables:

- `organization`
- `organization_as_store`
- `organization_as_store_attribute`
- `organization_as_store_address`

Store creation and update can also invoke routines that stage and execute NGCV migration, update store menu time-zone offsets, save current organization state, and refresh organization hierarchy data.

### StoreOwner Persistence

Eligible first stores for a franchise may create default StoreOwner data in:

- `app_permission_user`
- `app_permission_user_group`
- `app_permission_user_xref_language`

These StoreOwner rows are included in the flattened store-create audit payload when created.

### Audit Event Shape

Audit events are built with `AuditEventBuilder` and sent through `IAuditLogger`.

| Mutation | Audit action | Audit schema/table | Audit data |
| --- | --- | --- | --- |
| Organization create | Insert | `public.organization` | Created organization response record. |
| Organization update | Update | `public.organization` | Before and after organization snapshots. |
| Store create | Insert | `public.organization` | Flattened organization/store/address/attribute data and eligible StoreOwner data. |
| Store update | Update | `public.organization` | Before and after flattened store snapshots. |

## Verification

Use the following checks when changing these rules or validating this document against the current branch.

```powershell
dotnet test --project .\Lce.MenuAdmin.Api.Tests\Lce.MenuAdmin.Api.Tests.csproj
dotnet test --project .\Lce.MenuAdmin.Core.Tests\Lce.MenuAdmin.Core.Tests.csproj
```

For focused service validation, use Microsoft Testing Platform runner arguments rather than `dotnet test --filter`:

```powershell
dotnet test --project .\Lce.MenuAdmin.Api.Tests\Lce.MenuAdmin.Api.Tests.csproj -- --filter-class Lce.MenuAdmin.Api.Tests.Services.MenuManagerOrganizationServiceTests
dotnet test --project .\Lce.MenuAdmin.Api.Tests\Lce.MenuAdmin.Api.Tests.csproj -- --filter-class Lce.MenuAdmin.Api.Tests.Services.MenuManagerStoreServiceTests
dotnet test --project .\Lce.MenuAdmin.Api.Tests\Lce.MenuAdmin.Api.Tests.csproj -- --filter-class Lce.MenuAdmin.Api.Tests.Services.MenuManagerOrganizationStoreServiceHelpersTests
```

Run these commands from the API repository root: `c:\source\lce-menu-admin-api`.