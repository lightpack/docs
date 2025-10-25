# Query Builder

While you can definitely write raw SQL queries, **Lightpack** does come with a fluent query builder that helps you build SQL queries programatically.

It also helps you protect against SQL injection attacks by properly binding query parameters.

## Getting Started

Call the `table()` method to get an instance of the query builder for the database connection.

For example, this will create a query builder object for `products` table. 

```php
$products = db()->table('products');
```

Now you can start building and executing queries as documented below.

## Fetch all

Call the <code>all()</code> method to retrieve all the rows in a table.

```php
// SELECT * FROM products
$products->all();
```
## Fetch one

To retrieve only a single record, call <code>one()</code> method instead.</p>

```php
// SELECT * FROM products LIMIT 1
$products->one();
```

## Fetch column

To retrieve a specific column value from a record:

```php
// SELECT name FROM products LIMIT 1
$products->column('name');
```

## Select

You can specify table columns you need.

```php
// SELECT id, name FROM products
$products->select('id', 'name')->all();
```

```php
// SELECT id AS product_id, name FROM products
$products->select('id AS product_id', 'name')->all();
```


```php
// SELECT count(*) as total_products FROM products
$products->select('count(*) AS total_products')->all();
```

## Alias

You can alias a table name using `alias()` method.

```php
// SELECT * FROM products AS p
$products->alias('p')->all();
```

## Distinct

You can select distinct rows too.

```php
// SELECT DISTINCT name FROM products
$products->select('name')->distinct()->all();
```

## Where

You can narrow result set using where clauses.

```php
// SELECT * FROM products WHERE id = ?
$products->where('id', '=', 2)->all();

// SELECT * FROM products WHERE id > ?
$products->where('id', '>', 2)->all();

// SELECT * FROM products WHERE id > ? AND color = ?
$products->where('id', '>', 2)->where('color', '=', '#000')->all();

// SELECT * FROM products WHERE id > ? OR color = ?
$products->where('id', '>', 2)->orWhere('color', '=', '#000')->all();
```

### Where in

> **Note:** The SQL generated uses parentheses: `IN (?, ?, ?)`. If you pass an empty array to `whereIn`, the condition will always be false. If you pass an empty array to `whereNotIn`, the condition will always be true. You can also pass a closure for subqueries.

```php
// SELECT * FROM products WHERE id IN (?, ?, ?)
$products->whereIn('id', [23, 24, 25])->all();

// SELECT * FROM products WHERE id IN (?, ?, ?) OR color IN (?, ?)
$products->whereIn('id', [23, 24, 25])->orWhereIn('color', ['#000', '#FFF'])->all();

// SELECT * FROM products WHERE id NOT IN (?, ?, ?)
$products->whereNotIn('id', [23, 24, 25])->all();

// SELECT * FROM products WHERE id NOT IN (?, ?, ?) OR color NOT IN (?, ?)
$products->whereNotIn('id', [23, 24, 25])->orWhereNotIn('color', ['#000', '#FFF'])->all();

// Subquery support:
$products->whereIn('size', function($q) {
    $q->from('sizes')->select('id')->where('size', '=', 'XL');
})->all();
```

### Where null

```php
// SELECT * FROM products WHERE owner IS NULL
$products->whereNull('owner')->all();

// SELECT * FROM products WHERE owner IS NOT NULL
$products->whereNotNull('owner')->all();

// SELECT * FROM products WHERE owner IS NULL AND weight IS NULL
$products->whereNull('owner')->whereNull('weight')->all();

// SELECT * FROM products WHERE owner IS NULL OR weight IS NULL
$products->whereNull('owner')->orWhereNull('weight')->all();

// SELECT * FROM products WHERE owner IS NULL OR weight IS NOT NULL
$products->whereNull('owner')->orWhereNotNull('weight')->all();
```

### Where between

> **Note:** You must provide exactly two values for the between clause, e.g. `[min, max]`.

```php
// SELECT * FROM products WHERE price BETWEEN ? AND ?
$products->whereBetween('price', [10, 20]);
```

```php
// SELECT * FROM products WHERE price NOT BETWEEN ? AND ?
$products->whereNotBetween('price', [10, 20]);
```

```php
// SELECT * FROM products WHERE price BETWEEN ? AND ? OR size BETWEEN ? AND ?
$products->whereBetween('price', [10, 20])->orWhereBetween('size', ['M', 'L']);
```

