# master-menu - organization 4

```mermaid
erDiagram
    organization_xref_service_location {
        int organization_id PK
        int service_location_id PK
        boolean deliverect_integrated
        boolean deliverect_ready
        string deliverect_channel_link_id
        boolean publish_menu
        boolean publish_menu_auto_set
        boolean active
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
```
