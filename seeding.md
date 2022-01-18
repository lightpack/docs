# Seeding

Manually populating database with test data while developing the application can be cumbersome. This is where database seeders help a lot. Seeders are classes that contain logic for populating the database with sufficient test data. 

`Lightpack` provides a simple approach for populating test data with the help of database seeders.

Suppose you have a products table with following columns:

<table>
    <tr>
        <td class="token title important">products</td>
        <td>id</td>
        <td>title</td>
        <td>description</td>
    </tr>
</table>

## Creating Seeder

Fire this command from the root of your project:

```terminal
php lucy create:seeder ProductsSeeder
```

This should have created a seeder class named `ProductsSeeder` in `database/seeders` directory. 

This class has a method named `seed()` where you can write the logic for populating `products` table with test data.

For example, here we populate **25 fake products** as shown:

```php
public function seed()
{
    foreach(range(1, 25) as $index) {
        $product = new Product;
        $product->title = "Product {$index}";
        $product->description = "Product description {$index}";
        $product->save();
    }
}
```

Here in each loop, we populate a new product using the `Product` [model class](models.md).

## Calling Seeders

`Lightpack` ships with a `database/seeders/DatabaseSeeder.php` class file where you can execute the seeders you have created in the order you find appropriate.

For example, suppose we have two seeders for `brands` and `products` and you wish to seed `brands` first and then `products`. You can do so by calling seeders inside the `seed()` method of `DatabaseSeeder` class.
in that order.

For example:

```php
public function seed()
{
    (new BrandsSeeder)->seed();
    (new ProductsSeeder)->seed();
}
```

## Executing Seeders

As a final step, you need to execute all your seeders. For that, simply execute the following console command:

```terminal
php lucy db:seed
```

That's it!!