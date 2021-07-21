# Connecting to a Database

Lightpack aims to provide a performant thin layer of abstraction for easing working with relational database systems. Currently it supports **PDO** adapters for
<code>Sqlite</code> and <code>MySQL</code>.

## Configuration

Before you get set with a databse connection, you need to configure database credentials. 

You can set your database credentials in the [environment configuration](/environments) file.

```php
/**
 * MySQL settings.
 */

'DB_HOST' => 'localhost',
'DB_PORT' => 3306,
'DB_NAME' => '',
'DB_USER' => '',
'DB_PSWD' => '',
``` 

Now you can get a MySQL database connection by simply calling `app('db')`.

```php
$db = app('db');
```

<p class="tip">Once you make a database connection, you can start querying against it using the <a href="https://www.php.net/manual/en/book.pdo.php" target="_blank">PHP PDO APIs</a>.
</p>

## Drivers

You can change the database driver in [environment configuration](/environments) file. The two supported drivers are `mysql` and `sqlite`.

```php
DB_DRIVER => 'mysql';
```

## Raw Queries

<p class="tip">Once you make a database connection, you can start querying against it using the <a href="https://www.php.net/manual/en/book.pdo.php" target="_blank">PHP PDO APIs</a>.
</p>

You can execute raw queries against the database connection using the <code>query()</code>
method.

```php
app('db')->query('SELECT * FROM products WHERE id = 23');
```

This method optionally takes an array of parameters as its second argument to protect against SQL injection attacks.

```php
app('db')->query('SELECT * FROM products WHERE price > ?', [500]);
```

Ofcourse you can also use named **placeholders** in your raw queries.

```php
app('db')->query('SELECT * FROM products WHERE id = :id', [':id' => 23]);
```

You can also execute **insert** and **update** raw queries.

```php
app('db')->query('INSERT INTO products (name) VALUES ('Blue Denim'));
```

```php
app('db')->query('UPDATE articles SET status = ?', ['active']);
```

## Query Logging

To log queries that got executed throught an application request lifecycle, there are two methods that you can work with.

To get all the query logs as an array:

```php
app('db')->getQueryLogs();
```

To print all the query logs:

```php
app('db')->printQueryLogs();
```
