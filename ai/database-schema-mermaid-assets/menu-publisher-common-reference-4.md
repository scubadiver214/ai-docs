# menu-publisher - common-reference 4

```mermaid
erDiagram
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
    common_menu_service_location_language {
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
    common_menu_service_location_override {
        int root_organization_id PK
        int service_location_id PK
        int service_location_id_default_image
        int service_location_id_default_language
        int deliverect_language_id
        boolean price_update_allowed
        boolean price_is_markup
        decimal price_markup_rate
        boolean portal_supported
        int schedule_support_type_item
        int schedule_support_type_special
        int factura_print_device_type_id
        string factura_print_device_type_name
        datetime updated_on
    }
    common_menu_service_location_xref_order_origin {
        int service_location_id PK
        int sub_order_origin_id PK
        string sub_order_origin_code
        datetime updated_on
    }
    common_menu_topping_level {
        int level PK
        string internal_name
        string internal_description
        decimal default_count_as
        boolean active
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    common_menu_topping_level_language {
        int level PK
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
    corp_menu_addon_availability_restore {
        int service_location_id
        int addon_id
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
    corp_menu_topping_availability_restore {
        int service_location_id
        int topping_id
        decimal price
        decimal price_minimum_allowed
        decimal price_maximum_allowed
        int display_order
        boolean orderable
        boolean active
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    json_azure_published_file {
        int organization_id PK
        int service_location_id
        int content_version_id PK
        string content_version
        int file_type_id PK
        int file_id PK
        string file_connection_url
        string file_path
        string file_name
        datetime published_date
        boolean azure_write_bucket
        boolean azure_read_bucket
        boolean azure_archived
        datetime azure_archived_date
        string azure_archive_failure_reason
    }
    org_menu_addon_availability_restore {
        int organization_id
        int service_location_id
        int addon_id
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
    org_menu_item_availability_restore {
        int organization_id
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
        string additional_columns_omitted
    }
    org_menu_item_availability_schedule {
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
        boolean override_price_apply_markup
        decimal override_price_with
        boolean apply_to_future_orders
        string additional_columns_omitted
    }
    org_menu_item_availability_work_temp_427525_backup {
        int organization_id
        int item_id
        int service_location_id
        boolean active
        boolean orderable
        datetime updated_on
    }
    org_menu_special_availability_restore {
        int organization_id
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
    org_menu_special_availability_schedule {
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
    org_menu_special_availability_work_temp_427525_backup {
        int organization_id
        int special_id
        int service_location_id
        decimal price
        decimal price_maximum_allowed
        decimal price_minimum_allowed
        boolean active
        boolean orderable
        datetime updated_on
    }
    org_menu_topping_availability_restore {
        int organization_id
        int service_location_id
        int topping_id
        decimal price
        decimal price_minimum_allowed
        decimal price_maximum_allowed
        int display_order
        boolean orderable
        boolean active
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
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
    work_new_menu_request_389478_service_locations {
        int service_location_id
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
    work_org_menu_special_availability_snapshot_20260515 {
        int organization_id
        int special_id
        int service_location_id
        decimal price
        decimal price_minimum_allowed
        decimal price_maximum_allowed
        boolean active
    }
    common_menu_service_location ||--o{ corp_menu_addon_availability_restore : service_location_id
    common_menu_service_location ||--o{ corp_menu_item_availability_restore : service_location_id
    common_menu_service_location ||--o{ corp_menu_item_availability_schedule : service_location_id
    common_menu_service_location ||--o{ corp_menu_item_availability_temp : service_location_id
    common_menu_service_location ||--o{ corp_menu_special_availability_restore : service_location_id
    common_menu_service_location ||--o{ corp_menu_special_availability_schedule : service_location_id
    common_menu_service_location ||--o{ corp_menu_topping_availability_restore : service_location_id
    common_menu_service_location ||--o{ json_azure_published_file : service_location_id
    common_menu_service_location ||--o{ org_menu_addon_availability_restore : service_location_id
    common_menu_service_location ||--o{ org_menu_item_availability_restore : service_location_id
    common_menu_service_location ||--o{ org_menu_item_availability_schedule : service_location_id
    common_menu_service_location ||--o{ org_menu_item_availability_work_temp_427525_backup : service_location_id
    common_menu_service_location ||--o{ org_menu_special_availability_restore : service_location_id
    common_menu_service_location ||--o{ org_menu_special_availability_schedule : service_location_id
    common_menu_service_location ||--o{ org_menu_special_availability_work_temp_427525_backup : service_location_id
    common_menu_service_location ||--o{ org_menu_topping_availability_restore : service_location_id
    common_menu_service_location ||--o{ store_menu_item_availability_schedule : service_location_id
    common_menu_service_location ||--o{ store_menu_special_availability_schedule : service_location_id
    common_menu_service_location ||--o{ work_menu_redesign_image : service_location_id
    common_menu_service_location ||--o{ work_menu_redesign_image_source : service_location_id
    common_menu_service_location ||--o{ work_menu_redesign_image_source_qa : service_location_id
    common_menu_service_location ||--o{ work_new_menu_request_389478_service_locations : service_location_id
    common_menu_service_location ||--o{ work_online_product_image_import_mm_data : service_location_id
    common_menu_service_location ||--o{ work_org_menu_special_availability_snapshot_20260515 : service_location_id
```
