# Connecting to a Database

Lightpack aims to provide a performant thin layer of abstraction for easing working with relational database systems. Currently it supports **PDO** adapters for
<code>Sqlite</code> and <code>MySQL</code>.

## Configuration

Before you get set with a databse connection, you need to configure database credentials. For that, configure `env.php` and look for 'MySQL` settings to set your database credentials. For example:

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

Now you can get a MySQL database connect by simply calling `app('db')`.

```php
app('db')
```

<p class="tip">Once you make a database connection, you can start querying against it using the <a href="https://www.php.net/manual/en/book.pdo.php" target="_blank">PHP PDO APIs</a>.
</p>

## Raw Queries

Once you get a database connection, you can execute raw queries against it using the <code>query()</code>
method.

```php
app('db')->query('SELECT * FROM products WHERE id = 23');
```

This method optionally takes an array of parameters as its second argument to protect against SQL injection attacks.

```php
app('db')->query('SELECT * FROM products WHERE price > ?', [500]);
```

Or you can also use named placeholders.

```php
app('db')->query('SELECT * FROM products WHERE price > ?', [500]);
```

## Adapters

Lightpack supports `MySQL` and `Sqlite` adapters. To manually create a new connection, you can instantiate an adapter class passing it 
the database connections options.

```php
$mysql = new Lightpack\Database\Adapters\Mysql($options);
$sqlite = new Lightpack\Database\Adapters\Sqlite($options);
```       