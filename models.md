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

## Read Data

You can easily find a record by its **ID** when constructing the model. 

```php
$product = new Product(23);
```

Now you can access the column values as model properties.

```php
echo $product->title;
echo $product->size;
echo $product->color;
```

## Insert Data

Set properties on your new model and simply call the inherited method <code>save()</code> to insert a new record.

```php
// Create the instance
$product = new Product;

// Set product properties
$product->name = 'ACME Shoes';
$product->size = 10;
$product->color = 'black'

// Save new product
$product->save();
```
### Last Insert ID

If your table's primary key is an **auto-incrementing** field, you can get the last insert id:

```php
$product->lastInsertId();
```

### Save and Refresh
To insert a model and repopulate it with the newly created record, use `saveAndRefresh()` method:

```php
// Create the instance
$product = new Product;

// Set product properties
$product->name = 'ACME Shoes';
$product->size = 10;
$product->color = 'black'

// Save new product
$product->saveAndRefresh();
```

## Update Data

Use the same `save()` method to update an existing record in the database table. You first need to instantiate the model using the primary key of the table.

```php
// Get the product
$product = new Product(23);

// Set product properties 
$product->name = 'ACME Footwear';
$product->size = 11;
$product->color = 'brown'

// Update the product
$product->save();
```

## Delete Data

Simply call the <code>delete()</code> method.

```php
(new Product)->delete(23);
```

If you already have an existing instance of model, you can call `delete()` method there too.

```php
$product = new Product(23);
$product->delete();
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