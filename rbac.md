# Lightpack RBAC (Role-Based Access Control)

A minimal, efficient, and modular Role-Based Access Control (RBAC) implementation for the Lightpack PHP framework.

---

## Features
- **RbacTrait for User Models:** Add roles and permissions to any model using a single trait.
- **ORM-Centric:** All relationships return query or collection objects for full chaining and efficiency.
- **Pivot Table Management:** Assign and remove roles/permissions using expressive, ORM-native methods.
- **Integrated Filtering:** Filter users by role or permission with scope methods (`scopeRole`, `scopePermission`).
- **Highly Readable API:** Methods like `can`, `hasRole`, etc., are clear and intuitive.
- **Modular:** RBAC is opt-in. No pollution of the base user model.

---

## Database Schema

Tables created by the included migration:
- `roles`: Stores all roles.
- `permissions`: Stores all permissions.
- `user_role`: Pivot table linking users and roles.
- `role_permission`: Pivot table linking roles and permissions.

---

## Migration

Run this command to generate a migration file:

```cli
php console create:migration create_table_roles_permissions
```

Use the following code for the up() and down() methods:

```php
public function up(): void
{
    $this->create('roles', function (Table $table) {
        $table->id();
        $table->varchar('name', 100)->unique();
        $table->varchar('label', 150)->nullable();
        $table->timestamps();
    });

    $this->create('permissions', function (Table $table) {
        $table->id();
        $table->varchar('name', 100)->unique();
        $table->varchar('label', 150)->nullable();
        $table->timestamps();
    });

    $this->create('user_role', function (Table $table) {
        $table->column('user_id')->type('bigint');
        $table->column('role_id')->type('bigint');
        $table->unique(['user_id', 'role_id']);
        $table->foreignKey('user_id')->references('id')->on('users');
        $table->foreignKey('role_id')->references('id')->on('roles');
    });

    $this->create('role_permission', function (Table $table) {
        $table->column('role_id')->type('bigint');
        $table->column('permission_id')->type('bigint');
        $table->unique(['role_id', 'permission_id']);
        $table->foreignKey('role_id')->references('id')->on('roles');
        $table->foreignKey('permission_id')->references('id')->on('permissions');
    });
}

public function down(): void
{
    $this->drop('role_permission');
    $this->drop('user_role');
    $this->drop('permissions');
    $this->drop('roles');
}
```


## Usage

**Add RbacTrait to Your User Model:**

```php
class User extends Model {
    use RbacTrait;
}
```

### Assign/Remove Roles
   ```php
   // Attach a role by ID
   $user->roles()->atatch($roleId); 

   // Remove a role by ID
   $user->roles()->detach($roleId); 
   ```

### Check Roles & Permissions
   ```php
   // true/false by role name or ID
   $user->hasRole('admin'); 
   
   // true/false by permission name or ID
   $user->can('edit_post'); 

   // true/false
   $user->cannot('delete_post'); 

   // true if user has 'superadmin' role
   $user->isSuperAdmin(); 
   ```

### Fetch All Roles or Permissions

```php
// Collection of Role models
$user->roles; 

// Collection of Permission models (via roles)
$user->permissions; 
```

### Filter Users by Role or Permission

```php
// By role name
$admins = User::filters(['role' => 'admin'])->all();

// By role ID
$editors = User::filters(['role' => 2])->all();

// By permission name
$canEdit = User::filters(['permission' => 'edit_post'])->all();

// By permission ID
$canDelete = User::filters(['permission' => 11])->all();

// By role and permission both
$filtered = User::filters(['role' => 'admin', 'permission' => 'edit_post'])->all();
```

### Fetch Role Permissions

```php
$role = new Role(23);

// Collection of permissions assigned to this role
$role->permissions();
```

---