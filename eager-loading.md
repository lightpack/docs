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

**This is known as the classic `N+1` queries problem** 

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

This will find all categories along with their products count which you can access using `{relation_key}_count` property on individual object. For example:

```php
$categories = Category::query()->withCount('products')->all();

foreach($categories as $category) {
    echo $category->products_count;
}
```

You can also use `with()` and `withCount()` methods together. For example, this query finds all **projects** along with **manager** data and count of **tasks**.

```php
$projects = ProjectModel::query()->with('manager')->withCount('tasks')->all();
```

So you can loop each **projects** and access the **manager** data along with **task** count on each project:

```php
foreach($projects as $project) {
    $manager = $project->manager; // manager
    $tasksCount = $project->tasks_count; // tasks count
}
```

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
        $q->where('status', =, 'approved');
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

For example, you can pass provide **callbacks** to restric eager loading:

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