# Database schema diagrams

Generated from local PostgreSQL profile localhost on 2026-06-26 using pgsql MCP metadata queries against master-menu and menu-publisher.

These diagrams show table columns, Mermaid-safe PostgreSQL type categories, primary-key columns from the live catalog, and documented table relationships. The local databases do not enforce physical foreign keys, so relationship lines are documentation links rather than database constraints.

This file is an index. Each linked diagram file is a small Markdown document containing one Mermaid ER diagram. Some very wide tables cap the displayed column list and include `additional_columns_omitted`; use [database-schema.md](database-schema.md) for the exhaustive column catalog.

## master-menu

Tables: 176. Relationship candidates: 62. Diagram files:

- [common-reference 1](database-schema-mermaid-assets/master-menu-common-reference-1.md) - Shared menu reference data such as categories, crusts, display groups, service locations, tax categories, languages, and supporting lookup tables. Tables shown: 13; relationships shown: 12.

- [common-reference 2](database-schema-mermaid-assets/master-menu-common-reference-2.md) - Shared menu reference data such as categories, crusts, display groups, service locations, tax categories, languages, and supporting lookup tables. Tables shown: 17; relationships shown: 11.

- [common-reference 3](database-schema-mermaid-assets/master-menu-common-reference-3.md) - Shared menu reference data such as categories, crusts, display groups, service locations, tax categories, languages, and supporting lookup tables. Tables shown: 12; relationships shown: 6.

- [common-reference 4](database-schema-mermaid-assets/master-menu-common-reference-4.md) - Shared menu reference data such as categories, crusts, display groups, service locations, tax categories, languages, and supporting lookup tables. Tables shown: 11; relationships shown: 5.

- [common-reference 5](database-schema-mermaid-assets/master-menu-common-reference-5.md) - Shared menu reference data such as categories, crusts, display groups, service locations, tax categories, languages, and supporting lookup tables. Tables shown: 15; relationships shown: 12.

- [common-reference 6](database-schema-mermaid-assets/master-menu-common-reference-6.md) - Shared menu reference data such as categories, crusts, display groups, service locations, tax categories, languages, and supporting lookup tables. Tables shown: 6; relationships shown: 0.

- [corporate-menu 1](database-schema-mermaid-assets/master-menu-corporate-menu-1.md) - Corporate menu item, topping, crust, addon, special, availability, language, image, and schedule tables. Tables shown: 8; relationships shown: 2.

- [corporate-menu 2](database-schema-mermaid-assets/master-menu-corporate-menu-2.md) - Corporate menu item, topping, crust, addon, special, availability, language, image, and schedule tables. Tables shown: 14; relationships shown: 8.

- [corporate-menu 3](database-schema-mermaid-assets/master-menu-corporate-menu-3.md) - Corporate menu item, topping, crust, addon, special, availability, language, image, and schedule tables. Tables shown: 10; relationships shown: 5.

- [corporate-menu 4](database-schema-mermaid-assets/master-menu-corporate-menu-4.md) - Corporate menu item, topping, crust, addon, special, availability, language, image, and schedule tables. Tables shown: 6; relationships shown: 0.

- [corporate-menu 5](database-schema-mermaid-assets/master-menu-corporate-menu-5.md) - Corporate menu item, topping, crust, addon, special, availability, language, image, and schedule tables. Tables shown: 9; relationships shown: 4.

- [corporate-menu 6](database-schema-mermaid-assets/master-menu-corporate-menu-6.md) - Corporate menu item, topping, crust, addon, special, availability, language, image, and schedule tables. Tables shown: 6; relationships shown: 0.

- [corporate-menu 7](database-schema-mermaid-assets/master-menu-corporate-menu-7.md) - Corporate menu item, topping, crust, addon, special, availability, language, image, and schedule tables. Tables shown: 8; relationships shown: 2.

- [corporate-menu 8](database-schema-mermaid-assets/master-menu-corporate-menu-8.md) - Corporate menu item, topping, crust, addon, special, availability, language, image, and schedule tables. Tables shown: 3; relationships shown: 0.

- [menu-creation 1](database-schema-mermaid-assets/master-menu-menu-creation-1.md) - Menu creation metadata, list views, deployment, validation, and status tables. Tables shown: 6; relationships shown: 0.

- [menu-creation 2](database-schema-mermaid-assets/master-menu-menu-creation-2.md) - Menu creation metadata, list views, deployment, validation, and status tables. Tables shown: 6; relationships shown: 0.

- [miscellaneous](database-schema-mermaid-assets/master-menu-miscellaneous.md) - Miscellaneous tables and views that do not fit the primary database domains. Tables shown: 8; relationships shown: 2.

