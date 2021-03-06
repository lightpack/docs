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

Let us consider that each product is allowed to have multiple options and only one meta seo
record. 

We can say that there is <code>one-to-one</code> relationship between products and seo table
and <code>one-to-many</code> relationship between products and options table.

Before relating our tables we first need to define their models as shown below.

Fire these three model code generation commands from your terminal inside project root.

```terminal
php lucy create:model Seo --table=seo
php lucy create:model Option --table=options
php lucy create:model Product --table=products
```

It should have created three model classes inside `app/Models` folder. Now let us understand how to use relationships among these three models.

## Has One

It represents the <code>one-to-one</code> relationship between two models. To relate our 
<code>products</code> and <code>seo</code> table, define a method named <code>seo()</code>
in the <code>product</code> model.

```php
class Product extends Model
{
    public function seo()
    {
        return $this->hasOne(Seo::class, 'product_id');
    }
}
```

Now you can easily fetch seo details for a given product as its **property**:

```php
// Find product seo data
$seo = (new Product)->find(23)->seo;

// Access the values
echo $seo->meta_title;
echo $seo->meta_description;
```

It works because the <code>hasOne()</code> method actually joins the <code>products</code>
table with <code>seo</code> table. It accepts the model name as first parameter and related 
foreign key as the second parameter.

## Belongs To

It represent the inverse of <code>hasOne()</code> relationship method.

To relate the <code>Seo</code> model with <code>Product</code> model, use <code>belongsTo()</code>
method.

```php
class Seo extends Model
{
    public function product()
    {
        return $this->belongsTo(Product::class, 'product_id');
    }
}
```

Now you can get the product for the given seo record using product as its property.

```php
// Get the product details
$product = (new Seo)->find(23)->product;

// Access the values
echo $product->name;
echo $product->color;
```

## Has Many

It represents the <code>one-to-many</code> relationship between two models.

```php
class Product extends Model
{
    public function options()
    {
        return $this->hasMany(Option::class, 'product_id');
    }
}
```

Now you can fetch all options for a given product as its property.

```php
// Find product options
$options = (new Product)->find(23)->options;

// Access each option
foreach($options as $option) {
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
        return $this->belongsTo(Product::class, 'product_id');
    }
}
```

Now you can access the product that belongs to an option as its property.

```php
$product = (new Option)->find(23)->product;
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
$user = new User(23);

foreach($user->roles as $role) {
    echo $role->name;
}
```

## Conditional Queries

Documentation in progress...