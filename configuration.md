# Configuration

Every project needs to maintain some configuration data for the application. 
These configurations also depend on your project environment. For example, you
would prefer to have seperate configurations for staging database as compared to
production database.

`Lightpack` comes with a simple `array` based configuration approach which is lightweight
yet extendible to meet complex configuration requirements as per project.

`Lightpack` already registers your project configuration service provider with `config` key.

```php
$config = app('config');
```

All your project configurations goes under `config` folder in your app root.

## Default configuration

To view default configurations for your application browse `config/default.php` file. It
lists some pre-defined configurations as an `array` of key-value pairs.

Calling `app('config')->default` returns the complete array items defined as default configuration.

For example, consider the following configuration array:

```php
<?php

return [
    'environment' => 'development',
    'site' => [
        'url' => 'http://localhost',
        'timezone' => 'UTC',
        'locale' => 'en',
        'default_locale' => 'en',
    ],
    ...
];
```

To access a config item, simply specify it as an array key.

```php
app('config')->default['environment']; // development
app('config')->default['site']['locale']; // en
app('config')->default['site']['timezone']; // UTC
```

## Custom configuration

You can define your own custom configuration files as per your application needs. Say 
for example, you want to have a new configuration file for your `Redis` server. For that,
simply create a file `redis.php` in the `config` folder and put your configuration details
as an **array** as show below.

```php
<?php

return [
    'host' => '127.0.0.1',
    'port' => '6666',
    'password' => '1234',
];
```

Now you need to register this new `redis` config file in `bootstrap/services.php` file
as shown.

```php
<?php

...
/**
 * ------------------------------------------------------------
 * Register Configuration Service Provider.
 * ------------------------------------------------------------
 */

$container->register('config', function($container) {
    return new Lightpack\Config\Config([
        'default', 
        'events', 
        'filters', 
        'cors', 
        'redis'
    ]);
});
...
```

**That's it**. Now you can easily access the config values as shown:

```php
<?php

app('config')->redis['host']; // 127.0.0.1
app('config')->redis['port']; // 6666
```

**Note:** You can also get the whole configuration array as shown in the example below.

```php
<?php

$redis = app('config')->redis;

$redis['host']; // 127.0.0.1
$redis['port']; // 6666
```

`@todo` Documentation in progress...