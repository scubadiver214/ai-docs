# menu-publisher - corporate-menu 7

```mermaid
erDiagram
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
    common_menu_service_location {
        int service_location_id PK
        int service_location_id_default_image
        int service_location_id_default_language
        string internal_name
        string internal_description
        int order_origin_id
        string order_origin_code
        string vertex_customer_class_code
        boolean market_place_location
        int deliverect_language_id
        int deliverect_channel_id
        string deliverect_menu_code
        string deliverect_menu_name
        boolean deliverect_menu_flatten_required
        string deliverect_delivery_pricing_code
        string deliverect_pickup_pricing_code
        string additional_columns_omitted
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
    corp_menu_item_topping_size {
        int item_id PK
        int topping_id PK
        int topping_size_type_id PK
        int maximum_allowed_level
        decimal regular_level_count_as
        boolean included_in_item
        int included_in_item_at_level
        boolean active
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    corp_menu_special {
        int special_id PK
        int discount_type
        int upsell_category_id
        string internal_name
        string internal_description
        boolean allow_unbundling_for_promo
        boolean is_best_price_selection_allowed
        boolean is_lto
        boolean dmb_applicable
        boolean dmb_restricted
        int dmb_product_type_id
        boolean online_show_banner_on_mobile
        boolean online_show_banner_on_web
        string online_deep_link_url_mobile
        string online_deep_link_url_web
        boolean archived
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
        boolean is_hidden
        boolean active
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    corp_menu_special_availability_image {
        int special_id PK
        int service_location_id PK
        int image_id PK
        string image_type
        string image_file_name
        array image_file_metadata
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    corp_menu_special_availability_restore {
        int special_id
        int service_location_id
        decimal price
        decimal price_minimum_allowed
        decimal price_maximum_allowed
        int display_group_id
        int display_order
        boolean orderable
        boolean is_hidden
        boolean active
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    corp_menu_special_availability_schedule {
        int id PK
        int special_id
        int service_location_id
        string description
        date start_date
        date end_date
        datetime start_time
        datetime end_time
        boolean all_day_schedule
        boolean override_active
        boolean override_active_with
        boolean apply_to_future_orders
        boolean apply_to_day_of_week1
        boolean apply_to_day_of_week2
        boolean apply_to_day_of_week3
        boolean apply_to_day_of_week4
        string additional_columns_omitted
    }
    store_menu_special_availability_schedule {
        int id PK
        int organization_id PK
        int special_id
        int service_location_id
        string description
        date start_date
        date end_date
        datetime start_time
        datetime end_time
        boolean all_day_schedule
        boolean override_active
        boolean override_active_with
        boolean apply_to_future_orders
        boolean apply_to_day_of_week1
        boolean apply_to_day_of_week2
        boolean apply_to_day_of_week3
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
    work_org_menu_special_availability_snapshot_20260515 {
        int organization_id
        int special_id
        int service_location_id
        decimal price
        decimal price_minimum_allowed
        decimal price_maximum_allowed
        boolean active
    }
    common_menu_display_group ||--o{ corp_menu_special_availability : display_group_id
    common_menu_display_group ||--o{ corp_menu_special_availability_restore : display_group_id
    common_menu_service_location ||--o{ corp_menu_special_availability_restore : service_location_id
    common_menu_service_location ||--o{ corp_menu_special_availability_schedule : service_location_id
    common_menu_upsell_category ||--o{ corp_menu_special : upsell_category_id
    corp_menu_special ||--o{ corp_menu_special_availability_restore : special_id
    corp_menu_special ||--o{ corp_menu_special_availability_schedule : special_id
    corp_menu_special ||--o{ store_menu_special_availability_schedule : special_id
    corp_menu_special ||--o{ work_menu_redesign_image_source : special_id
    corp_menu_special ||--o{ work_menu_redesign_image_source_qa : special_id
    corp_menu_special ||--o{ work_online_product_image_import_step1 : special_id
    corp_menu_special ||--o{ work_online_product_image_import_step2 : special_id
    corp_menu_special ||--o{ work_org_menu_special_availability_snapshot_20260515 : special_id
```
