# Models

Models are classes that represent a **record/row** in a database table. For this reason, models are also called **Entities**.

Consider that you have a `products` table in your database.

```table
Table: products
-------------------------------------------------
id, name, size, color, status
-------------------------------------------------
```

Then you should define a `Product` model in <code>app/models</code> folder.

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
methods to deal with a record in your `products` table.

## Read Data

You can easily find a record by its **ID** when constructing the model. 

```php
$product = new Product(23);
```

Or you can also call the <code>find()</code> method passing it the record ID.

```php
$product = (new Product)->find(23);
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
$product = new Product(23);

$product->delete();
```

You can also do this:

```php
(new Product)->find(23)->delete();
```

## Querying Data

Every model instance inehrits `query()` method that returns a query builder object on the current model.

For example, to get all the products with active status:

```php
$product->query()->where('active', '=', 1)->all();
```

This gives a convinience that otherwise you would have done using [query builder](/db-query-builder) from `Query` class as shown.

```php
// Get the query builder 
$products = new Query('products');

// Query all active products
$products->where('active', '=', 1)->all();
```

<p class="tip">By inheriting the <code>query()</code> method, our models are violating the fact that a model class represents a single <b>record/row</b> in the corresponding database table. <br><br>Because if a model instance is capable of querying the whole table, then it ideally represents a collection of records, and not just a single record.</p>

<p class="tip">This is where the <a href="https://en.wikipedia.org/wiki/Domain-driven_design">Repository Pattern</a> shines and promotes an aspect of <b>Domain Driven Design capabilities</b> for your domain objects. But even they come with <b>disadvantages</b>. 😅</p>

## Timestamps

Document in progress...

## Hooks

Document in progress...