# Logging

**Lightpack** ships with a PSR-3 compatible logger that is already configured as a service for you.

You can access the logger instance as service using `logger()` which gives an instance of `Lightpack\Logger\Logger`. 

This class exposes following logging methods:

```php
logger()->log();
logger()->info();
logger()->alert();
logger()->debug();
logger()->notice();
logger()->warning();
logger()->critical();
logger()->emergency();
```

## Log Driver

**Lightpack** uses `file` as default log driver and logs the messages in the filename configured in `config/default.php` file.

```php
// ...
    'logger' => [
        'filename' => DIR_STORAGE . '/logs.txt',
    ],
// ...
```

### Disable Logging

It may be the case where you might want to disable logging messages. This specially is true in **shared hosting** where you would want to disable logging because of `disk space` restrictions.

To **disable** logging messages, you can set the log driver to null in `env.php` file.

```php
// ...
    /**
     * Log driver: file, null.
     */

    'LOG_DRIVER' => 'null',
// ...
```

## Available Methods

### log()

This method takes a log level and a message string to log.

```php
logger()->log('level', 'message');
```

### info()

This method sets the log level to `info` and takes a message string to log.

```php
logger()->info('message');
```

### alert()

This method sets the log level to `alert` and takes a message string to log.

```php
logger()->alert('message');
```

### debug()

This method sets the log level to `debug` and takes a message string to log.

```php
logger()->debug('message');
```

### notice()

This method sets the log level to `notice` and takes a message string to log.

```php
logger()->notice('message');
```

### warning()

This method sets the log level to `warning` and takes a message string to log.

```php
logger()->warning('message');
```

### critical()

This method sets the log level to `critical` and takes a message string to log.

```php
logger()->critical('message');
```

### emergency()

This method sets the log level to `emergency` and takes a message string to log.

```php
logger()->emergency('message');
```