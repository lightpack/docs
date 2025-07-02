# Connecting to a Database

Lightpack aims to provide a performant thin layer of abstraction for easing working with relational database systems. Currently it supports **PDO** adapters for <code>MySQL/MariaDB</code>.

## Configuration

Before you get set with a databse connection, you need to configure database credentials. 

You can set your database credentials in the [environment configuration](/environments) file.

```php
DB_HOST=localhost
DB_PORT=3306
DB_NAME=mydb
DB_USER=root
DB_PSWD=password
``` 

Now you can get a MySQL database connection by simply calling `db()` function.

## Raw Queries

> **Tip:** Once you have a database connection, you can use the full power of the [PHP PDO APIs](https://www.php.net/manual/en/book.pdo.php).

You can execute raw SQL queries using the `query()` method. This always returns a `PDOStatement` object, so you can fetch rows, columns, etc.

```php
// Select with no parameters
$stmt = db()->query('SELECT * FROM products WHERE id = 23');
$row = $stmt->fetch();
```

> **Warning:** Always use parameter binding to prevent SQL injection! Pass values as the second argument (positional or named):

```php
// Positional parameters
$stmt = db()->query('SELECT * FROM products WHERE price > ?', [500]);

// Named parameters
$stmt = db()->query('SELECT * FROM products WHERE id = :id', [':id' => 23]);
```

You can also execute **insert** and **update** queries safely:

```php
// Insert
$stmt = db()->query('INSERT INTO products (name) VALUES (?)', ['Blue Denim']);

// Update
$stmt = db()->query('UPDATE articles SET status = ?', ['active']);
```

> **Note:** The `query()` method returns a `PDOStatement` for all query types. Use PDO methods like `fetch()`, `fetchAll()`, or `rowCount()` as needed.

## Query Logging

> **Tip:** Query logging helps you debug and optimize your database usage by recording every SQL statement executed during a request.

Query logging is **only enabled when `APP_DEBUG=true`** in your environment.

To retrieve all query logs as an array:

```php
$logs = db()->getQueryLogs();
```

To print all query logs (pretty-prints to output):

```php
db()->printQueryLogs();
```

To clear the query logs (useful in tests or long-running scripts):

```php
db()->clearQueryLogs();
```

> **Note:** Query logs contain both the SQL statements and their parameter bindings. This is invaluable for debugging complex issues or performance bottlenecks.

## Transactions

Transactions can be managed manually or with a convenient closure-based API.

### Manual Transaction Control

Start a transaction:
```php
db()->begin();
```
Commit the transaction:
```php
db()->commit();
```
Rollback the transaction:
```php
db()->rollback();
```

### Closure-Based Transactions (Recommended)

The easiest and safest way to run multiple queries atomically is with a closure:

```php
db()->transaction(function() {
    db()->query('INSERT INTO products (name) VALUES (?)', ['T-shirt']);
    db()->query('UPDATE users SET points = points - 10 WHERE id = ?', [1]);
});
```

- If the closure throws an exception, the transaction is automatically rolled back.
- If it completes, the transaction is committed.

> **Tip:** You can return a value from the closure:
> 
```php
$id = db()->transaction(function() {
    db()->query('INSERT INTO products (name) VALUES (?)', ['Sneakers']);
    return db()->lastInsertId();
});
```
