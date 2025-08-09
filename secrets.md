# Lightpack Secrets

Lightpack Secrets is a robust, framework-level solution for managing sensitive credentials (API keys, tokens, passwords, and more) with end-to-end encryption, explicit APIs, and full developer control.

---

## Why Use Lightpack Secrets?
- **Protect sensitive credentials:** Prevent accidental leaks and unauthorized access.
- **Zero plaintext at rest:** All secrets are always encrypted in the database.
- **Multi-tenant ready:** Secrets can be scoped to users, apps, or organizations.
- **Framework-native:** Direct support

---


## Encryption & Key Management
- **AES-256-CBC encryption** with random IV per secret (non-deterministic ciphertext).
- **Master key** is required and must be set in your config/env as `app.secrets_key`.
- **Key never stored in DB**—only in your environment/config.
- **No backdoor:** If you lose the key, all secrets are irretrievable.
- **Key rotation:** Built-in method to re-encrypt all secrets with a new key (see below).

---

## Data Model: `secrets` Table
| Column      | Type      | Notes                                     |
|-------------|-----------|-------------------------------------------|
| id          | int       | PK, auto-increment                        |
| key         | string    | Unique secret key (per group/owner)       |
| value       | text      | Encrypted secret value (JSON-encoded)     |
| group       | string    | (default: 'global') Logical grouping      |
| owner_id    | int       | (nullable) User/org ID                    |
| created_at  | timestamp | Creation time                             |
| updated_at  | timestamp | Last update time                          |
| UNIQUE KEY  | (`key`, `group`, `owner_id`)                          |

---


## Migration

Create schema migration file:

```cli
php console create:migration --support=secrets
```

Run migration:

```cli
php console migrate:up
```
---

## Configuration
- Set your secrets key in your config or `.env`:
  ```env
  APP_KEY=your-secret-key-string
  ```

You can auto-generate `APP_KEY` using following command:

```cli
php console app:key
```

> Never commit secrets keys to version control.

---

## Core API (via Model Trait)
On any model using `SecretsTrait`:

```php
class User extends Model
{
    use Lightpack\Secrets\SecretsTrait;
}
```

```php
$user->secrets()->set('api_token', 'secret-value');
$user->secrets()->get('api_token');
$user->secrets()->delete('api_token');
```

## Global/App Settings
Instantiate via container or you can type hint **Secrets** as your controller's method dependency.

### Set a Secret
```php
app('secrets')
    ->group('users')
    ->owner(42)
    ->set('api_token', 'secret-value');
```

### Get a Secret
```php
$token = app('secrets')
    ->group('users')
    ->owner(42)
    ->get('api_token');
```

### Delete a Secret
```php
app('secrets')
    ->group('users')
    ->owner(42)
    ->delete('api_token');
```

### Change Group/Owner Scope
```php
app('secrets')
    ->group('global')
    ->owner(null)
    ->set('service_key', 'xyz');
```

---

## Key Rotation (Re-encrypt All Secrets)

Lightpack supports secrets rotation by exposing `rotateKey()` method:

```php
$oldKey = get_env('OLD_SECRETS_KEY');
$newKey = get_env('NEW_SECRETS_KEY');

// Rotate secrets in batch size of 100 (optional)
$result = app('secrets')->rotateKey($oldKey, $newKey, 100); 

// $result = ['success' => <count>, 'fail' => <count>]
// log the result for inspection
```

- **Efficient:** Processes secrets in batches (default: 500, configurable).
- **Safe:** Only updates secrets that decrypt successfully; failures are reported.
- **Flexible:** Use in CLI, web, or migration context—your choice.

### Key Rotation Checklist
- **Backup both old and new keys** in a secure password manager or vault before rotating.
- **Test rotation in staging** before production.
- **Update your config/env** to use the new key after rotation.
- **Never delete the old key until you confirm all secrets are accessible with the new key.**

---

## Advanced Usage
- **Batch size for rotation:** Tune the batch size for your environment (memory vs. speed).
- **Type safety:** Secrets are JSON-encoded, so you can store arrays, objects, or scalars.
- **Integration:** Use in CLI scripts, web admin panels, or migrations as needed.
---

## Security Model
- **No caching:** Secrets are always fetched from DB and decrypted on demand for maximum security.
- **No plaintext at rest:** All values are encrypted.
- **No fallback:** If the key is lost, secrets are unrecoverable.
- **No accidental logging:** Never log or expose decrypted secrets.

---
