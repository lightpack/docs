# Models

You should define your model classes in <code>app/models</code> folder.

Fire this command to generate a model from your terminal inside your project root.

```terminal
php lucy create:model Product --table=products
```

This should have created `Product` model in `app/Models` folder.

```php
class Product extends Model
{
    public function __construct()
    {
        parent::__construct('products');
    }
}
```

Defining your model in this manner gives you access to a number of utility
methods for the most frequently used case scenarios. 

## Insert Data

Set properties on your model and simply call the inherited method <code>save()</code> 
to insert a new record.

```php
$product = new Product;

// Set product properties
$product->title = 'ACME Shoes';
$product->size = 10;
$product->color = 'black'

// Save new product
$product->save();
```

## Read Data

Simply call the <code>find()</code> method passing it the record ID.

```php
$product = (new Product)->find(23);
```

Now you can easily access the record properties.

```php
echo $product->title;
echo $product->size;
echo $product->color;
```

## Update Data

You can only update a record that exists in the database table, right? So you first need
to <b>"find"</b> your model by passing the record ID. Rest of the steps is completely
same as inserting data.

```php
// Find our product with ID
$product = (new Product)->find(23);

// Set new properties to update
$product->title = 'ACME Footwear';
$product->color = 'black'

// Update the product
$product->save();
```

## Delete Data

Simply call the <code>delete()</code> method.

```php
(new Product)->find(23)->delete();
```

## Querying Data

Inside any method of your model, you can access SQL query builder by calling the `query()` method.

```php
class Product extends Model
{
    public function findActiveProducts()
    {
        return $this
                ->query()
                ->select(['price', 'title'])
                ->where('active', '=', 1)
                ->fetchAll();
    }
}
```