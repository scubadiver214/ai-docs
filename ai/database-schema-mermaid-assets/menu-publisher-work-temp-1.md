# menu-publisher - work-temp 1

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
    language {
        int language_id PK
        string culture_code
        string culture_code_neutral
        string language_name_in_english
        string language_name_in_language
        boolean right_to_left
    }
    organization {
        int organization_id PK
        int parent_organization_id
        int type_id
        string name
        string country
        int hierarchy_level
        boolean hide_from_menu_manager
        boolean fran_connect_managed
        boolean active
    }
    temp_dmb_store_warning_types {
        int franchise_number
        int store_number
        int xref_id
        int xref_type
        int warning_type
        int organization_id
    }
    temp_dmb_xref_image {
        int xref_id
        int xref_type
        string image_file
        int warning_type
    }
    temp_work_archive {
        int row_id
        int root_organization_id
        int xref_id
        int xref_type
        date disable_date
        boolean archive
    }
    work_display_group_update {
        int display_group_id PK
        int display_group_parent_id
        int display_group_level
        string display_name
        string internal_name
        int is_of_type
        boolean is_large
        boolean is_regular
        boolean is_thin
        boolean is_stuffed
        boolean is_pretzel
        boolean is_deep_dish
    }
    work_menu_redesign_image {
        int row_id
        int root_organization_id PK
        string country
        int service_location_id
        int language_id PK
        boolean is_default_language
        string culture_code
        int xref_type_id PK
        string xref_type_name
        boolean xref_type_is_default
        int id PK
        int map_to_alt_id
        string name
        int category_id
        string category_name
        int image_id
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
    work_online_load_test_store_mapping {
        int row_id PK
        int prod_organization_id
        int prod_location_number
        int prod_franchise_number
        int prod_store_number
        string prod_file_name
        int organization_id
        int location_number
        int franchise_number
        int store_number
        string file_name
        int block_id
        boolean is_vertex_set
        string main_division
        string address1
        string address2
        string additional_columns_omitted
    }
    work_online_product_image_import_copy {
        int image_id PK
        string image_file_source
        string image_file_source_url_extra
        string image_file_destination
        string image_file_destination_url_extra
        boolean image_copied
        boolean image_copy_failed
        string image_copy_failure_reason
    }
    work_online_product_image_import_mm_data {
        int xref_type PK
        int xref_id PK
        int service_location_id
        int image_id PK
        string image_variant
        string image_file_name
        string image_type
        string image_alt_text
    }
    common_menu_category ||--o{ work_menu_redesign_image : category_id
    common_menu_service_location ||--o{ work_menu_redesign_image : service_location_id
    common_menu_service_location ||--o{ work_menu_redesign_image_source : service_location_id
    corp_menu_item ||--o{ work_menu_redesign_image_source : item_id
    corp_menu_special ||--o{ work_menu_redesign_image_source : special_id
    language ||--o{ work_menu_redesign_image_source : language_id
    organization ||--o{ temp_dmb_store_warning_types : organization_id
    organization ||--o{ temp_work_archive : root_organization_id
    organization ||--o{ work_menu_redesign_image_source : root_organization_id
    work_menu_redesign_image ||--o{ work_menu_redesign_image_source : id
    work_menu_redesign_image ||--o{ work_menu_redesign_image_source : xref_type_id
    work_menu_redesign_image ||--o{ work_menu_redesign_image_source_qa : id
    work_menu_redesign_image ||--o{ work_menu_redesign_image_source_qa : xref_type_id
    work_online_load_test_store_mapping ||--o{ temp_work_archive : row_id
    work_online_load_test_store_mapping ||--o{ work_menu_redesign_image : row_id
    work_online_load_test_store_mapping ||--o{ work_menu_redesign_image_source : row_id
    work_online_product_image_import_copy ||--o{ work_menu_redesign_image : image_id
    work_online_product_image_import_copy ||--o{ work_menu_redesign_image_source : image_id
    work_online_product_image_import_mm_data ||--o{ temp_dmb_store_warning_types : xref_id
    work_online_product_image_import_mm_data ||--o{ temp_dmb_xref_image : xref_id
    work_online_product_image_import_mm_data ||--o{ temp_work_archive : xref_id
```
