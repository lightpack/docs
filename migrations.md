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

### Boolean/Bit Columns
- **boolean(string $name, bool $default = false)**: TINYINT(1), default 0/1

### Special Columns
- **ipAddress(string $name = 'ip_address')**: VARCHAR(45) for IPv4/IPv6
- **macAddress(string $name = 'mac_address')**: VARCHAR(17)
- **morphs(string $name)**: Adds `{name}_id` (BIGINT UNSIGNED) and `{name}_type` (VARCHAR(255)) for polymorphic relations

---

### Column Configuration (via Column object)
All column methods return a `Column` object, allowing further configuration:
- **type(string $type)**: Set SQL type manually
- **length(int $length)**: Set length for applicable types
- **default(mixed $value)**: Set default value
- **nullable()**: Mark column as nullable
- **attribute(string $attr)**: Add SQL attribute (e.g., UNSIGNED, ZEROFILL)
- **increments()**: Set AUTO_INCREMENT
- **enum(array $values)**: Set ENUM values

**Example:**
```php
$table->varchar('username', 50)->unique()->nullable();
$table->int('age')->default(0);
$table->decimal('balance', 12, 2)->attribute('UNSIGNED');
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

## Foreign Keys
**Foreign keys** enforce referential integrity between tables, ensuring that a column (or set of columns) in one table matches the primary key or unique key in another table. Foreign keys can be defined during table creation or added/removed during schema alteration. The following document details the support for working with foreign keys.


### foreignKey()

- `foreignKey(string $column)`
- Defines a foreign key on the specified column.

```php
$table->foreignKey('user_id')
    ->references('id')
    ->on('users')
    ->onDelete('CASCADE') // CASCADE, SET NULL, RESTRICT
    ->onUpdate('CASCADE'); // CASCADE, SET NULL, RESTRICT
```

### dropForeign()

- `dropForeign(string ...$constraintNames)`
- Drop one or more foreign key constraints by name.

```php
$table->dropForeign('users_user_id_foreign');
```

---