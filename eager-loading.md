# Eager Loading

Once again let us consider these three tables:

<table>
    <tr>
        <td class="token title important">products</td>
        <td>id</td>
        <td>title</td>
    </tr>
</table>
<table>
    <tr>
        <td class="token title important">options</td>
        <td>id</td>
        <td>product_id</td>
        <td>name</td>
        <td>color</td>
    </tr>
</table>
<table>
    <tr>
        <td class="token title important">seo</td>
        <td>id</td>
        <td>product_id</td>
        <td>meta_title</td>
        <td>meta_description</td>
    </tr>
</table>

The association between these entities is:

* A **product** has many **options**.
* A **product** has one **seo** details.

```php
class Product extends Model
{
    public function options()
    {
        return $this->hasMany(Option::class, 'product_id');
    }

    public function seo()
    {
        return $this->hasOne(Seo::class, 'product_id');
    }
}
```

Now suppose we have **20** **products** and correspondingly **20 seo** records. To fetch `products` along with their `seo` details, you can simply loop each product:

```php
$products = Product::query()->all();

foreach($products as $product) {
    echo $product->seo->meta_title;
}
```

 The problem with above approach is that it will fire one query for fetching all `products`
and **20** extra queries per product to fetch `seo` data. This will result in a total of `21 queries` which can be expensive.

In raw SQL terms, following queries will be executed

```sql
-- 1 query to fetch all products
SELECT * FROM products;

-- 20 queries (one per product) to fetch seo details
SELECT * FROM seo WHERE product_id = 1;
SELECT * FROM seo WHERE product_id = 2;
SELECT * FROM seo WHERE product_id = 3;
-- ...and so on, up to product_id = 20
SELECT * FROM seo WHERE product_id = 20;
```

> **This is known as the classic `N+1` queries problem**. 

For every **product**, the ORM issues an additional query to fetch its related **SEO** record—quickly adding up to many unnecessary database calls and poor performance.

**Eager loading** is a technique designed to solve this problem. Instead of fetching related data one record at a time, eager loading retrieves all the necessary related records in as few queries as possible—usually just one extra query, no matter how many products you have. This dramatically reduces database load and speeds up your application, especially when displaying lists of records with their associations.

In Lightpack ORM, eager loading is simple and explicit, empowering you to write efficient, high-performance code with minimal effort.

### Single-ended Associations

To eager load **single-ended** associations aka `1:1` mapping, use `with()` method:

```php
$products = Product::query()->with('seo')->all();
```

This will fetch all products and corresponding seo records with just two queries:

```sql
select * from products;

select * from seo where product_id in (1,2,3,4,5,...,N)
```

So you can loop each product and access its seo data:

```php
foreach($products as $product) {
    echo $product->seo->meta_title;
}
```

### One-to-many Associations

To eager load `1:N` associations, use the same `with` method passing it the associated method name as string:

```php
$products = Product::query()->with('options')->all();
```

### Loading multiple associations

To eager load multiple associations, pass associated method names in the `with` method:

```php
$products = Product::query()->with('seo', 'options')->all();
```

### Counting associations

To count the associated relations, use `withCount()` method:

```php
$categories = Category::query()->withCount('products')->all();
```

> **How do you access the count?**
>
> When you use `withCount()` or `loadCount()` in Lightpack, the ORM automatically adds a `_count` suffix to the relation property. For example, after calling `withCount('comments')`, you can access the count using `$post->comments_count`. This naming convention keeps your model properties clear and intention-revealing.

For example:

```php
$categories = Category::query()->withCount('products')->all();

foreach($categories as $category) {
    echo $category->products_count;
}
```

---

### Quick Reference: Eager Loading Methods

| Use When...                           | For fetching...     | Example Usage                                      |
|---------------------------------------|---------------------|----------------------------------------------------|
| `with()`                              | Related models      | `with('seo')`, `with('options')`                   |
| `withCount()`                         | Count of relations  | `withCount('products')`                            |
| `load()` (on collection)              | Related models      | `$products->load('seo')`                           |
| `loadCount()` (on collection)         | Count of relations  | `$products->loadCount('options')`                  |

