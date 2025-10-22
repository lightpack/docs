# Migrations

Migrations provide version control for your database schema. Each migration is a PHP class that represents a set of database changes.

<p class="tip">Please note that only <b>MySQL/MariaDB</b> based migrations are supported.</p>

## Creating Migrations

To create a new migration file, fire this command from console:

```terminal
php console create:migration create_table_products
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
php console migrate:up
```

This command will run the `up()` method in all the migration scripts defined inside **database/migrations** folder. 

To track the files that have been migrated, it will also create a `migrations` table in the database.

## Rollback Migrations

To rollback or undo all your migrations:

```terminal
php console migrate:down
```

This will run the `down()` method in all the migration scripts inside **database/migrations** folder.

To rollback a limited number of migrations, provide the `steps` flag. For example, 
this will rollback last two **batches** of migrations if present.

```terminal
php console migrate:down --steps=2
```

To rollback all the migrations in one go, provide the `--all` flag:

```terminal
php console migrate:down --all
```

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

### Truncate Table

```php
public function up(): void
{
    $this->truncate('users');
}
```

### Execute Raw SQL

You can execute raw SQL queries when needed:

```php
public function up(): void
{
    $this->execute('ALTER TABLE users ADD COLUMN custom_field VARCHAR(255)');
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
    // Drop single column
    $this->alter('users')->dropColumn('password');
}

public function down(): void
{
    // Drop multiple columns at once
    $this->alter('users')->dropColumn('password', 'email', 'phone');
}
```

#### Rename Column

```php
public function up(): void
{
    $this->alter('users')->renameColumn('name', 'full_name');
}

public function down(): void
{
    $this->alter('users')->renameColumn('full_name', 'name');
}
```

## Table Columns

This documentation summarizes all available column types, their configuration options, and usage patterns.

### Numeric Columns
- **id(string $name = 'id')**: BIGINT UNSIGNED AUTO_INCREMENT primary key
- **int(string $name, int $length = 11)**: INT
- **bigint(string $name)**: BIGINT
- **smallint(string $name)**: SMALLINT
- **tinyint(string $name)**: TINYINT
- **decimal(string $name, int $precision = 10, int $scale = 2)**: DECIMAL(precision, scale)

### String/Text Columns
- **varchar(string $name, int $length = 255)**: VARCHAR(length)
- **char(string $name, int $length = 255)**: CHAR(length)
- **text(string $name)**: TEXT
- **tinytext(string $name)**: TINYTEXT
- **mediumtext(string $name)**: MEDIUMTEXT
- **longtext(string $name)**: LONGTEXT
- **enum(string $name, array $values)**: ENUM(values)
- **json(string $name)**: JSON

### Date/Time Columns
- **date(string $name)**: DATE
- **time(string $name)**: TIME
- **datetime(string $name)**: DATETIME
- **timestamp(string $name)**: TIMESTAMP
- **year(string $name)**: YEAR
- **createdAt()**: DATETIME, default CURRENT_TIMESTAMP
- **updatedAt()**: DATETIME, nullable, ON UPDATE CURRENT_TIMESTAMP
- **deletedAt()**: DATETIME, nullable
- **timestamps()**: Adds both createdAt and updatedAt

### Boolean Columns
- **boolean(string $name, bool $default = false)**: TINYINT(1) with default value 0 or 1

### Special Columns
- **ipAddress(string $name = 'ip_address')**: VARCHAR(45) for IPv4/IPv6
- **macAddress(string $name = 'mac_address')**: VARCHAR(17)
- **morphs(string $name)**: Adds `{name}_id` (BIGINT UNSIGNED) and `{name}_type` (VARCHAR(255)) for polymorphic relations

---

### Column Configuration (via Column object)
All column methods return a `Column` object, allowing further configuration:
- **type(string $type)**: Set SQL type manually
- **length(int $length)**: Set length for applicable types
- **default(bool|string $value)**: Set default value
- **nullable(bool $nullable = true)**: Mark column as nullable
- **attribute(string $attr)**: Add SQL attribute (e.g., UNSIGNED, ON UPDATE CURRENT_TIMESTAMP)
- **unsigned()**: Fluent shortcut for UNSIGNED attribute
- **current()**: Fluent shortcut for CURRENT_TIMESTAMP default
- **increments()**: Set AUTO_INCREMENT and PRIMARY KEY
- **primary(?string $indexName = null)**: Mark as primary key
- **unique(?string $indexName = null)**: Add unique index
- **index(?string $indexName = null)**: Add regular index
- **fulltext(?string $indexName = null)**: Add fulltext index

