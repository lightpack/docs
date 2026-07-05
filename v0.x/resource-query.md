# Resource Query

When building resource APIs, every list endpoint in your application ends up writing the same boilerplate — reading filter parameters, applying sort conditions, eager loading relations, handling pagination, and shaping the response. The code works, but it is scattered across controllers and grows inconsistently over time.

Lightpack's `Model::resourceQuery()` makes it easy to handle HTTP query parameters from the current request and translates them into the appropriate ORM operations.

```php
public function index()
{
    $data = User::resourceQuery()
        ->allowFilters(['status', 'role', 'search'])
        ->allowSorts(['name', 'email', 'created_at'])
        ->allowIncludes(['profile', 'posts'])
        ->defaultSort('-created_at')
        ->paginate();

    return response()->json($data);
}
```

A client can now drive this endpoint entirely through the URL:

```
GET /api/users
    ?filter[status]=active
    &filter[role]=admin
    &sort=-created_at
    &include=profile,posts
    &fields=name,email
    &page=2
    &per_page=20
```

---

**No query parameter has any effect unless you explicitly allow it.**

If a client sends `?filter[password]=secret&sort=internal_score`, both parameters are silently ignored because neither appears in the allowlists you defined.

---

## Setting Up the Allowlists

### `allowFilters()`

Declares which filter keys the client may send. Each key maps to a scope method on the model — the same `scope*()` methods used by `Model::filters()`.

```php
Post::resourceQuery()
    ->allowFilters(['status', 'category', 'author_id', 'search']);
```

A client sends:

```
?filter[status]=published&filter[category]=news
```

The builder will call `$model->scopeStatus($builder, 'published')` and `$model->scopeCategory($builder, 'news')`. Any filter key not in the allowlist — or whose scope method does not exist on the model — is silently skipped.

The model defines the actual query logic:

```php
class Post extends Model
{
    protected function scopeStatus(Builder $query, string $value): void
    {
        $query->where('status', $value);
    }

    protected function scopeCategory(Builder $query, string $value): void
    {
        $query->where('category_slug', $value);
    }

    protected function scopeSearch(Builder $query, string $value): void
    {
        $query->where('title', 'LIKE', '%' . $value . '%');
    }
}
```

**Array-valued filters** are also supported natively. So `?filter[role][]=admin&filter[role][]=editor` is parsed into `['role' => ['admin', 'editor']]`, and is passed to the scope method.

```php
protected function scopeRole(Builder $query, array|string $value): void
{
    $query->whereIn('role', (array) $value);
}
```

### `allowSorts()`

Declares which columns the client may sort by. Unrecognised column names are ignored.

```php
Post::resourceQuery()
    ->allowSorts(['title', 'published_at', 'view_count']);
```

**Sort format**: a single `sort` query parameter, comma-separated. Prefix a column name with `-` for descending order.

```
?sort=-published_at          → ORDER BY published_at DESC
?sort=title                  → ORDER BY title ASC
?sort=-published_at,title    → ORDER BY published_at DESC, title ASC
```

**Default sort**: when the client sends no `sort` parameter, apply a fallback:

```php
->defaultSort('-created_at')
```

The default is applied only when `?sort` is absent. A client-provided sort always wins.

### `allowIncludes()`

Declares which relations the client may eager load. Dot-notation for nested relations is supported.

```php
Post::resourceQuery()
    ->allowIncludes(['author', 'comments', 'comments.author', 'tags']);
```

```
?include=author,tags                    → with('author', 'tags')
?include=comments.author               → with('comments.author')
?include=author,comments,tags          → with('author', 'comments', 'tags')
```

> Any relation not in the allowlist is silently dropped.

**Default includes**: relations to always load regardless of the `?include` parameter:

```php
->defaultIncludes(['author'])
```

Request includes are merged with the defaults and deduplicated.

### `allowCounts()`

Declares which relation row counts the client may request. Counts are loaded via efficient separate queries after the main result set is fetched — no joins, no subqueries in the main SQL.

