# menu-publisher - corporate-menu 2

```mermaid
erDiagram
    common_menu_category {
        int category_id PK
        string name
        string description
        string abbreviation
        int display_group_id
        int account_id
        boolean apply_to_item
        boolean apply_to_special
        boolean apply_to_topping
        boolean allow_topping
        boolean allow_addon
        boolean allow_customization
        boolean allow_oven_time
        boolean allow_expiration_time
        boolean active
        string updated_by_login_id
        string additional_columns_omitted
    }
    common_menu_crust {
        int crust_id PK
        string name
        string description
        string abbreviation
        int shape
        string shape_description
        decimal size
        int item_mapping_size
        boolean is_regular
        boolean is_deep_dish
        boolean is_thin
        boolean is_stuffed
        boolean is_pretzel
        boolean is_using_sheetout
        boolean is_parbaked
        boolean is_half
        string additional_columns_omitted
    }
    common_menu_package {
        int package_id PK
        string name
        int shape
        string shape_description
        decimal size
        string size_description
        boolean is_heatable
        boolean is_refrigerable
        boolean is_wastable
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    common_menu_upsell_category {
        int upsell_category_id PK
        string name
        string description
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    common_tax_category {
        string tax_category_code PK
        int tax_category_code_deliverect
        string category_description
        string secondary_tax_category_code
        string secondary_category_description
        int group_id
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    corp_menu_calories {
        int xref_type PK
        int xref_id PK
        int language_id PK
        string calories
        int apply_to_service_locations
        boolean active
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    corp_menu_custom_option_availability {
        int service_location_id PK
        int type_id PK
        int option_id PK
        decimal charge
        decimal charge_minimum_allowed
        decimal charge_maximum_allowed
        boolean orderable
        boolean active
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    corp_menu_deliverect_selection_language {
        int selection_id PK
        string selection_type
        int service_location_id PK
        int language_id PK
        string name
        string description
        string abbreviation
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    corp_menu_item {
        int item_id PK
        int legacy_item_id
        int category_id
        int upsell_category_id
        string internal_name
        string internal_description
        int hierarchy_level_id
        decimal oven_time
        decimal expiration_time
        string tax_category_code
        int core_map_to_custom_id
        boolean print_and_scan_label
        int crust_id
        int package_id
        boolean is_core
        boolean is_lto
        string additional_columns_omitted
    }
    corp_menu_item_addon_prompt_language {
        int category_id PK
        int service_location_id PK
        int language_id PK
        string prompt_description
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    corp_menu_item_allowed_addon {
        int item_id PK
        int addon_id PK
        boolean active
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    corp_menu_item_availability_restore {
        int item_id
        int service_location_id
        decimal price
        decimal price_minimum_allowed
        decimal price_maximum_allowed
        decimal topping_price
        decimal topping_price_minimum_allowed
        decimal topping_price_maximum_allowed
        int display_group_id
        int display_order
        boolean orderable
        boolean orderable_with_other
        boolean active
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        string additional_columns_omitted
    }
    corp_menu_item_availability_schedule {
        int id PK
        int item_id
        int service_location_id
        string description
        date start_date
        date end_date
        datetime start_time
        datetime end_time
        boolean all_day_schedule
        boolean override_active
        boolean override_active_with
        boolean override_price
        boolean override_price_apply_markup
        decimal override_price_with
        boolean apply_to_future_orders
        boolean apply_to_day_of_week1
        string additional_columns_omitted
    }
    corp_menu_item_availability_temp {
        int item_id
        int service_location_id
        decimal price
        decimal price_minimum_allowed
        decimal price_maximum_allowed
        decimal topping_price
        decimal topping_price_minimum_allowed
        decimal topping_price_maximum_allowed
        int display_group_id
        int display_order
        boolean orderable
        boolean orderable_with_other
        boolean active
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        string additional_columns_omitted
    }
    corp_menu_item_command_screen_schedule {
        int id PK
        int display_id
        int item_id
        string description
        date start_date
        date end_date
        datetime start_time
        datetime end_time
        boolean is_mandatory
        boolean all_day_schedule
        boolean apply_to_day_of_week1
        boolean apply_to_day_of_week2
        boolean apply_to_day_of_week3
        boolean apply_to_day_of_week4
        boolean apply_to_day_of_week5
        boolean apply_to_day_of_week6
        string additional_columns_omitted
    }
    corp_menu_item_copy {
        int item_id
        int legacy_item_id
        int category_id
        string internal_name
        string internal_description
        int hierarchy_level_id
        decimal oven_time
        decimal expiration_time
        string tax_category_code
        int core_map_to_custom_id
        boolean print_and_scan_label
        int crust_id
        int package_id
        boolean is_core
        boolean is_lto
        boolean is_test_product
        string additional_columns_omitted
    }
    corp_menu_item_snapshot_20260302 {
        int item_id
        int legacy_item_id
        int category_id
        int upsell_category_id
        string internal_name
        string internal_description
        int hierarchy_level_id
        decimal oven_time
        decimal expiration_time
        string tax_category_code
        int core_map_to_custom_id
        boolean print_and_scan_label
        int crust_id
        int package_id
        boolean is_core
        boolean is_lto
        string additional_columns_omitted
    }
    p_special_menu_item_ids {
        int item_id
    }
    store_menu_item_availability_schedule {
        int id PK
        int organization_id PK
        int item_id
        int service_location_id
        string description
        date start_date
        date end_date
        datetime start_time
        datetime end_time
        boolean all_day_schedule
        boolean override_active
        boolean override_active_with
        boolean override_price
        decimal override_price_with
        boolean override_price_apply_markup
        boolean apply_to_future_orders
        string additional_columns_omitted
    }
    store_menu_item_command_screen_schedule {
        int organization_id PK
        int id PK
        int display_id
        int item_id
        string description
        date start_date
        date end_date
        datetime start_time
        datetime end_time
        boolean is_mandatory
        boolean all_day_schedule
        boolean apply_to_day_of_week1
        boolean apply_to_day_of_week2
        boolean apply_to_day_of_week3
        boolean apply_to_day_of_week4
        boolean apply_to_day_of_week5
        string additional_columns_omitted
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
    common_menu_category ||--o{ corp_menu_item : category_id
    common_menu_crust ||--o{ corp_menu_item : crust_id
    common_menu_package ||--o{ corp_menu_item : package_id
    common_menu_upsell_category ||--o{ corp_menu_item : upsell_category_id
    common_tax_category ||--o{ corp_menu_item : tax_category_code
    corp_menu_item ||--o{ corp_menu_item_availability_restore : item_id
    corp_menu_item ||--o{ corp_menu_item_availability_schedule : item_id
    corp_menu_item ||--o{ corp_menu_item_availability_temp : item_id
    corp_menu_item ||--o{ corp_menu_item_command_screen_schedule : item_id
    corp_menu_item ||--o{ corp_menu_item_copy : item_id
    corp_menu_item ||--o{ corp_menu_item_snapshot_20260302 : item_id
    corp_menu_item ||--o{ p_special_menu_item_ids : item_id
    corp_menu_item ||--o{ store_menu_item_availability_schedule : item_id
    corp_menu_item ||--o{ store_menu_item_command_screen_schedule : item_id
    corp_menu_item ||--o{ work_menu_redesign_image_source : item_id
    corp_menu_item ||--o{ work_menu_redesign_image_source_qa : item_id
    corp_menu_item ||--o{ work_online_product_image_import_step1 : item_id
    corp_menu_item ||--o{ work_online_product_image_import_step2 : item_id
```
