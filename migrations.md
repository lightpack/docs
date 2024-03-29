# Migrations

**Lightpack** supports the concept of pure `SQL` based database migrations. So except learning a couple of migration commands to run, you need to learn nothing else.

**The goal is to:**

* reduce learning curve by not introducing another layer of classes and method APIs
* and not make awesome tools like MySQL Workbench, Sequel Pro, phpMyAdmin, MyJetBrains DataGrip, etc. worthless.

<p class="tip">Please note that only <b>MySQL/MariaDB</b> based migrations are supported.</p>

## Creating Migrations

To create a new migration file, fire this command from console:

```terminal
php lucy create:migration add_products_table
```

This will create the migration file prefixed with current **datetime** in `database/migrations` folder. 

You can define all your migration scripts as pure **SQL** inside `up()` method. Any reverse operations should go inside `down()` folder.

```php
public function up(): void
{
    $sql = "CREATE TABLE products (id int(11) NOT NULL AUTO_INCREMENT, `name` varchar(255) NOT NULL)";

    // Execute the SQL 
    $this->connection->query($sql);
}
```

## Running Migrations

To run your migration files:

```terminal
php lucy migrate:up
```

This command will run the `up()` method in all the migration scripts defined inside **database/migrations** folder. 

To track the files that have been migrated, it will also create a `migrations` table in the database.

## Rollback Migrations

To rollback or undo all your migrations:

```terminal
php lucy migrate:down
```

This will run the `down()` method in all the migration scripts inside **database/migrations** folder.

To rollback a limited number of migrations, provide the `steps` flag:

```terminal
php lucy migrate:down --steps=2
```

This will rollback last two **batches** of migrations if present.