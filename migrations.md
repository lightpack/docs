# Migrations

Migrations provide version control for your database schema. Each migration is a PHP class that represents a set of database changes.

<p class="tip">Please note that only <b>MySQL/MariaDB</b> based migrations are supported.</p>

## Creating Migrations

To create a new migration file, fire this command from console:

```terminal
php lucy create:migration create_table_products
```

This will create the migration file prefixed with current **datetime** in `database/migrations` folder. The migration class contains methods `up()` and `down()`.

The `up()` method contains definition for required schema changes. Any reverse operations should go inside `down()` folder.

| Method  | Called when                | Purpose                                                                 | Common contents                                                   |
|---------|----------------------------------|------------------------------------------------------------------------|-------------------------------------------------------------------|
| `up()`  | When you apply or run a migration| Build or evolve the schema—create tables, add columns/indexes/constraints, insert seed data that must exist, rename things, etc. | DDL or data-manipulating statements written in the migration DSL   |
| `down()`| When you roll back or undo a migration | Reverse whatever `up()` did so the database returns to its previous state | The inverse DDL (drop tables, remove columns, delete seed rows, etc.) |

> In short: `up()` is the **do** part of a migration; `down()` is the **undo**.</p>

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