# Relationships

Relationships provide a simple way to query related data from multiple
tables. Think of <code>joins</code> based queries when associating multiple tables together.

<p class="tip">Lucid ORM not only makes it easy to associate multiple tables together, but is
also very performant.</p>

Let us consider three tables for example: <code>products</code>, <code>options</code>, <code>seo</code>.

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
        <td>value</td>
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

Let us consider that each product is allowed to have multiple options and only one meta seo
record. 

We can say that there is <code>one-to-one</code> relationship between products and seo table
and <code>one-to-many</code> relationship between products and options table.

Before relating our tables we first need to define their models as shown below.

```php
<?php

namespace App\Models;

use Lightpack\Database\Lucid\Model;

class Product extends Model
{
    public function __construct()
    {
        parent::construct('products');
    }
}
```


```php
<?php

namespace App\Models;

use Lightpack\Database\Lucid\Model;

class Option extends Model
{
    public function __construct()
    {
        parent::construct('options');
    }
}
```


```php
<?php

namespace App\Models;

use Lightpack\Database\Lucid\Model;

class Seo extends Model
{
    public function __construct()
    {
        parent::construct('seo');
    }
}
```

## Has One

It represent the <code>one-to-one</code> relationship between two models. To relate our 
<code>products</code> and <code>seo</code> table, define a method named <code>seo()</code>
in the <code>product</code> model.

```php
// ...

class Product extends Model
{
    public function seo()
    {
        return $this->hasOne('Seo', 'product_id');
    }
}
```

Now you can easily fetch seo details for a given product as a property:

```php
$product = Product::find(23);

echo $product->seo->meta_title;
echo $product->seo->meta_description;
```

It works because the <code>hasOne()</code> method actually joins the <code>products</code>
table with <code>seo</code> table. It accepts the model name as first parameter and related 
foreign key as the second parameter.

## Belongs To

It represent the inverse of <code>hasOne()</code> relationship method.

To relate the <code>Seo</code> model with <code>Product</code> model, use <code>belongsTo()</code>
method.

```php
// ...

class Seo extends Model
{
    public function product()
    {
        return $this->belongsTo('Product', 'product_id');
    }
}
```

Now you can get the product for the given seo record using product as its property.

```php
$seo = Seo::find(1);
echo $seo->product->title;
```

## Has Many

It represents the <code>one-to-many</code> relationship between two models.

```php
class Product extends Model
{
    public function options()
    {
        return $this->hasMany('Option', 'product_id');
    }
}
```

Now you can fetch all options for a given products as its property.

```php
$product = Product::find(23);

foreach($product->options as $option) {
    echo $option->name;
    echo $option->value;
}
```

### Inverse of Has Many

You should not be surprised to know that <code>belongsTo()</code> also represents the
inverse of <code>hasMany()</code>.

```php
class Option extends Model
{
    public function product()
    {
        return $this->belongsTo('Product', 'product_id');
    }
}
```

Now you can access the product that belongs to an option as its property.

```php
$option = Option::find(1);

echo $option->product->title;
```

## Many to Many

<code>Many to Many</code> relationships are often represented using a `junction` aka `pivot` table in the database.

Let us consider `users` and `roles` table. Considering that a user may have multiple roles and a role may belong to multiple
users, there is a many-to-many relationship between `users` and `roles ` table. To represent this relationship, we need to
have a third table `user_role` or `role_user`. This table is called a `junction` or `pivot` table.

<table>
    <tr>
        <td class="token title important">users</td>
        <td>id</td>
        <td>name</td>
    </tr>
</table>

<table>
    <tr>
        <td class="token title important">roles</td>
        <td>id</td>
        <td>name</td>
    </tr>
</table>

<table>
    <tr>
        <td class="token title important">user_role</td>
        <td>user_id</td>
        <td>role_id</td>
    </tr>
</table>

Lightpack supports `many-to-many` relationship using `pivot()` method. Simply define a method named `roles()` in the `User` model.

```php
class User extends Model
{
    public function roles()
    {
        return $this->pivot(Role::class, 'user_role', 'user_id', 'role_id');
    }
}
```

Now you can access all the roles that a user may have as `$user->roles`.

```php
$user->find(23);

foreach($user->roles as $role) {
    echo $role->name;
}
```