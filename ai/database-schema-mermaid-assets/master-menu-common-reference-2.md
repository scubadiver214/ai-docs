# master-menu - common-reference 2

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
    common_menu_custom_option_group {
        int group_id PK
        string internal_name
        string internal_description
        int display_order
        int grouped_types
        boolean active
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    common_menu_custom_option_group_language {
        int group_id PK
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
    common_menu_custom_option_language {
        int type_id PK
        int option_id PK
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
    common_menu_custom_option_type {
        int type_id PK
        string internal_name
        string internal_description
        int apply_to_categories
        decimal charge
        decimal charge_minimum_allowed
        decimal charge_maximum_allowed
        int default_option_id
        int display_order
        boolean apply_on_make_station
        boolean apply_on_landing_station
        boolean is_always_visible
        boolean allow_additional_charge
        boolean auto_add_on_item_creation
        boolean required
        boolean active
        string additional_columns_omitted
    }
    common_menu_custom_option_type_language {
        int type_id PK
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
    common_menu_display_group {
        int display_group_id PK
        int display_group_parent_id
        int display_group_level
        boolean display_group_level_is_final
        boolean display_group_is_hnr
        boolean display_group_is_create_your_own
        int display_order
        string internal_name
        string internal_description
        boolean apply_to_topping
        boolean apply_to_category
        boolean apply_to_item
        boolean apply_to_special
        boolean apply_to_in_store
        boolean apply_to_online
        boolean apply_to_deliverect_only
        string additional_columns_omitted
    }
    corp_menu_item_availability {
        int item_id PK
        int service_location_id PK
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
    corp_menu_special_availability {
        int special_id PK
        int service_location_id PK
        decimal price
        decimal price_minimum_allowed
        decimal price_maximum_allowed
        int display_group_id
        int display_order
        boolean orderable
        boolean active
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    corp_menu_topping {
        int topping_id PK
        int category_id
        string internal_name
        string internal_description
        int maximum_allowed_level
        int display_group_id
        int display_order
        int make_display_order
        boolean post_bake_eligible
        boolean allow_sale_on_the_side
        boolean archived
        datetime archived_on
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    org_menu_item_availability {
        int organization_id PK
        int item_id PK
        int service_location_id PK
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
        string additional_columns_omitted
    }
    org_menu_special_availability {
        int organization_id PK
        int special_id PK
        int service_location_id PK
        decimal price
        decimal price_minimum_allowed
        decimal price_maximum_allowed
        int display_group_id
        int display_order
        boolean orderable
        boolean active
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    org_menu_topping {
        int organization_id PK
        int topping_id PK
        int category_id
        string internal_name
        string internal_description
        int maximum_allowed_level
        boolean post_bake_eligible
        int display_group_id
        int display_order
        int make_display_order
        boolean allow_sale_on_the_side
        boolean archived
        datetime archived_on
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        string additional_columns_omitted
    }
    store_process_menu_deliverect_product {
        int root_organization_id PK
        int service_location_id PK
        boolean service_location_flatten_required
        int xref_type PK
        int xref_id PK
        string product_plu
        boolean product_flatten_required
        int display_group_id
        int display_order
        boolean hidden
        string row_hash
        int product_status
        string product_status_description
        boolean archived
        datetime updated_on
    }
    work_new_menu_request_389478_existing_item_information {
        int item_id PK
        string internal_name
        string english_name
        decimal oven_time
        decimal expiration_time
        boolean available_online
        boolean available_3pd
        decimal price_non_3pd
        string english_description
        string english_abbreviation
        string calories
        string display_group_non_instore
        int display_group_id
        int display_order_non_instore
        string hnr_days
        string hnr_hours
        string additional_columns_omitted
    }
    work_new_menu_request_389478_existing_special_information {
        int special_id PK
        string internal_name
        string english_name
        decimal price_non_3pd
        string english_description
        string english_abbreviation
        string calories
        string display_group_non_instore
        int display_group_id
        int display_order_non_instore
    }
    work_new_menu_request_389478_new_item_information {
        int category_id
        int item_id PK
        string internal_name
        boolean is_core
        string name
        boolean is_lto
        boolean is_base_crust
        decimal oven_time
        decimal expiration_time
        string tax_category_code
        int crust_id
        int core_map_to_custom_id
        int package_id
        boolean available
        decimal price_non_3pd
        decimal price_3pd
        string additional_columns_omitted
    }
    common_menu_display_group ||--o{ common_menu_category : display_group_id
    common_menu_display_group ||--o{ corp_menu_item_availability : display_group_id
    common_menu_display_group ||--o{ corp_menu_special_availability : display_group_id
    common_menu_display_group ||--o{ corp_menu_topping : display_group_id
    common_menu_display_group ||--o{ org_menu_item_availability : display_group_id
    common_menu_display_group ||--o{ org_menu_special_availability : display_group_id
    common_menu_display_group ||--o{ org_menu_topping : display_group_id
    common_menu_display_group ||--o{ store_process_menu_deliverect_product : display_group_id
    common_menu_display_group ||--o{ work_new_menu_request_389478_existing_item_information : display_group_id
    common_menu_display_group ||--o{ work_new_menu_request_389478_existing_special_information : display_group_id
    common_menu_display_group ||--o{ work_new_menu_request_389478_new_item_information : display_group_id
```
