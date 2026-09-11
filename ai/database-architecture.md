# Database architecture guide

Generated from local PostgreSQL profile localhost on 2026-06-26 using pgsql MCP tools against databases master-menu and menu-publisher.

This guide explains how the two local PostgreSQL databases fit the API and how to navigate them without relying on foreign keys. For exhaustive object lists, use [database-schema.md](database-schema.md). For functions, procedures, trigger functions, dependencies, and side-effect notes, use [database-routines.md](database-routines.md). For model-readable flow charts, use [database-diagrams.md](database-diagrams.md).

## Database split

| Database | Runtime connection | Main purpose | Common object families |
| --- | --- | --- | --- |
| menu-publisher | MMData/default Dapper route | Organization-facing menu publishing, item/topping/crust/special reads and org-level updates | corp_menu_*, org_menu_*, common_menu_*, organization_*, api_get_*, api_update_org_* |
| master-menu | MMMData/MenuCreationDatabaseSpecification | Menu creation, amendments, validation, deployment/status, permission-aware edit views | menu_create_*, app_permission_*, common_menu_*, organization_xref_menu_root, api_menu_create_* |

The API uses Lce.MenuManager.Core/Helper/DapperWrapper.cs to choose the database connection. Calls using MenuCreationDatabaseSpecification go to master-menu; default/shared calls generally go to menu-publisher.

## Object inventory (verified 2026-06-26)

Counts come from the live localhost catalogs in the public schema. Routine totals split into non-trigger functions, procedures, and trigger functions.

| Database | Tables | Views | Sequences | Triggers | Functions | Procedures | Trigger functions | Standalone user types |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| master-menu | 176 | 4 | 1 | 5 | 129 | 45 | 2 | 7 |
| menu-publisher | 265 | 3 | 15 | 8 | 344 | 192 | 5 | 7 |

Neither database has materialized views or enum/domain/range types in the public schema. The standalone user types are the seven composites api_json_data_type, deliverect_pricing_data_type, image_file_metadata_data_type, org_svl_data_type, and the tablefunc extension types tablefunc_crosstab_2/3/4. All other composite types are table or view row types and are excluded from the schema catalog.

## Relationship strategy

The local schema does not rely on foreign keys for navigation. Infer relationships from:

- matching identifier names such as organization_id, language_id, service_location_id, item_id, display_group_id, special_id, data_type_id, and data_id;
- *_xref_* tables, especially organization hierarchy and menu-root mapping tables;
- joins inside functions, procedures, trigger functions, and views;
- C# repository/service call sites and database specification selection;
- migration SQL under sql/ when the live routine body is not enough context.

Use confidence labels when adding new relationship notes:

| Confidence | Meaning |
| --- | --- |
| strong | Repeated join/routine evidence or a catalog dependency plus code usage exists |
| medium | Naming convention plus one routine, view, or repository usage exists |
| weak | Naming convention only; verify before using for code changes |

## Core domains

### Organization and menu-root hierarchy

organization, organization_as_menu_root, organization_xref_ancestor, organization_xref_children, organization_xref_menu_root, and organization_xref_service_location form the hierarchy used by both menu publishing and master-menu workflows. These objects explain how corporate/default data is resolved to an organization, store, menu root, and service location.

### Menu publishing domain

menu-publisher exposes read APIs around menu items, toppings, crusts, specials, command screens, warnings/calories, language data, and availability. Functions named api_get_* usually compose corporate menu tables, organization override tables, language tables, service-location applicability, and availability/schedule tables. Procedures named api_update_org_* usually write org-level overrides or availability/status data.

### Menu creation and amendment domain

master-menu uses menu_create_data_type, menu_create_data_type_view, menu_create_data_type_column, menu_create_data_id, menu_create_list_view, and menu_create_list_view_amendment to drive the generic menu-create UI. data_type_id identifies the entity family and data_id identifies the concrete entity. The amendment-specific routine routing is documented in [amendment-feature-map.md](amendment-feature-map.md).

### Permission and validation domain

app_permission_* tables and routines drive field/view/user permissions. api_menu_create_get_column_permission, api_menu_create_get_data_type_validate_update_allowed, and validation routines combine permissions, deployment state, and data-type metadata to decide whether a menu-create row or column can be edited.

### Deployment/status domain

menu_create_deployment_*, ETL action routines, validation routines, and status functions determine list-view states and deployment progression. Prefer deterministic evidence queries and the menu-status skills before calling broad routines that create temp tables or mutate state.

### Data types and sequences

The public schema defines no enum, domain, or range types. The only standalone user-defined types are seven composites: api_json_data_type, deliverect_pricing_data_type, image_file_metadata_data_type, org_svl_data_type, and tablefunc_crosstab_2/3/4 from the tablefunc extension. Every other composite type is a table or view row type and is excluded from [database-schema.md](database-schema.md). Sequences are cataloged with ownership in the schema doc; master-menu has 1 sequence and menu-publisher has 15, and the local sequences are not catalog-owned by a column.

### Trigger behavior

Both databases use BEFORE INSERT/UPDATE triggers to normalize data; none are deferred or statement-level in the local schema.

- tax_category_to_upper_trigger runs tax_category_to_upper_trigger_function to uppercase tax category codes. It is attached to common_tax_category, corp_menu_addon, and org_menu_addon in both databases.
- Calorie normalization differs by database. master-menu uses calories_to_upper_function via corp_calories_to_uppercase_trigger and org_calories_to_uppercase_trigger. menu-publisher uses calories_trigger_function via corp_menu_calories_trigger and org_menu_calories_trigger, both on the calories tables.
- menu-publisher additionally runs store_process_info_feed_trigger_function via store_process_info_feed_trigger on organization_as_store, organization_as_store_attribute, and organization_xref_service_location to maintain store process feed data.
- menu-publisher defines 5 trigger functions but only 3 are currently attached to triggers; master-menu defines and attaches 2.

### Menu Admin audit boundary

Menu Admin uses PostgreSQL trigger auditing for its own database. Every new Menu Admin table migration must create or verify `audit.if_modified` trigger coverage in that migration, and the migration remains the authority for its own trigger attachment state.

Keep that separate from Menu Manager application-driven audit logging, where repositories and services use `IAuditLogger` and `AuditEventBuilder` against the Menu Manager application-audit flow instead of PostgreSQL triggers.

## Safe usage rules

- Read these docs first before querying localhost for object discovery.
- Do not copy connection strings, secrets, row dumps, or customer/user data into docs.
- Do not call write-capable routines for diagnostics.
- Treat functions that create temp objects as session-sensitive.
- For status reason or status repair, use menu-status-reason or menu-status-transition.
