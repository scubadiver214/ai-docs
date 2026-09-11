# master-menu - organization-menu-overrides 4

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
    common_menu_upsell_category {
        int upsell_category_id PK
        string name
        string description
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    org_menu_item_command_screen_display_empty {
        int organization_id PK
        int display_id PK
        boolean is_empty
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    org_menu_item_command_screen_schedule {
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
    org_menu_item_command_screen_schedule_restore {
        int process_id PK
        datetime process_saved_on
        boolean process_removed
        boolean process_removed_with_restore
        datetime process_removed_on
        int organization_id PK
        int item_id PK
        int id
        int display_id
        string description
        date start_date
        date end_date
        datetime start_time
        datetime end_time
        boolean is_mandatory
        boolean all_day_schedule
        string additional_columns_omitted
    }
    org_menu_item_topping_configuration {
        int organization_id PK
        int item_id PK
        int free_count
        int minimum_count
        int maximum_count
        int maximum_charged_count
        int regular_level
        string updated_by_login_id
        string updated_by_first_name
        string updated_by_last_name
        datetime updated_on
    }
    org_menu_item_topping_configuration_deliverect {
        int organization_id PK
        int item_id PK
        int free_count
        int minimum_count
        int maximum_count
        int maximum_allowed_level
        int maximum_allowed_level_sauce
        int maximum_allowed_level_cheese
        int maximum_allowed_level_premium
        datetime updated_on
    }
    org_menu_item_topping_size {
        int organization_id PK
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
    org_menu_special {
        int organization_id PK
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
    common_menu_display_group ||--o{ org_menu_special_availability : display_group_id
    common_menu_upsell_category ||--o{ org_menu_special : upsell_category_id
    org_menu_item_command_screen_display_empty ||--o{ org_menu_item_command_screen_schedule_restore : display_id
    org_menu_item_command_screen_schedule ||--o{ org_menu_item_command_screen_schedule_restore : id
    org_menu_special ||--o{ org_menu_special_availability_schedule : special_id
```
