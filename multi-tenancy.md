# Multi-Tenancy

Lightpack provides built-in support for **row-level multi-tenancy** through the `TenantModel` class—making it simple to build SaaS applications and multi-tenant systems.

---

## Understanding Multi-Tenancy

### What is a Tenant?

A **tenant** is an isolated group of users who share common access to your application with specific privileges. In practical terms:

- **SaaS Applications:** A tenant is typically a customer organization (e.g., Company A, Company B)
- **E-commerce Platforms:** A tenant could be an individual store or merchant
- **Educational Systems:** A tenant might be a school or university
- **Healthcare Systems:** A tenant could be a hospital or clinic

**Key Principle:** Each tenant's data must be completely isolated from other tenants—users in Tenant A should never see or access data belonging to Tenant B.

### Real-World Example

Imagine you're building a project management SaaS application:

```table
Tenants in your application:
-------------------------------------------------
Tenant ID | Organization Name    | Users
-------------------------------------------------
1         | Acme Corporation     | john@acme.com, jane@acme.com
2         | Beta Industries      | bob@beta.com, alice@beta.com
3         | Gamma Solutions      | charlie@gamma.com
-------------------------------------------------
```

When `john@acme.com` logs in:
- He should only see projects, tasks, and files belonging to **Acme Corporation** (Tenant 1)
- He should never see data from Beta Industries or Gamma Solutions
- All his actions (create, update, delete) should only affect Tenant 1's data

This isolation is what multi-tenancy provides.

---

## Multi-Tenancy Architectural Approaches

There are 3 main architectural approaches to multi-tenancy:

### Approach 1: Database-per-Tenant (Separate Database)

```
Tenant 1 → database_tenant_1
Tenant 2 → database_tenant_2
Tenant 3 → database_tenant_3
```

**Pros:** Maximum isolation, easy backup per tenant  
**Cons:** Resource intensive, complex connection management, scaling issues

---

### Approach 2: Schema-per-Tenant (Separate Schema)

```
Same Database:
  ├─ schema_tenant_1 (tables: posts, users)
  ├─ schema_tenant_2 (tables: posts, users)
  └─ schema_tenant_3 (tables: posts, users)
```

**Pros:** Good isolation, easier than separate databases  
**Cons:** Schema switching overhead, complex migrations

---

### Approach 3: Row-Level Tenancy (Shared Tables with Discriminator Column) ⭐

```table
posts table:
-------------------------------------------------
id | tenant_id | title
-------------------------------------------------
1  | 1         | Post A
2  | 1         | Post B
3  | 2         | Post C  ← Different tenant
```

**Pros:** Simple, scalable, database-agnostic, efficient  
**Cons:** Requires careful query filtering, less isolation

Lightpack chose **Approach 3 (Row-Level Tenancy)** because it provides the best balance for most applications:

---

## Quick Start

### Step 1: Create Tenant-Aware Schema

Add a tenant identifier column `tenant_id` to your tables:

```sql
CREATE TABLE posts (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    tenant_id INT NOT NULL,
    title VARCHAR(255) NOT NULL,
    content TEXT,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    
    INDEX idx_tenant_id (tenant_id)
);
```

<p class="tip"><b>Important:</b> Always index your tenant column for optimal performance.</p>

### Step 2: Create Tenant Models

Use the console command to generate a tenant model:

```terminal
php console create:model Post --tenant
```

This creates a model that extends `TenantModel`:

```php
<?php

namespace App\Models;

use Lightpack\Database\Lucid\TenantModel;

class Post extends TenantModel
{
    protected $table = 'posts';
    protected $tenantColumn = 'tenant_id';
}
```

### Step 3: Set Tenant Context

Set the tenant context:

```php
// e.g. after authenticating user
TenantModel::setContext($user->tenant_id);
```

### Step 4: Use Models Normally

Once tenant context is set, all queries are automatically scoped:

```php
// Only returns posts for current tenant
$posts = Post::query()->all();

// Auto-assigns tenant_id on create
$post = new Post();
$post->title = 'My Post';
$post->save(); // tenant_id automatically set

// Only updates current tenant's posts
$post->title = 'Updated Title';
$post->save();

// Only deletes current tenant's posts
$post->delete();
```

