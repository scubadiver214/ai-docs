# Database discovery query guide

Use this guide to regenerate the local PostgreSQL documentation set from pgsql MCP outputs without rediscovering the catalog strategy.

## When to use this

- Local database documentation is stale or missing after schema/routine changes.
- A future agent needs to refresh [database-schema.md](database-schema.md), [database-schema-diagrams.md](database-schema-diagrams.md), [database-routines.md](database-routines.md), or [database-diagrams.md](database-diagrams.md) from localhost.
- You need the safe scope for catalog extraction before querying PostgreSQL.

## When not to use this

- You only need to answer a specific runtime data question; use focused read-only diagnostics instead.
- You need to change data or schema; this guide is documentation-only and read-only.
- You need production facts; these docs are generated from the local localhost profile.

## Inputs expected

- pgsql MCP connection profile: localhost.
- Database targets: master-menu and menu-publisher.
- Schema context fetched first with pgsql_db_context.
- Read-only pgsql query result JSON files for schema and routine catalogs.

## Output expected

- [database-schema.md](database-schema.md)
- [database-schema-diagrams.md](database-schema-diagrams.md)
- [database-routines.md](database-routines.md)
- [database-architecture.md](database-architecture.md)
- [database-diagrams.md](database-diagrams.md)
- [database-discovery-queries.md](database-discovery-queries.md)

## Generator script

After extracting the pgsql MCP result JSON files, run:

```powershell
.\scripts\generate-database-docs-from-pgsql-results.ps1 `
    -MasterCatalogJson .\tmp\master-catalog.json `
    -PublisherCatalogJson .\tmp\publisher-catalog.json `
    -MasterRoutinesJson .\tmp\master-routines.json `
    -PublisherRoutinesJson .\tmp\publisher-routines.json `
    -GeneratedDate 2026-06-26
```

After extracting table/column/key metadata JSON for schema diagrams, run:

```powershell
.\scripts\generate-basic-schema-diagrams.ps1 `
    -MasterSchemaJson .\tmp\master-schema.json `
    -PublisherSchemaJson .\tmp\publisher-schema.json `
    -GeneratedDate 2026-06-26
```

Use this guide when regenerating [database-schema.md](database-schema.md), [database-routines.md](database-routines.md), and [database-diagrams.md](database-diagrams.md). Prefer pgsql MCP tools connected to the localhost profile. Fetch schema context first with pgsql_db_context before running read-only catalog queries.

## Scope

- Databases: master-menu, menu-publisher.
- Schema: public unless a future migration adds application objects elsewhere.
- Allowed data: schema metadata, object definitions, dependency metadata, C# call-site references.
- Disallowed data: secrets, connection strings, row-level business/customer/user data dumps.

## Catalog categories to collect

- Tables, views, materialized views, columns, comments, constraints, indexes.
- Sequences and sequence ownership/dependencies.
- User-defined data types: enum, domain, composite, range, multirange. Exclude table and view row-type composites; document only standalone types.
- Functions, procedures, aggregate/window functions if present, trigger functions.
- Triggers and trigger target tables/events.
- Dependencies from pg_depend, plus text-based routine body references for dynamic SQL.
- C# Dapper call sites and database specification routing.

## Validation expectations

- Every documented object should exist in the corresponding local database at generation time.
- Every strong relationship should have evidence from a routine body, trigger, view, SQL file, or C# call site.
- Any relationship inferred only by name should be marked weak.
- Run git diff --check after documentation updates.
