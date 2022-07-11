# Models

Models are classes that represent a table in your database. Consider that you have a `products` table in your database.

```table
Table: products
-------------------------------------------------
id, name, size, color, status
-------------------------------------------------
```

Then you should define a `Product` model in <code>app/Models</code> folder.

## Creating Model

Fire this command to generate a model from your terminal inside your project root.

```terminal
php lucy create:model Product --table=products
```

This should have created `Product` model in `app/Models` folder.

```php
class Product extends Model
{
    /** @inheritDoc */
    protected $table = 'products';

    /** @inheritDoc */
    protected $primaryKey = 'id';

    /** @inheritDoc */
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

Now you can easily access the column values as model properties.

```php
echo $product->title;
echo $product->size;
echo $product->color;
```

## Insert Data

Set properties on your new model and simply call the inherited method <code>save()</code> 
to insert a new record.

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

You can only update a record that exists in the database table, right? So you first need
to <b>"find"</b> your model by passing the product **ID**. Rest of the steps is completely
same as inserting data.

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


> Fetch all products
```php
$products = Product::query()->all();
```

> Fetch all active products
```php
$products = Product::query()->where('active', '=', '1')->all();
```

> Fetch products with matching ids
```php
$products = Product::query()->whereIn('id', [1,2,3])->all();
```