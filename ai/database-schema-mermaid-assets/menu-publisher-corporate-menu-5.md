# menu-publisher - corporate-menu 5

```mermaid
erDiagram
    corp_menu_item_grouping_xref_item {
        int group_id PK
        int item_id PK
        int priority
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    corp_menu_item_grouping_xref_item_language {
        int group_id PK
        int item_id PK
        int service_location_id PK
        int language_id PK
        string short_name
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    corp_menu_item_language {
        int item_id PK
        int service_location_id PK
        int language_id PK
        string name
        string description
        string description_short
        string abbreviation
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    corp_menu_item_size_grouping {
        int group_id PK
        string internal_name
        string internal_description
        int default_item_id
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    corp_menu_item_size_grouping_image {
        int group_id PK
        int service_location_id PK
        int image_id PK
        string image_type
        string image_file_name
        array image_file_metadata_data
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    corp_menu_item_size_grouping_image_temp {
        int group_id PK
        int service_location_id PK
        int image_id PK
        string image_type
        string image_file_name
        array image_file_metadata_data
        string image_alt_text
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
```
