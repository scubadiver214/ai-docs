# master-menu - corporate-menu 8

```mermaid
erDiagram
    corp_menu_warning {
        int warning_id PK
        int language_id PK
        int apply_to_service_locations
        string name
        string description
        int warning_type
        string image_file_name
        string image_alt_text
        boolean active
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    corp_menu_warning_xref_type {
        int xref_type PK
        int xref_id PK
        int warning_id PK
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    corp_menu_xref_online {
        int xref_type PK
        int xref_id PK
        boolean apply_to_legacy_only
        boolean apply_to_new_only
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
```
