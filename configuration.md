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

For example, open `config/db.php` configuration file:

```php
// ...
'db.mysql.host' => get_env('DB_HOST'),
'db.mysql.port' => get_env('DB_PORT'),
'db.mysql.username' => get_env('DB_USER'),
'db.mysql.password' => get_env('DB_PSWD'),
'db.mysql.database' => get_env('DB_NAME'),
// ...
```

To access a config item, simply specify it as a key.

```php
config('db.mysql.host');
config('db.mysql.port');
```

## Custom configuration

You can define your own custom configuration files as per your application needs. Say 
for example, you want to have a new configuration file for your `Redis` server. For that,
simply create a file `redis.php` in the `config` folder and put your configuration details
as an **array** as show below.

```php
<?php

return [
    'redis.host' => '127.0.0.1',
    'redis.port' => '6666',
    'redis.password' => '1234',
];
```

**That's it**. Now you can easily access the config values as shown:

```php
config('redis.host'); // 127.0.0.1
config('redis.port'); // 6666
```

## Changing configuration

You can dynamically add new configuration values at runtime by simply
setting the config item key as show below.

```php
app('config')->set('key', 'value');
```