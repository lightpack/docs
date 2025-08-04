# Lightpack Audit Facility

The Lightpack **Audit** module provides a simple, explicit, and powerful way to track and record changes, actions, and events in your application.

Auditing is essential for security, compliance, debugging, and accountability. With Lightpack's audit facility, you can log who did what, when, and where—across your entire app.

## Features
- **Explicit API:** Log audit events anywhere in your code—no hidden hooks.
- **Configurable:** Choose what to audit and when to log.
- **Flexible Storage:** Audit logs are stored in a dedicated `audit_logs` table.
- **Rich Context:** Store user, action, before/after values, URL, IP, and more.
- **Easy Querying:** Use the `AuditLog` model to fetch and analyze audit data.

## Practical Use Cases for Audit Logging

Lightpack's audit facility is ideal for a wide range of real-world scenarios, including:

1. **User Account Changes:**
   - Track profile updates, password changes, email changes, and other user modifications.
2. **Role & Permission Management:**
   - Audit who assigned or removed roles/permissions to users for security and compliance.
3. **Sensitive Data Access:**
   - Log when users view, export, or download sensitive data (e.g., financial, medical records).
4. **CRUD Operations on Core Entities:**
   - Record creation, update, and deletion of business-critical records (posts, orders, invoices, etc.).
5. **System-Generated Events:**
   - Capture background jobs, scheduled tasks, or automated cleanups (e.g., token expiry, cron jobs).
6. **Security Events:**
   - Log logins, failed logins, password resets, and suspicious activity for audit and investigation.
7. **Multi-Entity Actions:**
   - Track cascading deletes/updates (e.g., deleting a user and all their posts or permissions).
8. **Regulatory Compliance:**
   - Provide a full audit trail for GDPR, HIPAA, SOX, or other compliance requirements.
9. **Admin Actions:**
   - Monitor all admin panel actions for accountability and transparency.
10. **API Usage:**
    - Log API calls that mutate data, including which user or system performed them.

---

## Migration

Create schema migration file:

```cli
php console create:migration --support=audits
```

Run migration:

```cli
php console migrate:up
```

## Schema

The `audit_logs` table has the following fields:

| Column         | Type      | Description                        |
| -------------- | --------- | ---------------------------------- |
| id             | bigint    | Primary key                        |
| user_id        | bigint    | Acting user ID (nullable)          |
| action         | varchar   | Action performed                   |
| audit_type | varchar   | Class/model/table                  |
| audit_id   | bigint    | Affected record ID (nullable)      |
| old_values     | text      | JSON-encoded old values (nullable) |
| new_values     | text      | JSON-encoded new values (nullable) |
| url            | varchar   | URL of the request (nullable)      |
| ip_address     | varchar   | IP address (nullable)              |
| user_agent     | varchar   | User agent (nullable)              |
| created_at     | datetime  | Timestamp                          |
| updated_at     | datetime  | Timestamp                          |

---

## Usage

The module consists of:
- `AuditLog.php` (the model representing audit_logs table)
- `Audit.php` (the audit service class exposing auditing capabilities)
- `AuditTrait.php` (make any model auditable using this trait)

### Logging an Audit Event
Call the static `log` method from anywhere (controller, service, etc.):

```php
Audit::log([
    'user_id'        => $userId,              // (int|null) ID of the acting user
    'action'         => 'update',             // (string) Action performed
    'audit_type' => 'User',          // (string) table
    'audit_id'   => $user->id,            // (int|null) ID of affected record
    'old_values'     => $old,                 // (array|null) Before values
    'new_values'     => $new,                 // (array|null) After values
    'url'            => $request->url(),      // (string|null) Request URL
    'ip_address'     => $request->ip(),       // (string|null) IP address
    'user_agent'     => $request->userAgent(),// (string|null) User agent
    'message'        => 'User profile updated', // (string|null) Optional message
]);
```

> All fields are optional except `action` and `audit_type`.

---

## Logging System-Generated Events and Messages

You can log system-generated events (with no user) or add custom messages to any audit event using the `message` field. This is useful for tracking background jobs, cron tasks, or internal system actions.

### Example: System-Generated Event

```php
Audit::log([
    'action'     => 'system_cleanup',
    'audit_type' => 'Token',
    'audit_id'   => $tokenId,
    'old_values' => $tokenData,
    'user_agent' => 'system/cronjob',
    'message'    => 'Expired token removed by scheduled job',
]);
```

### Example: User Event with a Message

```php
Audit::log([
    'user_id'    => $userId,
    'action'     => 'login',
    'audit_type' => 'User',
    'audit_id'   => $userId,
    'ip_address' => $request->ip(),
    'user_agent' => $request->userAgent(),
    'message'    => 'User logged in successfully from web portal',
]);
```

> The `message` field is a free-form text column for any extra context, system notes, or explanations you want to attach to an audit entry.

---

## Audting Models

Make any model audit capable using `AuditTriat` on it. This trait exposes `audit()` method on the model. You do not need to pass `audit_id` and `audit_type` values because `audit()` method automatically sets:

- `audit_id` with model id,
- `audit_type` with model table name.

