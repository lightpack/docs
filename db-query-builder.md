# Query Builder

While you can definitely write raw SQL queries, Lightpack does come with a sleek query builder that helps you build SQL queries programatically.

It also helps you protect against SQL injection attacks by properly binding query parameters.

## Getting Started

First you need to create a query builder object passing it the name of the database **table** you want to query against.

For example, this will create a query builder object for `products` table. 

```php
<?php

use Lightpack\Database\Query\Query;

$products = new Query('products');
```

Or you can simply call the <code>table()</code> method on `app('db')` function.

```php
$products = app('db')->table('products');
```

Now you can start building and executing queries as documented below.

<p class="tip">The constructor for <code>Query</code> class optionally takes a database connection as its second argument. If you do not provide one, it will fallback to default database connection connection configured in services <code>app('db')</code></p>

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
$products->column('name');
```

## Select

You can specify table columns you need.

```php
// SELECT id, name FROM products
$products->select('id', 'name')->all();
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
// SELECT * FROM products WHERE id > ?
$products->where('id', '>', 2)->all();

// SELECT * FROM products WHERE id > ? AND color = ?
$products->where('id', '>', 2)->where('color', '=', '#000')->all();

// SELECT * FROM products WHERE id > ? AND color = ?
$products->where('id', '>', 2)->andWhere('color', '=', '#000')->all();

// SELECT * FROM products WHERE id > ? AND color = ? OR color = ?
$products->where('id', '>', 2)->andWhere('color', '=', '#000')->orWhere('color', '=', '#FFF')->all();

// SELECT * FROM products WHERE id IN ?, ?, ?
$products->whereIn('id', [23, 24, 25])->all();

// SELECT * FROM products WHERE id IN ?, ?, ? OR color IN ?, ?
$products->whereIn('id', [23, 24, 25])->orWhereIn('color', ['#000', '#FFF'])->all();

// SELECT * FROM products WHERE id NOT IN ?, ?, ?
$products->whereNotIn('id', [23, 24, 25])->all();

// SELECT * FROM products WHERE id NOT IN ?, ?, ? OR color NOT IN ?, ?
$products->whereNotIn('id', [23, 24, 25])->orWhereNotIn('color', ['#000', '#FFF'])->all();

// SELECT * FROM products WHERE owner IS NULL
$products->whereNull('owner')->all();

// SELECT * FROM products WHERE owner IS NOT NULL
$products->whereNotNull('owner')->all();

// SELECT * FROM products WHERE owner IS NULL AND weight IS NULL
$products->whereNull('owner')->andWhereNull('weight')->all();

// SELECT * FROM products WHERE owner IS NULL OR weight IS NULL
$products->whereNull('owner')->orWhereNull('weight')->all();

// SELECT * FROM products WHERE owner IS NULL OR weight IS NOT NULL
$products->whereNull('owner')->orWhereNotNull('weight')->all();
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

## Group By

You can group rows together.

```php
// SELECT id, name FROM products GROUP BY color, size
$products->select('id', 'name')->groupBy(['color', 'size'])->all();
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
// SELECT * FROM products LIMIT 10 OFFSET 2
$products->paginate(10)->all();
```

By default it will try to look for `page` query parameter from the URL string. But, you can also pass the current page value manually as second parameter.

```php
// SELECT * FROM products LIMIT 10 OFFSET 2
$products->paginate(10, 3)->all();
```

## Count

This methods returns the total number of rows in the table.

```php
// SELECT count(*) AS num FROM products
$products->count();

// SELECT count(* AS num FROM products WHERE price > 200
$products->where('price', '>', 200)->count();
```

## Joins

You can also join multiple tables.

```php
// SELECT * FROM products INNER JOIN options ON products.id = options.product_id
$products->join('options', 'products.id', 'options.product_id')->all();

// SELECT * FROM products LEFT JOIN options ON products.id = options.product_id
$products->leftJoin('options', 'options.product_id', 'products.id')->all();

// SELECT * FROM products RIGHT JOIN options ON products.id = options.product_id
$products->rightJoin('options', 'products.id', 'options.product_id')->all();

// SELECT products.*, options.name AS oname FROM products INNER JOIN options ON products.id = options.product_id
$products->select('products.*', 'options.name AS oname')->join('options', 'products.id', 'options.product_id')->all();
```

## Insert

Use `insert()` method to insert a new record.

```php
// INSERT INTO products (name, color) VALUES (?, ?)
$products->insert([
    'name' => 'Product 4',
    'color' => '#CCC',
]);
```

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

# toSql

To inspect the generated `SQL` query as string, use `toSql()` method:

```php
$products->toSql(); // SELECT * FROM PRODUCTS
```

Note that when you call `toSql()`, you cannot use methods that execute the query. For example, this is wrong to do:

```php
$products->all()->toSql(); // Error
$products->one()->toSql(); // Error
$products->where('id', '=', 23)->delete()->toSql(); // Error
```

This is because those methods actually execute the `SQL` query. So calling `toSql()` will result in error.