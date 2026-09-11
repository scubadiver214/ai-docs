# menu-publisher - work-temp 4

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
    work_org_menu_item_availability_400125 {
        int organization_id PK
        int item_id PK
        int service_location_id PK
        decimal price
        decimal price_minimum_allowed
        decimal price_maximum_allowed
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    work_org_menu_item_availability_prod_restore_11102025 {
        int organization_id PK
        int item_id PK
        int service_location_id PK
        boolean active
        decimal price
        decimal price_minimum_allowed
        decimal price_maximum_allowed
        decimal topping_price
        decimal topping_price_minimum_allowed
        decimal topping_price_maximum_allowed
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
    work_update_menu_request_information {
        int special_id PK
        string internal_name
        string name
        string description
        string abbreviation
        string image_file_name
        string calories
        boolean orderable_instore
        boolean orderable_online
        boolean orderable_3pd
        decimal price
        decimal price_maximum_allowed
        decimal price_minimum_allowed
        int display_group_id_pos
        int display_group_id_3pd
        int display_order
    }
    work_update_menu_request_service_locations_availability {
        int organization_id PK
        int service_location_id PK
    }
    work_update_menu_request_service_locations_language {
        int organization_id PK
        int language_id PK
        int service_location_id PK
    }
    common_menu_service_location ||--o{ work_org_menu_special_availability_snapshot_20260515 : service_location_id
    corp_menu_special ||--o{ work_org_menu_special_availability_snapshot_20260515 : special_id
    organization ||--o{ work_org_menu_special_availability_snapshot_20260515 : organization_id
```