- Use `with()` and `withCount()` when building your initial query.
- Use `load()` and `loadCount()` to eager load associations on an **existing collection**.

---

### Eager Loading Callbacks: Filtering Related Data

You can pass a callback to `with()`, `withCount()`, `load()`, or `loadCount()` to apply conditions to the eager loaded relationship. The callback receives the query builder for the related model, letting you add any filters you need.

```php
$projects = Project::query()->with(['tasks' => function($q) {
    $q->where('status', '=', 'pending');
}])->all();
```

Here, only tasks with status `pending` are eager loaded for each project.

You can also nest callbacks for deeper relations:

```php
$projects = Project::query()->with([
    'tasks' => function($q) {
        $q->where('status', '=', 'pending');
        $q->with(['comments' => function($q) {
            $q->where('status', '=', 'approved');
        }]);
    }
])->all();
```

---

### Chaining and Composing Eager Loading Methods

Lightpack ORM allows you to fluently chain eager loading methods with other query builder methods for expressive, composable queries. For example:

```php
$projects = Project::query()
    ->with('manager')
    ->withCount('tasks')
    ->where('status', '=', 'active')
    ->orderBy('created_at', 'desc')
    ->all();
```

This query fetches all active projects, eager loads the manager, counts the tasks, orders by creation date, and returns the results—all in a single, readable chain.

---

### Nested Eager Loading

Suppose we have table `projects` with many `tasks` and each task can have many `comments`. We can **eager load** `projects` with their `tasks` and `comments` together:

```php
$projects = Project::query()->with('tasks.comments')->all();
```

Note that such convinience can become a performance issue if there are too many `comments` for `tasks` for a given project.

### Conditional Eager Loading

You can restrict **eager loading** relations via `callback` functions. For example, to eager load all pending `tasks` for `projects`:

```php
$projects = Project::query()->with(['tasks' => function($q) {
    $q->where('status', '=', 'pending');
})->all();
```

You can also restrict **nested** eager loading. For example. to eager load `projects` with **pending** `tasks` and **approved** `comments`:

```php
$projects = Project::query()->with(['tasks' => function($q) {
    // Load tasks with pending status
    $q->where('status', '=', 'pending');

    // Load comments with approved status
    $q->with(['comments' => function($q) {
        $q->where('status', '=', 'approved');
    }]);
})->all();
```

**Note:** You can apply the same constraints on `withCount()` method too:

```php
$projects = Project::query()->withCount(['tasks' => function($q) {
    $q->where('status', '=', 'pending');
})->all();
```

### Deferred eager loading

Suppose that we have `100` records in products table. Eager loading `seo` and `options` for `100` products will be a huge performance miss.

In such cases, you might be interested in **paginating** `products` and then eager load associated relations.

```php
$products = Product::query()->paginate(10);
```

Once you have got the **products**, you can eager load its **associated** relations by calling `load()` and `loadCount()` methods on **products**.

```php
$products->load('seo');
$products->loadCount('options');
```

This will automatically populate `seo` data along with `options` count for each product in `$products` collection.

```php
foreach($products as $product) {
    $product->seo; 
    $product->options_count;
}
```

**Note:** You can also chain `load()` and `loadCount()` methods together. For example:

```php
$products = Product::query()->paginate(10);
$products->load('seo')->loadCount('options');
```

**Note:** All the capabilities that `with()` and `withCount()` methods have also applies to `load()` and `loadCount()` methods.

For example, you can pass **callbacks** to restrict eager loading:

```php
$products->load(['reviews' => function($q) {
    $q->where('status', '=', 'approved');
}]);
```

```php
$products->loadCount(['reviews' => function($q) {
    $q->where('status', '=', 'approved');
}]);
```

---

#### Eager Loading Polymorphic Parents with `loadMorphs`

When working with a collection of polymorphic models (e.g., a list of comments where each comment could belong to a different parent type), eager loading the parent models efficiently can be challenging. Lightpack ORM provides the `loadMorphs()` method to solve this elegantly.

