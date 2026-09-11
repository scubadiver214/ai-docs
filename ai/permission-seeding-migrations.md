# Permission seeding in migrations

Use this guide whenever a migration introduces new authorization seed data, especially new `PermissionNames` values.

## Scope

This pattern applies to:

- `auth_permission`
- `auth_role`
- `auth_role_permission`

The shared helper lives in `Lce.MenuAdmin.Migrations/Migrations/AuthorizationSeedHelper.cs`.

## Required pattern

1. Add new permission name constants in `Lce.MenuAdmin.Core/Constants/PermissionNames.cs` first.
2. Represent seeded authorization data as arrays of helper inputs inside the migration:
   - `AuthorizationPermissionSeed`
   - `AuthorizationRoleSeed`
   - `AuthorizationRolePermissionMapping`
3. Call `AuthorizationSeedHelper.SeedAuthorization(...)` from `Up()`.
4. Call `AuthorizationSeedHelper.RemoveAuthorization(...)` from `Down()` unless the migration drops the entire auth tables as part of a greenfield baseline teardown.
5. Keep names centralized in `PermissionNames` and `RoleNames`; do not inline repeated authorization string literals across the migration.

## Why this pattern exists

- It keeps authorization seeding idempotent.
- It makes `Up()` and `Down()` symmetrical.
- It prevents each migration from re-implementing the same `auth_permission`, `auth_role`, and `auth_role_permission` SQL.
- It keeps `auth_permission.name` and `auth_role.name` as the stable join keys used by migrations.

## ID rules

Use a fixed ID when the seeded row is part of the canonical authorization catalog and the UUID must remain stable across environments and over time.

Use a generated ID when the row only needs a stable `name` key and no external contract depends on a fixed UUID.

Examples in the repo:

- Baseline auth permissions and roles use fixed IDs.
- Audit permissions and the `AuditReader` role use generated IDs.
- Store buy/sell permissions use fixed IDs because they extend the canonical permission catalog.

## Helper behavior

`AuthorizationSeedHelper.SeedAuthorization(...)` preserves the current repo behavior:

- `auth_role`: `on conflict (name) do nothing`
- `auth_permission`: `on conflict (name) do update`
- `auth_role_permission`: `on conflict do nothing`

`AuthorizationSeedHelper.RemoveAuthorization(...)` removes:

1. exact role-permission mappings
2. optional roles
3. optional permissions

That order keeps teardown safe around foreign keys.

## Example

```csharp
private static readonly AuthorizationPermissionSeed[] ExamplePermissions =
[
    AuthorizationPermissionSeed.FixedId(
        "00000000-0000-0000-0000-000000000999",
        PermissionNames.SomeFuturePermission,
        "Describe the permission clearly",
        isGlobal: false)
];

private static readonly AuthorizationRoleSeed[] ExampleRoles =
[
    AuthorizationRoleSeed.GeneratedId(
        RoleNames.SomeFutureRole,
        "Describe the role clearly")
];

private static readonly AuthorizationRolePermissionMapping[] ExampleMappings =
[
    new(RoleNames.SomeFutureRole, PermissionNames.SomeFuturePermission),
    new(RoleNames.Admin, PermissionNames.SomeFuturePermission)
];

public override void Up()
{
    AuthorizationSeedHelper.SeedAuthorization(
        this,
        roles: ExampleRoles,
        permissions: ExamplePermissions,
        rolePermissions: ExampleMappings);
}

public override void Down()
{
    AuthorizationSeedHelper.RemoveAuthorization(
        this,
        rolePermissions: ExampleMappings,
        roleNames: [RoleNames.SomeFutureRole],
        permissionNames: [PermissionNames.SomeFuturePermission]);
}
```

## Review checklist

- The permission name exists in `PermissionNames`.
- The migration uses the shared helper instead of inline authorization seed SQL.
- `Up()` and `Down()` cover the same authorization data.
- The descriptions are specific enough to explain the permission or role intent.
- Fixed IDs are only used when a stable UUID is required.