```php
// SELECT * FROM products WHERE price NOT BETWEEN ? AND ? OR size NOT BETWEEN ? AND ?
$products->whereNotBetween('price', [10, 20])->orWhereNotBetween('size', ['M', 'L']);
```

### Logical Grouping

You can group `where` conditions logically by passing a callback. This callback will recieve an instance of query builder.

```php
// SELECT * FROM products WHERE (color = ? OR size = ?)
$products->where(function($q) {
    $q->where('color', '=', '#000')->orWhere('size', '=', 'XL');
})->all();
```

```php
// SELECT * FROM products WHERE id = ? AND (color = ? OR color = ?)
$products->where('id', '=', 1)->where(function($q) {
    $q->where('color', '=', '#000')->orWhere('color', '=', '#FFF');
})->all();
```

### Subqueries

You can specify subqueries as callback functions in `where` clauses.

```php
// SELECT * FROM products WHERE size IN (SELECT id FROM sizes WHERE size = ?)
$products->whereIn('size', function($q) {
    $q->from('sizes')->select('id')->where('size', '=', 'XL');
})->all();
```

### Where exists

To specify `WHERE EXISTS` subquery, use `whereExists()` method.

```php
// SELECT * FROM products WHERE EXISTS (SELECT id FROM sizes WHERE size = ?)';
$products->whereExists(function($q) {
    $q->from('sizes')->select('id')->where('size', '=', 'XL');
});
```

To specify `WHERE NOT EXISTS` subquery, use `whereNotExists()` method.

```php
// SELECT * FROM products WHERE NOT EXISTS (SELECT id FROM sizes WHERE size = ?)';
$products->whereNotExists(function($q) {
    $q->from('sizes')->select('id')->where('size', '=', 'XL');
});
```

### Raw queries

Sometimes it's handy to write complex `where` clauses using **raw** query strings. For such cases, use `whereRaw()` and `orWhereRaw()` methods.

```php
// SELECT * FROM products WHERE color = '#000' AND size = 'XL';
$products->whereRaw("color = '#000' AND size = 'XL'");
```

To protect **raw** where queries against SQL injection attacks, you can pass an array of parameters as the second argument.

```php
// SELECT * FROM products WHERE color = ? AND size = ?';
$products->whereRaw('color = ? AND size = ?', ['#000', 'XL']);
```

```php
// SELECT * FROM products WHERE color = ? OR status = 'active'";
$products->where('color', '=', '#000')->orWhereRaw("status = 'active'");
```

## Order By

You can specify order of result set.

```php
// SELECT id, name FROM products ORDER BY id ASC
$products->select('id', 'name')->orderBy('id')->all();

// SELECT id, name FROM products ORDER BY id DESC
$products->select('id', 'name')->orderBy('id', 'DESC')->all();

// SELECT id, name FROM products ORDER BY name DESC, id DESC
$products->select('id', 'name')->orderBy('name', 'DESC')->orderBy('id', 'DESC')->all();
```

