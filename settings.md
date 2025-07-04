# Lightpack Settings

Modern web applications need dynamic, database-driven settings to power everything from feature toggles and branding to user preferences and operational parameters.

---

## Why Database-Driven Settings?
- **Instant updates:** Change app behavior without code deploys.
- **Multi-tenant support:** Each user, organization, product, or any model can have their own settings.
- **Feature toggles:** Enable/disable features on the fly.
- **Central management:** Manage all settings from an admin UI or API.
- **Polymorphic:** Settings can be attached to any model, not just users or orgs.

---

## Data Model

**settings table**

| Column      | Type         | Notes                                              |
|-------------|--------------|----------------------------------------------------|
| id          | int          | PK, auto-increment                                 |
| key         | string(150)  | Setting key (not necessarily unique globally)      |
| value       | text         | Serialized or plain value                          |
| key_type    | string(25)   | (optional) for type hints (int, bool, json, etc.)  |
| group       | string(150)  | Logical namespace (e.g., 'global', 'users', etc.)  |
| owner       | bigint       | The owner id within the group (nullable for global)|
| updated_at  | timestamp    | Last update time                                   |

**Indexes:**
- Composite index on (`group`, `owner`, `key`) for efficient lookups.

---


## Migration

Run this command to generate a migration file:

```cli
php console create:migration create_table_settings
```

Use the following code for the up() and down() methods:

```php
public function up(): void
{
    $this->create('settings', function (Table $table) {
        $table->id();
        $table->varchar('key', 150);
        $table->varchar('key_type', 25)->nullable();
        $table->text('value');
        $table->varchar('group', 150)->default('global');
        $table->column('owner_id')->type('bigint')->attribute('unsigned')->nullable();
        $table->timestamps();
        $table->index(['group', 'owner_id', 'key']);
    });
}

public function down(): void
{
    $this->drop('settings');
}
```

---

## Configuration

Look into `config/settings.php` file for configuration related options:

```php
'settings' => [
    // Enable or disable caching for settings
    'cache' => true,

    // Cache TTL (time to live) in seconds (e.g., 3600 = 1 hour)
    'ttl' => 3600,
],
```

---

## Core API (via Model Trait)
On any model using `SettingsTrait`:

```php
class User extends Model
{
    use Lightpack\Settings\SettingsTrait;
}
```

```php
$user->settings()->set('key', 'value');
$value = $user->settings()->get('key', 'default');
$all = $user->settings()->all();
$user->settings()->forget('key');
```

## Global/App Settings

Instantiate via container or you can type hint **Settings** as your controller's method dependency.

```php
app('settings')
    ->group('global')
    ->owner(null)
    ->set('site_name', 'Lightpack Demo');
```

```php
$name = app('settings')
    ->group('global')
    ->owner(null)
    ->get('site_name', 'Default Name');
```

## Owner/Tenant/Model specific

```php
app('settings')
    ->group('users')
    ->owner($userId)
    ->set('theme', 'dark');
```

```php
app('settings')
    ->group('products')
    ->owner($productId)
    ->set('tax_rate', 0.18);
```

Or, better using the trait on the model:

```php
$user = new User(23);
$user->settings()->set('theme', 'dark');
```

```php
$product = new Product(100);
$product->settings()->set('tax_rate', 0.18);
```

---

## Design Principles

- **Explicit API:** No global helpers or magic loading—always opt-in via trait or direct instantiation.
- **No Magic:** Settings are loaded only when requested, not injected everywhere.
- **Type Safety:** Values are stored with explicit type hints (`key_type`), and serialization/casting is handled explicitly.
- **Extensible:** Add new groups, owners, or custom serialization as needed.
- **Cache-Friendly:** Caches settings per group/owner scope, with auto-invalidation on update.
- **Polymorphic:** Any model or logical group can have settings—no schema changes needed for new types.
- **Isolation:** Always enforce tenant/model ownership before reading/writing settings.

---

## Advanced Features

- **JSON/Array Support:** Store and retrieve structured data with automatic (de)serialization.
- **Caching:** Settings are cached per group/owner (using Lightpack's cache system), and cache is invalidated on update/delete for consistency and performance.
- **Type Handling:** Values are automatically cast to the declared type (`int`, `bool`, `float`, `json`, etc.) on retrieval.
- **Multi-tenant Ready:** Use `group`/`owner` for per-tenant, per-user, per-product, etc. settings.
- **Environment Awareness:** Optionally fallback to `.env` or config files if not found in DB.
---