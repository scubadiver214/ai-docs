# master-menu - organization 3

```mermaid
erDiagram
    app_permission_group {
        string group_id PK
        string group_description
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    organization_xref_children {
        int organization_id PK
        int child_id PK
        int child_level
    }
    organization_xref_language {
        int organization_id PK
        int language_id PK
        boolean is_default
        boolean is_store_supported
        boolean is_online_supported
    }
    organization_xref_menu_root {
        int organization_id PK
        int xref_type PK
        int xref_id PK
        int restrict_to_organizations
        boolean in_draft_mode
        boolean validated
        boolean activated
        datetime activated_date
        boolean disabled
        datetime disabled_date
        boolean archived
        datetime archived_date
        datetime updated_on
    }
    organization_xref_menu_root_action {
        int organization_id PK
        int xref_type PK
        int xref_id PK
        int action_id PK
        int action_sequence_id PK
        string action_name
        string action_processing_group_id
        date action_date
        boolean action_processed
        boolean action_forced
        string action_message
        boolean approved
        string approved_by_login_id
        string approved_by_first_name
        string approved_by_last_name
        datetime approved_on
        string additional_columns_omitted
    }
    organization_xref_menu_root_action_adjust_display_order {
        string group_id
        int organization_id PK
        int xref_type PK
        int xref_id PK
        int service_location_id PK
        int display_order
        boolean make_first
        boolean processed
        datetime processed_on
        datetime updated_on
    }
    organization_xref_menu_tax_included {
        int organization_id PK
        int item_id PK
        boolean tax_included
        datetime updated_on
    }
    app_permission_group ||--o{ organization_xref_menu_root_action_adjust_display_order : group_id
```
