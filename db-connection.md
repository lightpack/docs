# Connecting to a Database

Lightpack aims to provide a performant thin layer of abstraction for easing working with relational database systems. Currently it supports PDO adapters for
<code>Sqlite</code> and <code>MySQL</code>.

To get a new connection, simply instantiate an adapter class passing it 
the database connections options.

```php
$mysql = new \Framework\Database\Adapters\Mysql($options);
$sqlite = new \Framework\Database\Adapters\Sqlite($options);
```        

<p class="tip">Once you make a database connection, you can start querying against it using the <a href="https://www.php.net/manual/en/book.pdo.php" target="_blank">PHP PDO APIs</a>.
</p>

## Configuration

You should set the database configuration options in <code>config/database.php</code> file. If you open the database config file, you will find configurations
defined for <code>mysql</code> and <code>sqlite</code>. Change them as per your
database options. 

You can access an adpater's configuration using <code>app('config')</code> function call.

```php
$options = app('config')->database['mysql'];
```

## Default Connection

Most of the time you would want to have a single instance of database connection to avoid re-connecting database multiple times throughout your
application lifecycle. For that you can register your database instance as a service in the service configuration file at <code>config/services.php</code>. 

For example, to create an instance of MySQL database connection and store it 
in the service connector, you can register it as a service as shown below.

```php
$container->register('mysql', function($container) {
    return new Framework\Database\Adapters\MySql(
        $container->get('config')->database['mysql']
    );
});
```

<p class="tip">For your convinience, Lightpack already comes with a registered service for MySQL in <code>config/services.php</code> file. You can simply access the connection using <code>app('db')</code> function call.</p>

## Raw Queries

Once you get a database connection, you can execute raw queries against it using the <code>query()</code>
method.

```php
$db->query('SELECT * FROM products WHERE id = 23');
```

This method optionally takes an array of parameters as its second argument to protect against SQL injection attacks.

```php
$db->query('SELECT * FROM products WHERE id = :id', [':id' = 23]);
```