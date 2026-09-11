# Database diagrams and flow charts

Generated from local PostgreSQL profile localhost on 2026-06-26 using pgsql MCP tools against databases master-menu and menu-publisher.

These Mermaid diagrams are model-readable navigation aids. Relationships are inferred because the local databases do not rely on foreign keys. Verify exact joins in [database-routines.md](database-routines.md) and live routine definitions before changing behavior.

## Two-database overview

~~~mermaid
flowchart LR
    API[ASP.NET API]
    Dapper[DapperWrapper]
    DefaultSpec[Default/shared DB route]
    MenuCreateSpec[MenuCreationDatabaseSpecification]
    Publisher[(menu-publisher)]
    Master[(master-menu)]

    API --> Dapper
    Dapper --> DefaultSpec --> Publisher
    Dapper --> MenuCreateSpec --> Master

    Publisher --> PubReads[api_get_* read functions]
    Publisher --> PubWrites[api_update_org_* procedures]
    Master --> CreateReads[api_menu_create_get_* functions]
    Master --> CreateWrites[api_menu_create_update_* procedures]
    Master --> Deploy[validation and deployment routines]
~~~

## Organization hierarchy and menu-root flow

~~~mermaid
flowchart TD
    Organization[organization]
    Ancestor[organization_xref_ancestor]
    Children[organization_xref_children]
    Store[organization_as_store]
    Root[organization_as_menu_root]
    RootXref[organization_xref_menu_root]
    ServiceXref[organization_xref_service_location]
    ServiceLocation[common_menu_service_location]
    OrgOverrides[org_menu_* overrides]
    CorpMenu[corp_menu_* base data]

    Organization --> Ancestor
    Organization --> Children
    Organization --> Store
    Organization --> Root
    Root --> RootXref
    Root --> ServiceXref
    ServiceXref --> ServiceLocation
    CorpMenu --> RootXref
    RootXref --> OrgOverrides
~~~

## Menu publisher read/update flow

~~~mermaid
flowchart TD
    Repo[Repository method]
    GetRoutine[api_get_* function]
    UpdateRoutine[api_update_org_* procedure]
    Corp[corp_menu_*]
    Org[org_menu_*]
    Common[common_menu_*]
    Language[language and *_language]
    OrgHierarchy[organization_xref_*]
    Response[API response model]

    Repo --> GetRoutine
    Repo --> UpdateRoutine
    GetRoutine --> Corp
    GetRoutine --> Org
    GetRoutine --> Common
    GetRoutine --> Language
    GetRoutine --> OrgHierarchy
    GetRoutine --> Response
    UpdateRoutine --> Org
    UpdateRoutine --> OrgHierarchy
~~~

## Menu creation list/status/update flow

~~~mermaid
flowchart TD
    UIRequest[Menu creation API request]
    Service[MenuCreationDataService]
    Spec[MenuCreationDatabaseSpecification]
    DataType[menu_create_data_type and view metadata]
    DataId[menu_create_data_id]
    ListView[menu_create_list_view / amendment view]
    Permission[app_permission_*]
    Status[api_menu_create_get_data_type_view_status]
    Validation[validation routines]
    UpdateWrapper[api_menu_create_update_view_*]
    Amendment[optional *_amendment wrapper]
    BaseUpdate[base update procedure]

    UIRequest --> Service --> Spec
    Spec --> DataType
    Spec --> DataId
    Spec --> ListView
    Spec --> Permission
    ListView --> Status
    Status --> Validation
    Service --> UpdateWrapper
    UpdateWrapper --> Amendment
    Amendment --> BaseUpdate
    UpdateWrapper --> BaseUpdate
~~~

## Permission and column update flow

~~~mermaid
flowchart TD
    User[user id]
    DataTypeColumn[menu_create_data_type_column]
    PermissionTables[app_permission_*]
    DeploymentState[deployment/status state]
    ColumnPermission[api_menu_create_get_column_permission]
    Enforce[api_menu_create_update_view_enforce_write_allowed]
    Update[api_menu_create_update_view_*]

    User --> PermissionTables
    DataTypeColumn --> ColumnPermission
    PermissionTables --> ColumnPermission
    DeploymentState --> ColumnPermission
    ColumnPermission --> Enforce
    Enforce --> Update
~~~

## Trigger event flow

Local triggers are BEFORE INSERT/UPDATE row triggers that normalize data. The diagram shows the actual trigger functions in both databases.

~~~mermaid
flowchart TD
    TaxTables[common_tax_category, corp_menu_addon, org_menu_addon]
    TaxFn[tax_category_to_upper_trigger_function]
    CalTablesM[corp_menu_calories, org_menu_calories]
    CalFnMaster[calories_to_upper_function - master-menu]
    CalFnPublisher[calories_trigger_function - menu-publisher]
    StoreTables[organization_as_store, organization_as_store_attribute, organization_xref_service_location]
    StoreFn[store_process_info_feed_trigger_function - menu-publisher]

    TaxTables -- BEFORE INSERT/UPDATE --> TaxFn --> UppercaseTax[uppercased tax category code]
    CalTablesM -- BEFORE INSERT/UPDATE --> CalFnMaster --> NormalizedCalories[normalized calories]
    CalTablesM -- BEFORE INSERT/UPDATE --> CalFnPublisher --> NormalizedCalories
    StoreTables -- BEFORE INSERT/UPDATE --> StoreFn --> StoreFeed[store process feed data]
~~~

## Sequence/key-generation flow

~~~mermaid
flowchart LR
    Sequence[sequence]
    DefaultExpr[column default / nextval]
    TableColumn[table primary or identifier column]
    InsertRoutine[insert/update routine]

    Sequence --> DefaultExpr --> TableColumn
    InsertRoutine --> TableColumn
~~~

## Routine dependency flow

~~~mermaid
flowchart TD
    CSharp[C# repository/service call site]
    Routine[function or procedure]
    TriggerFunction[trigger function]
    Reads[referenced read objects]
    Writes[referenced write objects]
    Calls[called routines]
    Trigger[trigger]

    CSharp --> Routine
    Routine --> Reads
    Routine --> Writes
    Routine --> Calls
    Trigger --> TriggerFunction
    TriggerFunction --> Reads
    TriggerFunction --> Writes
    TriggerFunction --> Calls
~~~