> **Tip:** You can also use `asc()` and `desc()` shortcuts. See [Ordering Shortcuts](#ordering-shortcuts) in the Date & Time Filters section.

## Group By

You can group rows together.

```php
// SELECT * FROM products GROUP BY color
$products->groupBy('color')->all();
```

```php
// SELECT * FROM products GROUP BY color, size
$products->groupBy('color', 'size')->all();
```

## Limit

```php
// SELECT * FROM products LIMIT 10
$products->limit(10)->all();
```

## Offset

```php
// SELECT * FROM products LIMIT 10 OFFSET 2
$products->limit(10)->offset(2)->all();
```

## Paginate

Use `paginate()` method to fetch the records page wise. 

So if the request URL is `http://domain.com?page=3`,

```php
// SELECT * FROM products LIMIT 10 OFFSET 20
$rows = $products->paginate(10);
```

By default it will try to look for `page` query parameter from the URL string. But, you can also pass the current page value manually as second parameter. For example, following query will paginate the result with `10` results for `3rd` page.

```php
// SELECT * FROM products LIMIT 10 OFFSET 20
$rows = $products->paginate(10, 3);
```

Now you can iterate the result as an array.

```php
foreach($rows as $product) {
    $product->name;
    $product->color;
}
```

## Count

This methods returns the total number of rows in the table.

```php
// SELECT count(*) AS total FROM products
$products->count();

// SELECT count(*) AS total FROM products WHERE price > 200
$products->where('price', '>', 200)->count();
```

## Exists / Not Exists

Check if any rows exist matching the query conditions:

```php
// Check if expensive products exist
if ($products->where('price', '>', 1000)->exists()) {
    // Show premium category
}

// Check if out-of-stock products exist
if ($products->where('stock', '=', 0)->notExists()) {
    // All products are in stock
}

// Check if user has any orders
$orders = db()->table('orders');
if ($orders->where('user_id', '=', $userId)->exists()) {
    // Show order history
}
```

## Joins

You can also join multiple tables.

```php
// SELECT * FROM products INNER JOIN options ON products.id = options.product_id
$products->join('options', 'products.id', 'options.product_id')->all();

// SELECT * FROM products LEFT JOIN options ON products.id = options.product_id
$products->leftJoin('options', 'products.id', 'options.product_id')->all();

// SELECT * FROM products RIGHT JOIN options ON products.id = options.product_id
$products->rightJoin('options', 'products.id', 'options.product_id')->all();

// SELECT products.*, options.name AS oname FROM products INNER JOIN options ON products.id = options.product_id
$products->select('products.*', 'options.name AS oname')->join('options', 'products.id', 'options.product_id')->all();
```

## Insert

Use `insert()` method to insert a new record. The method returns the result of the query execution (not the inserted ID).

```php
// INSERT INTO products (name, color) VALUES (?, ?)
$products->insert([
    'name' => 'Product 4',
    'color' => '#CCC',
]);
```

> **Note:** To get the last inserted auto-incremented ID, use the `lastInsertId()` method after insert.

### Get Last Inserted ID

After performing an insert, you can retrieve the last auto-incremented primary key value using the `lastInsertId()` method. This is useful for working with records that use auto-incrementing IDs.

```php
$products->insert([
    'name' => 'Product 4',
    'color' => '#CCC',
]);

$lastId = $products->lastInsertId(); // Gets the last inserted ID
```

### Bulk Insert

To insert multiple records, simply pass an array of arrays to the `insert()` method:

```php
$products->insert([
    ['name' => 'Product 1', 'color' => '#CCC'],
    ['name' => 'Product 2', 'color' => '#DDD'],
    ['name' => 'Product 3', 'color' => '#EEE'],
]);
```

> **Note:** There is no `bulkInsert()` method. Use `insert()` for both single and multiple records.

## Update

Use `update()` method to modify an existing record.

```php
// UPDATE products SET name = ?, color = ? WHERE id = 23
$products->where('id', '=', 23)->update([
    'name' => 'Product 4',
    'color' => '#CCC',
]);
```

## Delete

Use `delete()` method to delete an existing record.

```php
// DELETE FROM products WHERE id = 23
$products->where('id', '=', 23)->delete();
```


## Insert Ignore

Insert a record, ignoring errors (like duplicate keys):

```php
$products->insertIgnore([
    'name' => 'Product 4',
    'color' => '#CCC',
]);
```

## Upsert (Insert or Update)

Insert or update records using MySQL's ON DUPLICATE KEY UPDATE:

```php
$products->upsert(['id' => 1, 'name' => 'New Name']);
```

You can specify which columns to update:

```php
$products->upsert(['id' => 1, 'name' => 'New Name'], ['name']);
```

---

## toSql

To inspect the generated `SQL` query as string, use `toSql()` method:

```php
$products->toSql(); // SELECT * FROM products
```

Note that when you call `toSql()`, you cannot use methods that execute the query. For example, this is wrong to do:

```php
$products->all()->toSql(); // Error
$products->one()->toSql(); // Error
$products->where('id', '=', 23)->delete()->toSql(); // Error
```

This is because those methods actually execute the `SQL` query. So calling `toSql()` will result in error.

---

## Raw Select Expressions

If you need to select expressions or use SQL functions, use `selectRaw()`:

```php
// SELECT id, SUM(score) AS total FROM products
$products->select('id')->selectRaw('SUM(score) AS total')->groupBy('id')->all();
```

You can pass bindings as the second argument for safety:

```php
$products->selectRaw('SUM(score) > ? AS high', [100]);
```

---

## HAVING Clauses

You can filter groups after aggregation using `having()`, `orHaving()`, `havingRaw()`, and `orHavingRaw()`:

```php
// SELECT category, COUNT(*) FROM products GROUP BY category HAVING COUNT(*) > 5
$products->select('category')->selectRaw('COUNT(*)')->groupBy('category')->having('COUNT(*)', '>', 5)->all();
```

For more complex conditions, use raw SQL:

```php
$products->havingRaw('SUM(score) > ?', [100]);
```

---

## Row Locking

You can lock rows for update using `forUpdate()`, or skip locked rows with `skipLocked()`:

```php
$products->where('stock', '>', 0)->forUpdate()->all();
$products->where('stock', '>', 0)->forUpdate()->skipLocked()->all();
```

---

## Full-Text Search

To perform full-text search on indexed columns:

```php
// WHERE MATCH(title, body) AGAINST ('foo bar' IN BOOLEAN MODE)
$products->search('foo bar', ['title', 'body'])->all();
```

---

## Boolean Shortcuts

You can quickly filter on boolean columns:

```php
$products->whereTrue('is_active')->all();
$products->whereFalse('is_deleted')->all();
$products->orWhereTrue('is_featured')->all();
$products->orWhereFalse('is_archived')->all();
```

---

## Conditional Query Building

Add conditions only if a value is present:

```php
$products->whereIf($userId, 'user_id', '=', $userId);
```

Or, run a callback if a condition is true:

```php
$products->when($isAdmin, function($q) {
    $q->where('is_admin', true);
});
```

---

## Increment/Decrement

Atomically increase or decrease a column value. **IMPORTANT:** A `where()` clause is **required** - the operation will throw a `RuntimeException` if no where clause is present.

```php
// Increment stock by 5 for product with id 1
$products->where('id', 1)->increment('stock', 5);

// Decrement stock by 2 for product with id 1
$products->where('id', 1)->decrement('stock', 2);

// Default increment/decrement is 1
$products->where('id', 1)->increment('views'); // +1
$products->where('id', 1)->decrement('stock'); // -1
```

> **Warning:** Calling `increment()` or `decrement()` without a `where()` clause will throw: `RuntimeException: Increment/Decrement operations require a where clause`

---

## Chunked Processing

Process large datasets in batches:

```php
$products->chunk(100, function($chunk) {
    foreach ($chunk as $product) {
        // Process each product
    }
});
```

---

## Lazy Loading with Cursor

For memory-efficient processing of large result sets, use `cursor()` which yields one row at a time instead of loading all results into memory:

```php
foreach ($products->where('status', 'active')->cursor() as $product) {
    // Process one product at a time
    // Only one row in memory at any time
}
```

### Cursor vs Chunk vs All

```php
// ❌ all() - Loads ALL rows into memory
$products = $products->all();
foreach ($products as $product) {
    // Process
}

// ✅ chunk() - Loads N rows at a time (good for batch operations)
$products->chunk(1000, function($batch) {
    // Memory: 1000 rows at a time
    foreach ($batch as $product) {
        // Process batch
    }
});

// ✅ cursor() - Streams one row at a time (best for memory efficiency)
foreach ($products->cursor() as $product) {
    // Memory: 1 row at a time
    // Perfect for exports, migrations, large reports
}
```

### When to Use Cursor

**Use `cursor()` for:**
- ✅ Exporting large datasets (CSV, Excel)
- ✅ Data migrations and transformations
- ✅ Processing millions of records
- ✅ Generating large reports
- ✅ ETL operations
- ✅ Any operation where memory is a concern

**Use `chunk()` for:**
- ✅ Batch updates/deletes
- ✅ Sending bulk emails
- ✅ Rate-limited API calls
- ✅ Operations that benefit from batching

**Use `all()` for:**
- ✅ Small result sets (< 1000 rows)
- ✅ When you need the full collection
- ✅ When you need to count/filter in PHP

> **Note:** `cursor()` returns a PHP Generator, so you can only iterate through it once. If you need to iterate multiple times, call `cursor()` again.

---

## Aggregates

You can use the following methods for aggregate queries:

```php
$products->sum('price');
$products->avg('rating');
$products->min('created_at');
$products->max('updated_at');
$products->countBy('category'); // returns an array of objects with counts for each group
```

---

## Date & Time Filters

### Date Part Filtering

Filter records by specific parts of date/time columns:

```php
// Filter by date (ignores time)
$orders->whereDate('created_at', '2024-01-15')->all();
$orders->whereDate('created_at', '>=', '2024-01-01')->all();

// Filter by year
$reports->whereYear('created_at', 2024)->all();
$archives->whereYear('created_at', '<', 2020)->all();

// Filter by month (1-12 or month name)
$sales->whereMonth('created_at', 12)->all();  // December
$sales->whereMonth('created_at', 'dec')->all();  // Short name
$sales->whereMonth('created_at', 'december')->all();  // Full name
$sales->whereMonth('created_at', '>=', 6)->all();

// Filter by day of month (1-31)
$payroll->whereDay('pay_date', 15)->all();
$reminders->whereDay('due_date', '<=', 5)->all();

// Filter by time
$appointments->whereTime('scheduled_at', '>=', '09:00:00')->all();
$appointments->whereTime('scheduled_at', '<=', '17:00:00')->all();
```

> **Note:** Month names are case-insensitive and support both short (jan, feb, mar) and full names (january, february, march).

All date/time methods have OR variants:

```php
$records->whereYear('created_at', 2023)->orWhereYear('created_at', 2024)->all();
$logs->whereDate('created_at', today())->orWhereDate('created_at', yesterday())->all();
```

### Relative Date Filters

Filter records using relative time periods:

```php
// Today's records
$orders->today()->all();
$orders->today('order_date')->all();  // Custom column

// Yesterday's records
$logs->yesterday()->all();

// This week (Monday-Sunday)
$tasks->thisWeek()->all();

// Last week
$reports->lastWeek()->all();

// This month
$sales->thisMonth()->all();

// Last month
$invoices->lastMonth()->all();

// This year
$users->thisYear()->all();

// Last year
$analytics->lastYear()->all();
```

### Last N Periods

Filter records from the last N days, weeks, or months:

```php
// Last N days
$orders->lastDays(7)->all();   // Last 7 days
$orders->lastDays(30)->all();  // Last 30 days
$orders->lastDays(90)->all();  // Last 90 days

// Last N weeks
$reports->lastWeeks(4)->all();  // Last 4 weeks

// Last N months
$sales->lastMonths(3)->all();   // Last 3 months
$sales->lastMonths(6)->all();   // Last 6 months
$sales->lastMonths(12)->all();  // Last 12 months
```

### Age-Based Filtering

Filter records based on how old they are:

```php
// Records older than specified time
$tickets->olderThan(7, 'days')->all();
$tickets->olderThan(48, 'hours')->all();
$leads->olderThan(30, 'days')->all();
$cache->olderThan(1, 'hour')->all();

// Records newer than specified time
$activities->newerThan(24, 'hours')->all();
$posts->newerThan(7, 'days')->all();

// Supported units: 'minutes', 'hours', 'days', 'weeks', 'months', 'years'
```

### Before/After Shortcuts

Filter records before or after a specific date:

```php
// Before a specific date
$orders->before('2024-01-01')->all();
$orders->before('2024-01-01', 'shipped_at')->all();

// After a specific date
$orders->after('2024-01-01')->all();
$orders->after('2024-01-01', 'shipped_at')->all();
```

### Weekday/Weekend Filtering

Filter records by day of week:

```php
// Weekdays only (Monday-Friday)
$orders->weekdays()->all();

// Weekends only (Saturday-Sunday)
$orders->weekends()->all();
```

### Ordering Shortcuts

Quick methods for ascending/descending order:

```php
// Descending order (highest/newest first)
$products->desc()->all();              // ORDER BY id DESC
$products->desc('price')->all();       // ORDER BY price DESC
$posts->desc('created_at')->all();     // ORDER BY created_at DESC

// Ascending order (lowest/oldest first)
$products->asc()->all();               // ORDER BY id ASC
$products->asc('name')->all();         // ORDER BY name ASC
$posts->asc('created_at')->all();      // ORDER BY created_at ASC

// Multiple ordering
$products->desc('price')->asc('name')->all();
```

### Real-World Examples

```php
// Dashboard: Today's sales
$todaySales = db()->table('orders')->today()->sum('amount');

// Report: This month's signups
$signups = db()->table('users')->thisMonth()->count();

// Analytics: Last 30 days revenue
$revenue = db()->table('orders')
    ->lastDays(30)
    ->where('status', 'completed')
    ->sum('amount');

// CRM: Stale leads (no contact in 30 days)
$staleLeads = db()->table('leads')
    ->olderThan(30, 'days', 'last_contact_at')
    ->where('status', 'active')
    ->all();

// Support: Overdue tickets
$overdue = db()->table('tickets')
    ->where('status', 'open')
    ->olderThan(48, 'hours')
    ->desc('created_at')
    ->all();

// Weekend vs weekday sales
$weekendSales = db()->table('orders')->thisMonth()->weekends()->sum('amount');
$weekdaySales = db()->table('orders')->thisMonth()->weekdays()->sum('amount');
```

---