```php
class User extends Model
{
    use AuditTrait;
}
```

```php
$user = new User(23);

$user->audit([
    'action'     => 'update',
    'new_values' => $user->toArray(),
    'message'    => 'User details updated.',
]);
```

## Logging Multi-Entity Actions

If a single action (such as deleting a role) causes changes in multiple tables or entities (e.g., roles and their attached permissions), **log a separate audit entry for each impacted entity**. This ensures your audit trail is complete and granular.

### Example: Cascading Delete (Role and Permissions)

Suppose you delete a role and it cascades to permissions:

```php
// Log the role deletion
Audit::log([
    'user_id'    => $adminId,
    'action'     => 'delete',
    'audit_type' => 'Role',
    'audit_id'   => $role->id,
    'old_values' => $role->toArray(),
    'message'    => 'Role deleted by admin',
]);

// Log each affected permission
foreach ($permissions as $permission) {
    Audit::log([
        'user_id'    => $adminId,
        'action'     => 'cascade_delete',
        'audit_type' => 'Permission',
        'audit_id'   => $permission->id,
        'old_values' => $permission->toArray(),
        'message'    => 'Permission unlinked/deleted due to role deletion',
    ]);
}
```

> Each audit entry's `audit_type` and `audit_id` point to the precise entity affected, making querying and reporting straightforward.

---

## AuditLog Helpers for Building Audit Views

The audit facility provides several helpers to make building audit log screens and APIs easier:

### 1. Resolving User Information: `user()`

The `AuditLog` model provides a `user()` relation that resolves the `user_id` field to an `AuthUser` instance.

```php
$log = AuditLog::query()->where('user_id', 5)->one();
echo $log->user->name; // Outputs the user's name
```

If the action was performed by the system (no user), `$log->user` will be `null`.

### 2. Computing Changes

You can compute the difference between `old_values` and `new_values` using the `diff()` method:

```php
$log = AuditLog::query()->where('id', 123)->one();
$diff = $log->diff();
// $diff['added'] contains fields added or changed
// $diff['removed'] contains fields removed or changed
```

#### Example
Suppose:
```php
$log->old_values = ['name' => 'Alice', 'email' => 'alice@old.com', 'status' => 'active'];

$log->new_values = ['name' => 'Alice B.', 'email' => 'alice@new.com', 'role' => 'admin'];

$diff = $log->diff();
```
Result:
```php
[
    'added' => [
        'name' => 'Alice B.',         // changed value
        'email' => 'alice@new.com',   // changed value
        'role' => 'admin',            // new field
    ],
    'removed' => [
        'name' => 'Alice',                // old value
        'email' => 'alice@old.com',       // old value
        'status' => 'active',             // removed field
    ]
]
```
- Fields present in both but changed appear in both `added` (new value) and `removed` (old value).
- Fields only in `old_values` are in `removed`.
- Fields only in `new_values` are in `added`.

### 3. Query Scopes & Filters

You can filter audit logs by user, action, or audit type using the `filters()` method:

```php
// By user
$logs = AuditLog::filters(['user' => 5])->all();

// By action
$logs = AuditLog::filters(['action' => 'update'])->all();

// By audit type
$logs = AuditLog::filters(['auditType' => 'User'])->all();

// Combine filters
$logs = AuditLog::filters([
    'user' => 5,
    'action' => 'delete',
    'auditType' => 'Post',
])->all();
```

These helpers make it much easier to build rich, user-friendly audit log screens and APIs.

---

## Content Versioning with Audit

### What is Content Versioning?
Content versioning is the practice of tracking, storing, and (optionally) restoring previous states of your application's data. This is essential for features like undo/redo, rollback, historical inspection, and regulatory compliance. Versioning is especially valuable for business-critical records (posts, orders, invoices, etc.) where you may need to see or revert changes over time.

### Why Content Versioning?
- **Accountability:** Know exactly what changed, when, and by whom.
- **Recovery:** Restore previous versions after mistakes or unwanted edits.
- **Auditability:** Demonstrate compliance with regulations by showing historical changes.
- **Transparency:** Enable users/admins to review or compare past versions.

### How to Achieve Content Versioning in Lightpack
Lightpack's Audit module already provides the foundation for content versioning, thanks to its ability to log both `old_values` and `new_values` for every change. With minimal extra code, you can leverage these audit logs to implement versioning for any model or entity.

#### Basic Versioning Pattern
1. **Log Changes:** Use `Audit::log()` to record every create, update, or delete, including full `old_values` and `new_values`.
2. **Fetch Versions:** Use the `AuditLog` model to retrieve all audit entries for a given record (by `audit_type` and `audit_id`).
3. **Restore a Version:** To "roll back" to a previous state, fetch the desired audit log and set your model's attributes to the `old_values` (for undo) or `new_values` (for redo), then save.

#### Example: Restoring a Previous Version
```php
// Fetch the audit log entry you want to restore
$auditLog = AuditLog::query()
    ->where('audit_type', 'User')
    ->where('audit_id', $user->id)
    ->where('id', $auditLogId)
    ->one();

// Restore the old values
$user->setAttributes($auditLog->old_values);
$user->save();
```

---
