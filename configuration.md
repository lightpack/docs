# Configuration

Every project needs to maintain some configuration data for the application. 
These configurations also depend on your project environment. For example, you
would prefer to have seperate configurations for staging database as compared to
production database.

`Lightpack` comes with a simple `array` based configuration approach which is lightweight
yet extendible to meet complex configuration requirements as per project.

## config()

To access a config item, simply call `config()` function passing it the configuration key.

```php
config('key'); 
```

<p class="tip">All your project configurations goes in <b>config</b> folder in your app root.</p>

## Accessing configuration

To view configurations for your application browse `config` folder. It
lists some pre-defined configurations as an `array` of key-value pairs.

For example, you can access `config/db.php` configuration file keys like:

```php
config('db.mysql.host');
config('db.mysql.port');
```

## Custom configuration

You can define your own custom configuration files as per your application needs. Say 
for example, you want to have a new configuration file for your `Redis` server. For that,
simply create a file `redis.php` in the `config` folder.

```cli
php console create:config redis
```

Now you can put your configuration details as an **array** as shown below.

```php
<?php

return [
    'redis' => [
        'host' => '127.0.0.1',
        'port' => '6666',
        'password' => '1234',
    ],
];
```

Now you can easily access the config values as shown:

```php
config('redis.host'); // 127.0.0.1
config('redis.port'); // 6666
```

## Default values

You can provide a default value as the second argument to `config()` function. This value will be returned if the configuration key does not exist.

```php
config('app.timezone', 'UTC');  // Returns 'UTC' if not set
```

## Setting configuration

You can dynamically add new configuration values at runtime by calling the `set()` method:

```php
config()->set('cache.driver', 'redis');
```

<p class="tip">The <code>set()</code> method only sets the value if the key doesn't already exist. This prevents accidental overwrites.</p>

## Checking configuration

To check if a configuration key exists, you can use the `has()` method:

```php
if (config()->has('redis.host')) {
    // Configuration exists
}
```

## Nested configuration

Configuration files can have deeply nested arrays. You can access nested values using dot notation:

```php
// config/database.php
return [
    'database' => [
        'connections' => [
            'mysql' => [
                'host' => 'localhost',
                'port' => 3306,
            ],
        ],
    ],
];

// Access nested values
config('database.connections.mysql.host');  // localhost
config('database.connections.mysql.port');  // 3306
```

---