##### Why use `loadMorphs()`?

If you have a collection of comments, each referencing a different parent type (Post, Video, etc.), calling `$comments->load('parent')` is not sufficient, because the ORM needs to know all possible parent types to perform efficient eager loading. `loadMorphs()` lets you specify the possible types so Lightpack can fetch all parents in as few queries as possible.

##### Example Usage

Suppose you fetch a set of comments:

```php
$comments = Comment::query()->where('user_id', '=', 42)->all();
```

You can eager load their parents like this:

```php
$comments->loadMorphs([
    Post::class,
    Video::class,
    Photo::class,
]);
```

Now, for each comment, `$comment->parent` will be the appropriate parent model instance, and all parents will have been loaded efficiently—no N+1 problem!

##### Best Practices & Notes
- Always specify all possible parent types for the morph relation.
- Use `loadMorphs` after fetching the collection (not on the query builder).
- This method is especially useful for API responses or UI screens showing polymorphic lists with their parents.
- If you omit a possible type, parents of that type will not be eager loaded and may trigger lazy loading (and N+1 queries) if accessed.

---

## Lazy Loading

If you access a relation property that hasn't been loaded yet, Lightpack ORM will transparently execute a new query to fetch it. This aspect is know as **lazy loading** relationships.

```php
$user = new User(1); // No relations loaded
$posts = $user->posts; // Triggers a query to fetch posts for this user
```

- The first time you access `$user->posts`, Lightpack issues a query and caches the result on the model instance.
- Subsequent accesses to `$user->posts` use the cached data—no additional queries are made.

### Tradeoffs and Risks

- **N+1 Query Problem:** If you loop over a collection and access a relation on each item, you can easily trigger many queries:
  
  ```php
  $users = User::query()->all();
  
  foreach ($users as $user) {
        $user->profile; // Triggers 1 profile query per user
  }
  ```

  This leads to poor performance and unpredictable database load.

- **Performance Unpredictability:** Lazy loading can make it hard to know how many queries your code will run, especially as your data grows.

- **Best Practice:** Always prefer eager loading (`with()`, `load()`) for predictable, efficient queries. Use lazy loading only if you are certain the relation will be accessed infrequently or in non-performance-critical code (e.g., admin dashboards, prototypes).

### When is Lazy Loading Acceptable?

- For small datasets, prototyping, or admin tools where performance is not critical.
- When you explicitly whitelist certain relations using `allowedLazyRelations` in strict mode.

---

## Strict Mode & Lazy Loading

Preventing **`N+1`** query issues and ensuring predictable performance is a core principle in Lightpack ORM. While eager loading is the primary tool for fetching related data efficiently, Lightpack also offers **strict mode** for even greater safety and explicitness.

> **Tip:** For maximum safety against N+1 issues, enable strict mode on your models. See the section below for details.

### What is Strict Mode?

Strict mode prevents accidental lazy loading of relations. In strict mode, if you try to access a relation that was not eager loaded (via `with`, `load`, etc.), Lightpack will throw an exception—unless that relation is explicitly whitelisted. This is crucial for API performance, large-scale applications, and when you want to guarantee predictable queries.

### How to Enable Strict Mode

Enable strict mode by setting the following property on your model:

```php
protected $strictMode = true;
```

Optionally, you can allow specific relations to be lazy loaded by whitelisting them:

```php
protected $allowedLazyRelations = ['profile', 'roles'];
```

### How It Works
- Accessing a relation that was not eager loaded or whitelisted throws an exception.
- Only eager loaded relations or whitelisted `allowedLazyRelations` can be accessed without error.
- This ensures all relation access is explicit and safe for large-scale or API use.

### Example: Strict Mode in Action

```php
class User extends Model
{
    protected $strictMode = true;
    protected $allowedLazyRelations = ['profile'];
}

// Eager loading
$user = User::query()->with('roles')->one(); // OK
$user->roles; // OK

// Not eager loaded and not whitelisted
$user = new User($id); // No eager load

// Throws exception
$user->roles;

// Whitelisted lazy relation
$user->profile; // OK
```

---