- [operations-audit-process](database-schema-mermaid-assets/master-menu-operations-audit-process.md) - Audit, file/image, JSON menu, maintenance, store process, and operational support tables. Tables shown: 4; relationships shown: 1.

- [organization 1](database-schema-mermaid-assets/master-menu-organization-1.md) - Organization hierarchy, store, menu-root, service-location, and ownership tables. Tables shown: 8; relationships shown: 2.

- [organization 2](database-schema-mermaid-assets/master-menu-organization-2.md) - Organization hierarchy, store, menu-root, service-location, and ownership tables. Tables shown: 7; relationships shown: 2.

- [organization 3](database-schema-mermaid-assets/master-menu-organization-3.md) - Organization hierarchy, store, menu-root, service-location, and ownership tables. Tables shown: 7; relationships shown: 1.

- [organization 4](database-schema-mermaid-assets/master-menu-organization-4.md) - Organization hierarchy, store, menu-root, service-location, and ownership tables. Tables shown: 2; relationships shown: 0.

- [organization-menu-overrides 1](database-schema-mermaid-assets/master-menu-organization-menu-overrides-1.md) - Organization-level menu override, availability, pricing, status, and schedule tables. Tables shown: 8; relationships shown: 2.

- [organization-menu-overrides 2](database-schema-mermaid-assets/master-menu-organization-menu-overrides-2.md) - Organization-level menu override, availability, pricing, status, and schedule tables. Tables shown: 13; relationships shown: 7.

- [organization-menu-overrides 3](database-schema-mermaid-assets/master-menu-organization-menu-overrides-3.md) - Organization-level menu override, availability, pricing, status, and schedule tables. Tables shown: 12; relationships shown: 9.

- [organization-menu-overrides 4](database-schema-mermaid-assets/master-menu-organization-menu-overrides-4.md) - Organization-level menu override, availability, pricing, status, and schedule tables. Tables shown: 11; relationships shown: 5.

- [organization-menu-overrides 5](database-schema-mermaid-assets/master-menu-organization-menu-overrides-5.md) - Organization-level menu override, availability, pricing, status, and schedule tables. Tables shown: 9; relationships shown: 4.

- [permissions 1](database-schema-mermaid-assets/master-menu-permissions-1.md) - Application permission, user, group, field, view, and operation tables. Tables shown: 8; relationships shown: 2.

- [permissions 2](database-schema-mermaid-assets/master-menu-permissions-2.md) - Application permission, user, group, field, view, and operation tables. Tables shown: 7; relationships shown: 1.

- [permissions 3](database-schema-mermaid-assets/master-menu-permissions-3.md) - Application permission, user, group, field, view, and operation tables. Tables shown: 7; relationships shown: 1.

- [work-temp 1](database-schema-mermaid-assets/master-menu-work-temp-1.md) - Work, temporary, import, archive, load-test, migration, and one-off operational tables. Tables shown: 12; relationships shown: 12.

- [work-temp 2](database-schema-mermaid-assets/master-menu-work-temp-2.md) - Work, temporary, import, archive, load-test, migration, and one-off operational tables. Tables shown: 2; relationships shown: 1.

Relationship lines emitted across sections: 119.

## menu-publisher

Tables: 265. Relationship candidates: 180. Diagram files:

- [common-reference 1](database-schema-mermaid-assets/menu-publisher-common-reference-1.md) - Shared menu reference data such as categories, crusts, display groups, service locations, tax categories, languages, and supporting lookup tables. Tables shown: 18; relationships shown: 18.

- [common-reference 2](database-schema-mermaid-assets/menu-publisher-common-reference-2.md) - Shared menu reference data such as categories, crusts, display groups, service locations, tax categories, languages, and supporting lookup tables. Tables shown: 25; relationships shown: 19.

- [common-reference 3](database-schema-mermaid-assets/menu-publisher-common-reference-3.md) - Shared menu reference data such as categories, crusts, display groups, service locations, tax categories, languages, and supporting lookup tables. Tables shown: 15; relationships shown: 9.

- [common-reference 4](database-schema-mermaid-assets/menu-publisher-common-reference-4.md) - Shared menu reference data such as categories, crusts, display groups, service locations, tax categories, languages, and supporting lookup tables. Tables shown: 30; relationships shown: 24.

- [common-reference 5](database-schema-mermaid-assets/menu-publisher-common-reference-5.md) - Shared menu reference data such as categories, crusts, display groups, service locations, tax categories, languages, and supporting lookup tables. Tables shown: 19; relationships shown: 18.

- [common-reference 6](database-schema-mermaid-assets/menu-publisher-common-reference-6.md) - Shared menu reference data such as categories, crusts, display groups, service locations, tax categories, languages, and supporting lookup tables. Tables shown: 12; relationships shown: 6.