---

## How It Works

### Automatic Query Filtering

The `TenantModel` uses a `globalScope()` to automatically add a `WHERE tenant_id = ?` clause to all queries:

```php
// Your code
Post::query()->where('status', 'published')->all();

// Actual SQL executed
SELECT * FROM posts WHERE status = 'published' AND tenant_id = 1
```

This applies to all operations:
- **SELECT** - Only fetch current tenant's records
- **UPDATE** - Only update current tenant's records
- **DELETE** - Only delete current tenant's records
- **COUNT** - Only count current tenant's records

### Automatic Tenant Assignment

When creating new records, `TenantModel` automatically sets the tenant column:

```php
$post = new Post();
$post->title = 'New Post';
$post->save();
// tenant_id is automatically set to current tenant
```

This works for both `save()` and `insert()` methods, ensuring you never accidentally create records without a tenant.

---

## Customizing Tenant Column

By default, `TenantModel` uses `tenant_id` as the tenant column. You can customize this per model:

```php
class Article extends TenantModel
{
    protected $table = 'articles';
    protected $tenantColumn = 'site_id';  // Custom column name
}
```

You can also specify this when generating the model:

```terminal
php console create:model Article --tenant --tenant-column=site_id
```

---

## Tenant Resolution Strategies

You set the tenant context using `TenantModel::setContext()`. Here are common patterns for different application types:

### Strategy 1: Session-Based

Best for traditional web applications with cookie-based authentication.

```php
// In your route filter
use Lightpack\Database\Lucid\TenantModel;

$user = auth()->user();
TenantModel::setContext(session()->get('tenant.id'));

// Or get from authenticated user
TenantModel::setContext($user->tenant_id);
```

**When to use:**
- Traditional multi-page web apps
- Cookie-based authentication
- Server-side session management

### Strategy 2: JWT/Token-Based (API)

Best for RESTful APIs, mobile apps, and SPAs.

```php
// In your API authentication middleware
use Lightpack\Database\Lucid\TenantModel;

$user = auth()->user(); // From JWT token
TenantModel::setContext($user->tenant_id);
```

**When to use:**
- RESTful APIs
- Mobile applications
- Single Page Applications (SPAs)
- Stateless authentication

### Strategy 3: Domain/Subdomain-Based

Best for SaaS applications where each tenant has their own domain or subdomain (e.g., `acme.yourapp.com`).

```php
// In your route filter or middleware
use Lightpack\Database\Lucid\TenantModel;

$domain = request()->host();
$tenant = Tenant::query()->where('domain', $domain)->one();

if ($tenant) {
    TenantModel::setContext($tenant->id);
}
```

**When to use:**
- Custom domain per tenant
- Subdomain-based tenancy
- Public-facing tenant pages

<p class="tip"><b>Performance Tip:</b> Cache domain-to-tenant lookups to avoid database queries on every request.</p>

## Bypassing Tenant Isolation

Sometimes you need to access data across all tenants—for example, in admin dashboards or analytics. Use `queryWithoutScopes()` to bypass tenant filtering:

```php
// Access all posts from all tenants
$allPosts = Post::queryWithoutScopes()->all();

// Count posts across all tenants
$totalPosts = Post::queryWithoutScopes()->count();

// Get posts from specific tenant
$tenant2Posts = Post::queryWithoutScopes()
    ->where('tenant_id', 2)
    ->all();
```

<p class="tip"><b>Security Warning:</b> Only use <code>queryWithoutScopes()</code> in admin areas with proper authorization checks.</p>

---

## Summary

Lightpack's `TenantModel` provides:

✅ **Built-in multi-tenancy** - No packages needed  
✅ **Automatic isolation** - Queries filtered automatically  
✅ **Flexible tenant context** - Set via `setContext()` from any source  
✅ **Simple API** - `setContext()`, `getContext()`, `clearContext()`  
✅ **Production-ready** - Comprehensive test coverage  

This makes it ideal for building SaaS applications and multi-tenant systems with minimal complexity and maximum flexibility.
