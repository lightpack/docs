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

## Defining Migrations

The following documentation details about creating/modifying tables, columns, and indexes.

### Create Table

```php
public function up(): void
{
    $this->create('users', function(Table $table) {
        $table->id();
        $table->varchar('name');
        $table->varchar('email');
    });
}

public function down(): void
{
    $this->drop('users');
}
```

### Rename Table

```php
public function up(): void
{
    $this->rename('users', 'customers');
}

public function down(): void
{
    $this->rename('customers', 'users');
}
```

### Alter Table

You may alter an existing table definition as documented below.

#### Add New Columns

```php
public function up(): void
{
    $this->alter('users')->add(function(Table $table) {
        $table->varchar('password');
        $table->timestamps();
    });
}
```

#### Modify Existing Columns

```php
public function up(): void
{
    $this->alter('users')->modify(function(Table $table) {
        $table->varchar('name', 55);
    });
}
```

#### Drop Existing Columns

```php
public function up(): void
{
    $this->alter('users')->modify(function(Table $table) {
        $table->dropColumn('password');
    });
}
```

## Table Indexes

**Supported Index Types**

- **Primary Key**
- **Unique Index**
- **Regular Index**
- **Fulltext Index**
- **Spatial Index**

---

### primary() 

- Defines a primary key (single or composite).
- primary(string|array $columns)

```php
// single primary key
$table->primary('id');

// composite key
$table->primary(['user_id', 'post_id']);
```

### dropPrimary()

- Drops the primary key constraint (not the column).

```php
$table->dropPrimary();
```

### unique()

- Adds a unique index to one or more columns.
- `unique(string|array $columns, ?string $indexName = null)`
- Supports custom unique index name.

```php
$table->unique('email');
$table->unique(['first', 'last'], 'name_unique');
```

### dropUnique()

- Drops a unique index by name.
- `dropUnique(string $indexName)`

```php
$table->dropUnique('email');
$table->unique(['first', 'last'], 'name_unique');
```

### index()
- Adds a regular (non-unique) index.
- `index(string|array $columns, ?string $indexName = null)`
- Supports custom index name.

```php
$table->index('created_at');
$table->index(['user_id', 'status'], 'user_status_idx');
```

### dropIndex()

- Drops a regular index by name.
- `dropIndex(string $indexName)`

```php
$this->dropIndex('user_status_idx');
```

### fulltext()
- Adds a FULLTEXT index for text search.
- `fulltext(string|array $columns, ?string $indexName = null)`
- Supports custom full text index name.

```php
$table->fulltext('body');
$table->fulltext(['title', 'body'], 'post_fulltext');
```

### dropFulltext() 
- Drops one or more FULLTEXT indexes by name.
- `dropFulltext(string ...$indexName)`

```php
$table->dropFulltext('post_fulltext');
```

### spatial()
- Adds a SPATIAL index for GIS data.
- `spatial(string|array $columns, ?string $indexName = null)`
- Support passing custom spatial index name.

```php
$table->spatial('location', 'loc_idx');
```

### dropSpatial() 
- Drops a SPATIAL index by name.
- `dropSpatial(string $indexName)`

```php
$this->dropSpatial('loc_idx');
```

---