- [corporate-menu 1](database-schema-mermaid-assets/menu-publisher-corporate-menu-1.md) - Corporate menu item, topping, crust, addon, special, availability, language, image, and schedule tables. Tables shown: 9; relationships shown: 4.

- [corporate-menu 2](database-schema-mermaid-assets/menu-publisher-corporate-menu-2.md) - Corporate menu item, topping, crust, addon, special, availability, language, image, and schedule tables. Tables shown: 24; relationships shown: 18.

- [corporate-menu 3](database-schema-mermaid-assets/menu-publisher-corporate-menu-3.md) - Corporate menu item, topping, crust, addon, special, availability, language, image, and schedule tables. Tables shown: 9; relationships shown: 6.

- [corporate-menu 4](database-schema-mermaid-assets/menu-publisher-corporate-menu-4.md) - Corporate menu item, topping, crust, addon, special, availability, language, image, and schedule tables. Tables shown: 14; relationships shown: 10.

- [corporate-menu 5](database-schema-mermaid-assets/menu-publisher-corporate-menu-5.md) - Corporate menu item, topping, crust, addon, special, availability, language, image, and schedule tables. Tables shown: 6; relationships shown: 0.

- [corporate-menu 6](database-schema-mermaid-assets/menu-publisher-corporate-menu-6.md) - Corporate menu item, topping, crust, addon, special, availability, language, image, and schedule tables. Tables shown: 12; relationships shown: 6.

- [corporate-menu 7](database-schema-mermaid-assets/menu-publisher-corporate-menu-7.md) - Corporate menu item, topping, crust, addon, special, availability, language, image, and schedule tables. Tables shown: 15; relationships shown: 13.

- [corporate-menu 8](database-schema-mermaid-assets/menu-publisher-corporate-menu-8.md) - Corporate menu item, topping, crust, addon, special, availability, language, image, and schedule tables. Tables shown: 6; relationships shown: 0.

- [corporate-menu 9](database-schema-mermaid-assets/menu-publisher-corporate-menu-9.md) - Corporate menu item, topping, crust, addon, special, availability, language, image, and schedule tables. Tables shown: 9; relationships shown: 4.

- [corporate-menu 10](database-schema-mermaid-assets/menu-publisher-corporate-menu-10.md) - Corporate menu item, topping, crust, addon, special, availability, language, image, and schedule tables. Tables shown: 4; relationships shown: 0.

- [miscellaneous 1](database-schema-mermaid-assets/menu-publisher-miscellaneous-1.md) - Miscellaneous tables and views that do not fit the primary database domains. Tables shown: 10; relationships shown: 4.

- [miscellaneous 2](database-schema-mermaid-assets/menu-publisher-miscellaneous-2.md) - Miscellaneous tables and views that do not fit the primary database domains. Tables shown: 12; relationships shown: 8.

- [miscellaneous 3](database-schema-mermaid-assets/menu-publisher-miscellaneous-3.md) - Miscellaneous tables and views that do not fit the primary database domains. Tables shown: 12; relationships shown: 8.

- [miscellaneous 4](database-schema-mermaid-assets/menu-publisher-miscellaneous-4.md) - Miscellaneous tables and views that do not fit the primary database domains. Tables shown: 7; relationships shown: 2.

- [operations-audit-process 1](database-schema-mermaid-assets/menu-publisher-operations-audit-process-1.md) - Audit, file/image, JSON menu, maintenance, store process, and operational support tables. Tables shown: 11; relationships shown: 6.

- [operations-audit-process 2](database-schema-mermaid-assets/menu-publisher-operations-audit-process-2.md) - Audit, file/image, JSON menu, maintenance, store process, and operational support tables. Tables shown: 9; relationships shown: 4.

- [operations-audit-process 3](database-schema-mermaid-assets/menu-publisher-operations-audit-process-3.md) - Audit, file/image, JSON menu, maintenance, store process, and operational support tables. Tables shown: 8; relationships shown: 3.

- [operations-audit-process 4](database-schema-mermaid-assets/menu-publisher-operations-audit-process-4.md) - Audit, file/image, JSON menu, maintenance, store process, and operational support tables. Tables shown: 7; relationships shown: 2.

- [operations-audit-process 5](database-schema-mermaid-assets/menu-publisher-operations-audit-process-5.md) - Audit, file/image, JSON menu, maintenance, store process, and operational support tables. Tables shown: 9; relationships shown: 3.

- [operations-audit-process 6](database-schema-mermaid-assets/menu-publisher-operations-audit-process-6.md) - Audit, file/image, JSON menu, maintenance, store process, and operational support tables. Tables shown: 6; relationships shown: 0.

