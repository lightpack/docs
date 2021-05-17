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

`@todo` Documentation in progress...