**Example:**
```php
$table->varchar('username', 50)->unique()->nullable();
$table->int('age')->default(0);
$table->int('score')->unsigned();
$table->decimal('balance', 12, 2)->attribute('UNSIGNED');
$table->datetime('created_at')->current();
```

---

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

<p class="tip"><b>Important Notes:</b></p>

- Dropping the primary key does not remove the column from the table but only the primary key constraint.
- Remember that there can be only one auto column and it must be defined as a key. So if the primary key is defined as auto, you will need to remove the auto attribute first.
- Do not forget to drop or alter foreign keys referencing the primary key.

```php
$table->dropPrimary();
```

### unique()

- Adds a unique index to one or more columns.
- `unique(string|array $columns, ?string $indexName = null)`
- Supports custom unique index name.

<p class="tip"><b>Note:</b> You should remove duplicate values from the columns before adding unique index otherwise it may result in "mysql error 1062".</p>

```php
$table->unique('email');
$table->unique(['first', 'last'], 'name_unique');
```

### dropUnique()

- Drops a unique index by name.
- `dropUnique(string $indexName)`

```php
$this->alter('users')->dropUnique('email_unique');
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
$this->alter('users')->dropIndex('user_status_idx');
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
// Drop single fulltext index
$this->alter('posts')->dropFulltext('post_fulltext');

// Drop multiple fulltext indexes
$this->alter('posts')->dropFulltext('title_fulltext', 'body_fulltext');
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
$this->alter('locations')->dropSpatial('loc_idx');
```

---

## Table Configuration

You can configure table engine, charset, and collation:

```php
public function up(): void
{
    $this->create('users', function(Table $table) {
        $table->id();
        $table->varchar('name');
        
        // Configure table settings
        $table->engine('InnoDB');  // Default: InnoDB
        $table->charset('utf8mb4'); // Default: utf8mb4
        $table->collation('utf8mb4_unicode_ci'); // Default: utf8mb4_unicode_ci
    });
}
```

---

## Foreign Keys
**Foreign keys** enforce referential integrity between tables, ensuring that a column (or set of columns) in one table matches the primary key or unique key in another table. Foreign keys can be defined during table creation or added/removed during schema alteration. The following document details the support for working with foreign keys.


### Defining Foreign Keys

To define a foreign key constraint, use the `foreignKey()` method:

```php
$table->foreignKey('author_id')
    ->references('id')
    ->on('users')
    ->cascadeOnDelete();
```

### Foreign Key Actions

Available actions for `ON UPDATE` and `ON DELETE`:

- **CASCADE**: Automatically update/delete related rows
- **RESTRICT**: Prevent update/delete if related rows exist (default)
- **SET NULL**: Set foreign key column to NULL

**Available Methods:**
- `cascadeOnDelete()`: Set ON DELETE CASCADE
- `cascadeOnUpdate()`: Set ON UPDATE CASCADE
- `restrictOnDelete()`: Set ON DELETE RESTRICT (default)
- `restrictOnUpdate()`: Set ON UPDATE RESTRICT (default)
- `nullOnDelete()`: Set ON DELETE SET NULL
- `nullOnUpdate()`: Set ON UPDATE SET NULL

<p class="tip"><b>Note:</b> Default behavior is RESTRICT for both ON UPDATE and ON DELETE.</p>

**Example:**
```php
$table->foreignKey('category_id')
    ->references('id')
    ->on('categories')
    ->nullOnDelete()      // Set to NULL when parent is deleted
    ->restrictOnUpdate(); // Prevent parent updates if children exist
```

### Dropping Foreign Keys

To drop one or more foreign key constraints:

```php
public function up(): void
{
    // Drop single foreign key
    $this->alter('posts')->dropForeign('posts_user_id_foreign');
    
    // Drop multiple foreign keys
    $this->alter('posts')->dropForeign('posts_user_id_foreign', 'posts_category_id_foreign');
}
```

---