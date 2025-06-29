# Models

![ORM Model Overview](_media/orm/orm-model-overview.svg)

## Introduction to ORM Models

Lightpack ORM is an **Active Record** pattern implementation. Each model is a class directly corresponds to a single table in your database, and each instance of a model represents a single row within that table. The model not only holds data, but also encapsulates all the logic required to create, read, update, and delete (CRUD) records.

**Key Fundamentals of Active Record:**
- **Class-to-Table Mapping:** Each model class maps to a database table. For example, a `User` model maps to a `users` table.
- **Object-to-Row Mapping:** Each model instance represents a row in the corresponding table.
- **CRUD Operations:** Models provide methods to perform CRUD operations directly, such as `save()`, `find()`, `update()`, and `delete()`, without needing to write SQL manually.

The following sections will explore how to define models, establish relationships, and utilize the full capabilities of the ORM system.

---


Consider that you have a `products` table in your database.

```table
Table: products
-------------------------------------------------
id, name, size, color, status
-------------------------------------------------
```

Then you should define a `Product` model in <code>app/Models</code> folder.

## Defining Model

Fire this command to generate a model from your terminal inside your project root.

```terminal
php lucy create:model Product --table=products
```

This should have created `Product` model in `app/Models` folder.

```php
class Product extends Model
{
    protected $table = 'products';

    protected $primaryKey = 'id';

    protected $timestamps = false;
}
```

Defining your model in this manner gives you access to a number of utility
methods to deal with records in your `products` table.

## Performing CRUD

Once the model class is defined, performing **CRUD** operations per single database record becomes very easy. You do not need to manually wire-up raw SQL queries for:

- **(C)** creating a new record, 
- **(R)** reading an existing record, 
- **(U)** update an existing record, 
- **(D)** or delete an existing record.

### Create

Set properties on your new model and simply call the inherited method <code>insert()</code> to insert a new record.

```php
// Create the instance
$product = new Product;

// Set product properties
$product->name = 'ACME Shoes';
$product->size = 10;
$product->color = 'black'

// Create new product
$product->insert();
```
#### Last Insert ID

If your table's primary key is an **auto-incrementing** field, you can get the last insert id:

```php
$product->lastInsertId();
```

#### Manual Primary Key

If your table's primary key is a **non auto-incrementing** field, you must override the inherited model attribute `autoIncrements` to false.

```php
class Product extends Model
{
    protected $autoIncrements = false;
}
```

Now when calling `insert()` method, you must pass a unique primary key value for the new record to be created.

```php
// Create the instance
$product = new Product;

// Set product properties
$product->id = 'sku1000';
$product->name = 'ACME Shoes';
$product->size = 10;
$product->color = 'black'

// Create new product
$product->insert();
```


### Read

You can easily fetch a record by its **ID** when constructing the model. 

```php
$product = new Product(23);
```

Now you can access the column values as model properties.

```php
echo $product->title;
echo $product->size;
echo $product->color;
```

### Update

Use the `update()` method to update an existing record in the database table. You first need to instantiate the model using the primary key of the table.

```php
// Get an existing product having id: 23
$product = new Product(23);

// Set product properties to update
$product->name = 'ACME Footwear';
$product->size = 11;
$product->color = 'brown'

// Update the product
$product->update();
```

### Delete

Simply call the <code>delete()</code> passing it the id of the record to be deleted from database.

```php
(new Product)->delete(23);
```

If you already have an existing instance of model, you can call `delete()` method there too.

```php
$product = new Product(23);
$product->delete(); // passing id not required
```

## Timestamps

Consider **products** table:

```table
Table: products
-------------------------------------------------
id, name, price, created_at, updated_at
-------------------------------------------------
```

Setting `$timestamps` property to `true` automatically sets values for **created_at** and **updated_at** columns in your table. So
when you create a new product or update an existing product, you don't have to manually set values for these columns.

<p class="tip">In order for timestamps to work, the table must have <b>created_at</b> and <b>updated_at</b> columns both.</p>

## Refetch

Sometimes, the data in your existing model instance can become outdated—especially if changes are made to the database from somewhere else in your application. 

For an example, consider the scenario where the `timestamps` attribute in model class definition is set to **true**.

```php
class Product extends Model
{
    protected $timestamps = true;
}
```

In such case, performing an `insert()` or `update()` automatically sets **created_at** and **updated_at** columns. This happens as a side-effect of the framework's ORM implementation as convinience. 

```php
/**
 * created_at, updated_at column is set automatically
 */
$product->insert(); 
```

Now if you try this:

```php
echo $product->created_at; // null
echo $product->updated_at; // null
```

To ensure you are working with the latest data for the current record from the database, you can call `refetch()` method.

```php
$product = $product->refetch();
```

`$product` will contain the latest data. If the record was deleted or the primary key isn’t set, it will return `null`.

## Cloning a Model

There may be times when you want to create a new record in your database that’s almost identical to an existing one—without re-entering all the data. The `clone` method makes this easy: it creates a new model instance with the same attribute values as the original, but leaves out the primary key and timestamps, so you can safely save it as a new record.

This is especially useful for duplicating templates, copying products, or quickly creating similar entries.

**How it works:**  
- The new instance copies all attributes from the original, except for the primary key (`id`), `created_at`, and `updated_at` fields.
- You can also specify additional fields to exclude if needed.
- You can only clone an existing record (one that already exists in the database).

**Example:**  
Let’s say you want to duplicate a product but change its name:

```php
$original = new Product(23); // Load an existing product
$copy = $original->clone();  // Create a new instance with the same data

$copy->name = 'New Product Name';
$copy->insert(); // Save as a new product in the database
```

If you want to exclude more fields from being copied, just pass them as an array:

```php
$copy = $original->clone(['description', 'price']);
```

If you try to clone a model that doesn’t exist in the database, you’ll get an error.

## Query Builder

**Lightpack** models are capable [query builders](/query-builder) too. 

To get a query builder on a model, call the static method `query()`:

```php
$productQuery = Product::query();
```

Now you can access all the methods on [query builders](/query-builder). Below are some example for using query builder on a model.

**Fetch all products**
```php
$products = Product::query()->all();
```

**Fetch all active products**
```php
$products = Product::query()->where('active', '=', '1')->all();
```

**Fetch products with matching ids**
```php
$products = Product::query()->whereIn('id', [1,2,3])->all();
```

**Fetch all products with at least one order**

```php
$products = Product::query()->has('orders')->all();
```

The above is same as:

```php
$products = Product::query()->has('orders', '>', 0)->all();
// or
$products = Product::query()->has('orders', '>=', 1)->all();
```

**Fetch products with no orders**

```php
$products = Product::query()->has('orders', '=', 0)->all();
```

**Fetch products with at least 2 orders**

```php
$products = Product::query()->has('orders', '>', 1)->all();
// or
$products = Product::query()->has('orders', '>=', 2)->all();
```

**Fetch products with atmost 2 orders**

```php
$products = Product::query()->has('orders', '<', 3)->all();
// or
$products = Product::query()->has('orders', '<=', 2)->all();
```

**Callbacks as query constraints**

You can even pass a callback as **4th** parameter to `has()` method to add more constraints on relationship. For example, suppose you want to fetch **products** with atleast **2 paid orders**.

```php
$products = Product::query()->has('orders', '>=', 2, function($q) {
    $q->where('paid', '=', true);
});
```

## Cast Into Array

To convert loaded models into **array**, use `toArray()` method.

```php
$product = new Product(23);
$productArray = $product->toArray();
```

```php
$products = Product::query()->limit(10)->all();
$productsArray = $products->toArray();
```