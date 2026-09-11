# menu-publisher - common-reference 6

```mermaid
erDiagram
    dmb_product_hnr_exclude {
        int item_id PK
        int apply_to_root_organizations
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    dmb_product_image {
        int organization_id PK
        int language_id PK
        int xref_type PK
        int xref_id PK
        string image_type PK
        string image_file_name
        string image_file_name_original
        string image_alt_text
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    dmb_product_language {
        int organization_id PK
        int xref_type PK
        int xref_id PK
        int language_id PK
        string name
        string description
        string calories
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    dmb_product_topping_as_addon {
        int item_id PK
        int topping_id PK
        int apply_to_root_organizations
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    dmb_product_type {
        int product_type_id PK
        string product_type_code
        string product_type_name
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    language {
        int language_id PK
        string culture_code
        string culture_code_neutral
        string language_name_in_english
        string language_name_in_language
        boolean right_to_left
    }
    mm_maintenance_schedules {
        int id PK
        int language_id
        datetime start_time
        datetime end_time
        string title
        string description
        datetime updated_on
        string updated_by
    }
    work_menu_redesign_image_source {
        int row_id
        int root_organization_id
        string country
        int service_location_id
        int language_id
        string culture_code
        int xref_type_id
        string xref_type_name
        string item_code
        int id
        int item_id
        int special_id
        int special_source_row_id
        int addon_id_combined
        int addon_id_only
        boolean addon_id_is_default
        string additional_columns_omitted
    }
    work_menu_redesign_image_source_qa {
        int row_id
        int root_organization_id
        string country
        int service_location_id
        int language_id
        string culture_code
        int xref_type_id
        string xref_type_name
        string item_code
        int id
        int item_id
        int special_id
        int special_source_row_id
        int addon_id_combined
        int addon_id_only
        boolean addon_id_is_default
        string additional_columns_omitted
    }
    work_online_product_image_import {
        int row_id PK
        int xref_type
        int xref_id
        string xref_name
        int language_id
        string culture_code
        string image_type
        string image_type_code
        array image_types
        array image_types_code
        array image_file_urls
        string image_file_name
        string image_file_name_to_use
        string image_file_name_full
        boolean orderable
    }
    work_online_product_image_import_step1 {
        int row_id PK
        string menu_code_full
        int item_id
        int special_id
        string culture_code
        string image_type
        string image_file_name_full
        int xref_type
        int xref_id
        int language_id
        string image_file_name
    }
    work_online_product_image_import_step2 {
        int row_id PK
        string menu_code_full
        array menu_code_parts
        int menu_code_parts_count
        string menu_code
        int item_id
        int special_id
        string culture_code
        string image_type
        string image_file_name_full
        string image_file_folder
        array image_file_parts
        int image_file_parts_count
        string image_file_name
        int xref_type
        int xref_id
        string additional_columns_omitted
    }
    language ||--o{ mm_maintenance_schedules : language_id
    language ||--o{ work_menu_redesign_image_source : language_id
    language ||--o{ work_menu_redesign_image_source_qa : language_id
    language ||--o{ work_online_product_image_import : language_id
    language ||--o{ work_online_product_image_import_step1 : language_id
    language ||--o{ work_online_product_image_import_step2 : language_id
```
