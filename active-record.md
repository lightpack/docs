# Models

You should define your model classes in <code>app/models</code> folder.
Simply call the parent constructor by explicitly passing it the <code>table</code>
name that your model should represent.

```php
<?php

namespace App\Models;

use Framework\Database\Lucid\Model;

class Product extends Model
{
    public function __construct()
    {
        parent::__construct('products');
    }
}
```

Defining your model in this manner gives you access to a number of utility
methods for the most frequently used case scenarios. Simply create an instance of 
<code>Product</code> to start accessing the available utility methods.

```php
$product = new Product();
```

## Insert Data

Set properties on your model and simply call the inherited method <code>save()</code> 
to insert a new record.

```php
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
$product->find(23);
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
$product->find(23);

// Set new properties to update
$product->title = 'ACME Footwear';
$product->color = 'black'

// Update the product
$product->save();
```

## Delete Data

Simply call the <code>delete()</code> method.

```php
$product->find(23)->delete();
```