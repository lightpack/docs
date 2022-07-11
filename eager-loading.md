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

Suppose we have **20** **products** and correspondingly **20 seo** records. To display `products` along with their `seo` details, you can simply loop each product:

```php
$products = Product::query()->all();

foreach($products as $product) {
    echo $product->seo->meta_title;
}
```

 The problem with above approach is that it will fire one query for fetching all `products`
and **20** extra queries per product to fetch `seo` data. This will result in a total of `21 queries` which can be expensive.

This is called `N+1` queries problem. To resolve such issue, the concept of **eager loading** exists that helps resolve `N+1` issues. 

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