- [operations-audit-process 7](database-schema-mermaid-assets/menu-publisher-operations-audit-process-7.md) - Audit, file/image, JSON menu, maintenance, store process, and operational support tables. Tables shown: 6; relationships shown: 0.

- [operations-audit-process 8](database-schema-mermaid-assets/menu-publisher-operations-audit-process-8.md) - Audit, file/image, JSON menu, maintenance, store process, and operational support tables. Tables shown: 3; relationships shown: 0.

- [organization 1](database-schema-mermaid-assets/menu-publisher-organization-1.md) - Organization hierarchy, store, menu-root, service-location, and ownership tables. Tables shown: 28; relationships shown: 22.

- [organization 2](database-schema-mermaid-assets/menu-publisher-organization-2.md) - Organization hierarchy, store, menu-root, service-location, and ownership tables. Tables shown: 8; relationships shown: 2.

- [organization 3](database-schema-mermaid-assets/menu-publisher-organization-3.md) - Organization hierarchy, store, menu-root, service-location, and ownership tables. Tables shown: 8; relationships shown: 3.

- [organization 4](database-schema-mermaid-assets/menu-publisher-organization-4.md) - Organization hierarchy, store, menu-root, service-location, and ownership tables. Tables shown: 6; relationships shown: 1.

- [organization-menu-overrides 1](database-schema-mermaid-assets/menu-publisher-organization-menu-overrides-1.md) - Organization-level menu override, availability, pricing, status, and schedule tables. Tables shown: 11; relationships shown: 6.

- [organization-menu-overrides 2](database-schema-mermaid-assets/menu-publisher-organization-menu-overrides-2.md) - Organization-level menu override, availability, pricing, status, and schedule tables. Tables shown: 6; relationships shown: 0.

- [organization-menu-overrides 3](database-schema-mermaid-assets/menu-publisher-organization-menu-overrides-3.md) - Organization-level menu override, availability, pricing, status, and schedule tables. Tables shown: 16; relationships shown: 10.

- [organization-menu-overrides 4](database-schema-mermaid-assets/menu-publisher-organization-menu-overrides-4.md) - Organization-level menu override, availability, pricing, status, and schedule tables. Tables shown: 12; relationships shown: 15.

- [organization-menu-overrides 5](database-schema-mermaid-assets/menu-publisher-organization-menu-overrides-5.md) - Organization-level menu override, availability, pricing, status, and schedule tables. Tables shown: 11; relationships shown: 5.

- [organization-menu-overrides 6](database-schema-mermaid-assets/menu-publisher-organization-menu-overrides-6.md) - Organization-level menu override, availability, pricing, status, and schedule tables. Tables shown: 10; relationships shown: 9.

- [organization-menu-overrides 7](database-schema-mermaid-assets/menu-publisher-organization-menu-overrides-7.md) - Organization-level menu override, availability, pricing, status, and schedule tables. Tables shown: 8; relationships shown: 5.

- [permissions 1](database-schema-mermaid-assets/menu-publisher-permissions-1.md) - Application permission, user, group, field, view, and operation tables. Tables shown: 8; relationships shown: 2.

- [permissions 2](database-schema-mermaid-assets/menu-publisher-permissions-2.md) - Application permission, user, group, field, view, and operation tables. Tables shown: 6; relationships shown: 0.

- [permissions 3](database-schema-mermaid-assets/menu-publisher-permissions-3.md) - Application permission, user, group, field, view, and operation tables. Tables shown: 3; relationships shown: 0.

- [work-temp 1](database-schema-mermaid-assets/menu-publisher-work-temp-1.md) - Work, temporary, import, archive, load-test, migration, and one-off operational tables. Tables shown: 16; relationships shown: 21.

- [work-temp 2](database-schema-mermaid-assets/menu-publisher-work-temp-2.md) - Work, temporary, import, archive, load-test, migration, and one-off operational tables. Tables shown: 21; relationships shown: 22.

- [work-temp 3](database-schema-mermaid-assets/menu-publisher-work-temp-3.md) - Work, temporary, import, archive, load-test, migration, and one-off operational tables. Tables shown: 17; relationships shown: 18.

- [work-temp 4](database-schema-mermaid-assets/menu-publisher-work-temp-4.md) - Work, temporary, import, archive, load-test, migration, and one-off operational tables. Tables shown: 9; relationships shown: 3.

- [work-temp 5](database-schema-mermaid-assets/menu-publisher-work-temp-5.md) - Work, temporary, import, archive, load-test, migration, and one-off operational tables. Tables shown: 4; relationships shown: 3.

Relationship lines emitted across sections: 342.