```php
Article::resourceQuery()
    ->allowCounts(['comments', 'likes', 'shares']);
```

```
?count=comments,likes   →   withCount(['comments', 'likes'])
```

Each counted relation adds a `relation_count` attribute to every model in the result:

```json
{
    "id": 1,
    "title": "Getting Started",
    "comments_count": 42,
    "likes_count": 108
}
```

The client cannot request counts for relations not in `allowedCounts`. Unrecognised names are silently dropped.

Note: counting and loading are independent. A client can request `?count=comments` (to know how many comments exist) without also requesting `?include=comments` (which would load all the comment rows). Use both together only when you need both the count and the actual data.

### `aggregates`

Whitelist relation aggregates the client may request. Each maps a relation name to an array of allowed columns.

```php
Merchant::resourceQuery()
    ->allowSum(['products' => ['price'], 'orders' => ['total']])
    ->allowAvg(['products' => ['price'], 'reviews' => ['rating']])
    ->allowMin(['products' => ['price']])
    ->allowMax(['products' => ['price']])
    ->paginateAndTransform();
```

The client requests them via dot-notation in the query string:

```
?sum=products.price,orders.total
&avg=products.price,reviews.rating
&min=products.price
&max=products.price
```

Only `relation.column` pairs listed in the allowlist are executed. Wrong columns or disallowed relations are silently ignored. The resulting attributes (`products_sum_price`, `products_avg_rating`, etc.) appear on each model in the result and flow through the transformer naturally.

---

### `allowFields()`

Declares which root-model fields the client may request. **Only relevant when the model has a Transformer defined.** It controls which fields the transformer outputs — it does NOT limit the SQL query or affect `paginate()`, `all()`, or `one()` when no transformer is involved.

```php
Post::resourceQuery()
    ->allowFields(['title', 'excerpt', 'published_at', 'view_count'])
    ->paginateAndTransform();   // transformer respects the field list
```

**Simple form** — applies to the root model:

```
?fields=title,excerpt,published_at
```

**Bracketed form** — use the model's table name to address the root model, and any relation name to address a relation:

```
?fields[posts]=title,excerpt
?fields[author]=name,avatar_url
```

Only root-model fields are validated against `allowedFields`. Relation fields are validated only against whether the relation itself is in `allowedIncludes` — the specific field names within a relation are not restricted. This is intentional: the transformer on each related model controls what its own data looks like.

> Do not use `allowFields()` unless you are using `paginateAndTransform()`, `all()->transform()`, or calling `transformOptions()` manually to pass to a transformer.

---

## Executing the Query

### `paginate()`

Executes the query and returns a `Pagination` object. The current page is read from `?page` in the request. The per-page count is read from `?per_page`, falling back to `?limit`, then to the server default of 15 (customisable via `perPage()`). Capped at `maxPerPage` (default 100) to prevent clients from requesting arbitrarily large result sets.

```php
$pagination = $rq->paginate();          // reads ?per_page from request, capped at 100
```

```php
Post::resourceQuery()
    ->perPage(20)         // when client sends no ?per_page, use 20 instead of 15
    ->maxPerPage(50)      // a client sending ?per_page=500 will get 50 results
    ->paginate();
```

### `all()`

Executes without pagination and returns a full `Collection`.

```php
$posts = Post::resourceQuery()
    ->allowFilters(['status'])
    ->allowSorts(['published_at'])
    ->all();
```

Useful for small, bounded result sets (e.g., lookup lists) where pagination is unnecessary.

### `one()`

Executes and returns a single matching model or `null`.

```php
$post = Post::resourceQuery()
    ->allowFilters(['slug'])
    ->one();
```

### `getBuilder()`

Returns the fully configured `Builder` without executing. Use this when you need to add constraints beyond what `Resource Query` supports before executing:

