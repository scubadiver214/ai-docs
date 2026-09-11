# menu-publisher - organization 4

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
    organization_xref_menu_root_action {
        int organization_id PK
        int xref_type PK
        int xref_id PK
        int action_id PK
        int action_sequence_id PK
        string action_processing_group_id
        string action_name
        date action_date
        boolean action_processed
        boolean action_forced
        string action_message
        int menu_creation_source_environment_id
        boolean menu_creation_update_required
        boolean menu_creation_update_completed
        boolean approved
        string approved_by_login_id
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
    organization_xref_service_location {
        int organization_id PK
        int service_location_id PK
        boolean deliverect_integrated
        boolean deliverect_ready
        string deliverect_channel_link_id
        boolean publish_menu
        boolean publish_menu_auto_set
        boolean active
        datetime activate_on
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    organization_xref_service_location_markup_rate {
        int organization_id PK
        int service_location_id PK
        decimal price_markup_rate
        datetime updated_on
    }
    app_permission_group ||--o{ organization_xref_menu_root_action_adjust_display_order : group_id
```
