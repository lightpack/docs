# Collections in Lightpack ORM

Collections are a powerful abstraction in Lightpack ORM for working with groups of models. Whenever you fetch multiple records from the database—such as via `all()`, `where()`, or relationship queries—you receive a `Collection` object. This class provides a rich, expressive API for transforming, filtering, mapping, and interacting with sets of models in a clean, object-oriented way.

## What is a Collection?

A `Collection` is an object that wraps an array of models and provides utility methods for iteration, searching, transformation, and batch operations. It implements PHP interfaces like `IteratorAggregate`, `Countable`, `ArrayAccess`, and `JsonSerializable`, so it behaves like an array but with much more power.

### Example: Getting a Collection

```php
// Collection of User models
$users = User::query()->where('status', 'active')->all();
```

You can loop over a collection just like an array:

```php
foreach ($users as $user) {
    echo $user->email;
}
```

## Core Collection Methods

### ids()
Get all primary keys from the collection.
```php
$userIds = $users->ids(); // [1, 2, 3, ...]
```

### isEmpty()
Check if the collection is empty.
```php
if ($users->isEmpty()) { ... }
```

### isNotEmpty()
Check if the collection is not empty.
```php
if ($users->isNotEmpty()) { ... }
```

### find()
Find a model by its primary key. Returns `null` if not found, or a default value if provided.
```php
$user = $users->find(5);
$user = $users->find(99, new User()); // Returns default if not found
```

### first()
Get the first model in the collection, or the first that matches conditions.
```php
$firstUser = $users->first();
$admin = $users->first(['role' => 'admin']);
```

### last()
Get the last model in the collection. Returns `null` if the collection is empty.
```php
$latestUser = $users->last();
```

### column()
Get an array of values for a given column from all models. Falsy values (`null`, `''`, `false`, `0`) are skipped.
```php
$emails = $users->column('email');

// Only non-empty status values are returned
$statuses = $users->column('status'); // ['active', 'inactive']
```

### filter()
Return a new `Collection` with only items matching the callback.
```php
$activeUsers = $users->filter(fn($u) => $u->active);
// Returns a new Collection instance
```

### map()
Transform each item in the collection, returning a new `Collection`. The callback can return any type, not just models.
```php
$names = $users->map(fn($u) => strtoupper($u->name));
// Returns Collection of strings

$transformed = $users->map(fn($u) => ['id' => $u->id, 'name' => $u->name]);
// Returns Collection of arrays
```

### asMap()
Return an associative array keyed by a property.
```php
$userMap = $users->asMap('email');
// $userMap['foo@bar.com'] => User model
```

### any()
Check if any model in the collection has a given attribute.
```php
if ($users->any('last_login')) { ... }
```

### exclude()
Return a new `Collection` excluding models with the given primary key(s). Accepts a single key or array of keys.
```php
$withoutAdmins = $users->exclude([1, 2]);
$withoutOne = $users->exclude(5); // Single key
// Returns a new Collection instance
```

### each()
Run a callback on each item in the collection. Returns `$this` for chaining.
```php
$users->each(function($user) {
    $user->sendWelcomeEmail();
});

// Can be chained
$users->each(fn($u) => $u->activate())
      ->load('profile');
```

### toArray()
Convert the collection to an array of arrays (each model as array).
```php
$array = $users->toArray();
```

### load()
Eager load one or more relationships for existing model instances.
```php
$users->load('profile', 'roles');
```

### loadCount()
Eager load counts of related models.
```php
$users->loadCount('posts');
foreach ($users as $user) {
    echo $user->posts_count;
}
```

### loadSum() / loadAvg() / loadMin() / loadMax()
Eager load aggregate values from a related model's column. The attribute is attached using the convention `{relation}_{function}_{column}`.
```php
$users->loadSum('orders', 'amount');
$users->loadAvg('orders', 'amount');
$users->loadMin('orders', 'amount');
$users->loadMax('orders', 'amount');

foreach ($users as $user) {
    echo $user->orders_sum_amount;
    echo $user->orders_avg_amount;
}
```

### sort()
Return a new `Collection` sorted by a column. Direction defaults to `'asc'`. Does not mutate the original collection.

This is particularly useful when you need to sort by an attribute that was attached via eager loading (e.g. `withCount`, `withSum`).
```php
// alphabetical sort
$teams = Team::query()
    ->withCount('deals')
    ->all()
    ->sort('name');

// ranked by deals count
$teams = Team::query()
    ->withCount('deals')
    ->all()
    ->sort('deals_count', 'desc');
```

### loadMorphs()
Efficiently eager load polymorphic parents for a collection of models with a `morphTo` relation. Optionally specify the attribute name for the parent (default: `'parent'`).
```php
$comments->loadMorphs([
    Post::class,
    Video::class,
    Photo::class,
]);

// Access the parent
foreach ($comments as $comment) {
    echo $comment->parent->title; // Post, Video, or Photo instance
}

// Custom parent key
$comments->loadMorphs([Post::class, Video::class], 'commentable');
echo $comment->commentable->title;
```

## Transforming Collections

Collections support transforming all models using the model's transformer. This is especially useful for APIs:

```php
$payload = $users->transform([
    'fields' => [
        'self' => ['id', 'name', 'email'],
        'profile' => ['bio', 'avatar']
    ],
    'includes' => ['profile'],
    'context' => 'api' // Optional: for multi-context transformers
]);
```

**Returns:** An array of transformed data (not a Collection).

> **See also:** For advanced transformation options, custom field selection, relation includes, and best practices for API responses, refer to the [Transformers documentation](/docs/v0.x/transformers).

## Array-like and JSON Behavior

- **ArrayAccess:** You can use array syntax: `$users[0]`, `isset($users[3])`, etc.
- **Countable:** Use `count($users)` to get the number of items.
- **JsonSerializable:** Collections can be safely passed to `json_encode()`.

---

## Best Practices & Tips

- **Chain methods** for expressive code:
  ```php
  $emails = $users->filter(fn($u) => $u->active)
                  ->map(fn($u) => $u->email);
  // Returns Collection, call toArray() if you need a plain array
  ```
- **Avoid N+1 queries**: Use `load()` or `loadMorphs()` before accessing related models in a loop.
- **Use asMap()** for fast lookups by a property value.
- **Use transform()** for API responses, respecting field selection and includes.
- **Prefer collection methods** over manual array manipulation for clarity and consistency.

## Example: Putting It All Together

```php
$users = User::query()->with('profile')->all();

// Eager load relations and aggregates
$users->load('posts')->loadCount('posts');

// Filter to users who have at least one post
$active = $users->filter(fn($u) => $u->posts->isNotEmpty());
$emails = $active->column('email');

// Sort by post count (SQL ordering not possible on eager-loaded counts)
$ranked = $users->sort('posts_count', 'desc');

// Transform for API
return $ranked->transform([
    'fields' => ['id', 'name', 'email'],
    'includes' => ['profile', 'posts'],
]);
```

---