```php
$builder = Post::resourceQuery()
    ->allowFilters(['status'])
    ->allowSorts(['published_at'])
    ->getBuilder();

// Add your own constraints, then execute
$pagination = $builder
    ->where('author_id', auth()->id())
    ->paginate();
```

---

## Shaping the Response

**Transformers are entirely optional.** `Resource Query` builds and executes the ORM query regardless of whether you have `Transformer` classes defined. `paginate()`, `all()`, `one()`, and `getBuilder()` have no transformer dependency at all.

If you are not using transformers, just serialise the result directly:

```php
$data = Article::resourceQuery()
    ->allowFilters(['status'])
    ->paginate();

return response()->json($data);   // no transformer involved
```

When you do have a `Transformer` defined on the model, use `paginateAndTransform()` to execute and transform in one call. The client controls which fields and relations appear through `?fields` and `?include`:

```php
$data = Post::resourceQuery()
    ->allowIncludes(['author', 'tags'])
    ->allowFields(['title', 'excerpt', 'published_at'])
    ->paginateAndTransform();

return response()->json($data);
```

For non-paginated collections, chain `transform()` after `all()`:

```php
$data = Post::resourceQuery()
    ->allowIncludes(['author'])
    ->allowFields(['title', 'excerpt'])
    ->all()
    ->transform();

return response()->json($data);
```

When no `?fields` or `?include` is present in the request, the transformer uses its full default output — the same result as calling `transform()` with no arguments.

---

## A Complete Controller Example

```php
<?php

namespace App\Http\Controllers\Api;

use App\Models\Post;

class PostController
{
    public function index()
    {
        $data = Post::resourceQuery()
            ->allowFilters(['status', 'category', 'search'])
            ->allowSorts(['title', 'published_at', 'view_count'])
            ->allowIncludes(['author', 'tags', 'comments'])
            ->allowFields(['title', 'excerpt', 'published_at', 'view_count'])
            ->defaultSort('-published_at')
            ->maxPerPage(50)
            ->paginateAndTransform();

        return response()->json($data);
    }
}
```

Sample response for `?filter[status]=published&include=author&fields=title,published_at&page=1&per_page=5`:

```json
{
    "data": [
        {
            "title": "Getting Started with Lightpack",
            "published_at": "2024-03-15",
            "author": {
                "name": "Jane Doe",
                "email": "jane@example.com"
            }
        }
    ],
    "meta": {
        "current_page": 1,
        "per_page": 5,
        "total": 42,
        "total_pages": 9
    },
    "links": {
        "first": "/api/posts?page=1",
        "last": "/api/posts?page=9",
        "prev": null,
        "next": "/api/posts?page=2"
    }
}
```

---

## Quick Reference

```php
ModelClass::resourceQuery()

    // Allowlists — define what the client may control
    ->allowFilters(['key', ...])        // maps to scopeKey() methods
    ->allowSorts(['column', ...])       // validates sort column names
    ->allowIncludes(['relation', ...])  // validates ?include values
    ->allowCounts(['relation', ...])    // validates ?count values
    ->allowSum(['relation' => ['col']])     // validates ?sum=relation.col
    ->allowAvg(['relation' => ['col']])     // validates ?avg=relation.col
    ->allowMin(['relation' => ['col']])     // validates ?min=relation.col
    ->allowMax(['relation' => ['col']])     // validates ?max=relation.col
    ->allowFields(['field', ...])       // transformer only — ignored without a Transformer

    // Server-side defaults
    ->defaultSort('-created_at')        // applied when ?sort is absent
    ->defaultIncludes(['relation'])     // always eager loaded
    ->perPage(20)                       // default when ?per_page is absent (default: 15)
    ->maxPerPage(50)                    // caps ?per_page (default: 100)

    // Execution
    ->paginate()                        // returns Pagination
    ->paginateAndTransform()            // returns array (transformed pagination data)
    ->all()                             // returns Collection
    ->one()                           // returns Model|null
    ->getBuilder()                      // returns Builder for further chaining
```

---
