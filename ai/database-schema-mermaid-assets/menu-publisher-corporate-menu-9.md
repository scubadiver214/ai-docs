# menu-publisher - corporate-menu 9

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
        boolean is_out_of_stock
        boolean archived
        datetime archived_on
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        string additional_columns_omitted
    }
    corp_menu_topping_availability {
        int service_location_id PK
        int topping_id PK
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
    corp_menu_topping_image {
        int topping_id PK
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
    corp_menu_topping_language {
        int topping_id PK
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
    corp_menu_topping_xref_category {
        int topping_id PK
        int category_id PK
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    common_menu_category ||--o{ corp_menu_topping : category_id
    common_menu_display_group ||--o{ corp_menu_topping : display_group_id
    common_menu_service_location ||--o{ corp_menu_topping_availability_restore : service_location_id
    corp_menu_topping ||--o{ corp_menu_topping_availability_restore : topping_id
```
