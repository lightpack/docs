# Environments

You may want to change how your application behaves on a production server than on a local
development machine. For example, you should display all PHP errors when developing your
app but you should turn off error display to end users when your application goes live.

## .env

**Lightpack** ships with a sample configuration file where some pre-defined environment configurations are available for your reference. 

When you install **Lightpack**, it creates a `.env` file in the root directory of your project. This file contains a set of environment variables that you can use as a template for your own environment variables.

You can also generate the **.env** file from command line itself.

```terminal
php console create:env
```

<p class="tip">You should not commit your <b>.env</b> file to your version control system. Keep it in <b>.gitignore</b>.</p>

## .env format

The `.env` file uses a simple `KEY=VALUE` format:

```bash
APP_ENV=production
APP_DEBUG=false
DB_HOST=localhost
DB_PORT=3306
DB_NAME=myapp
```

### Comments

You can add comments using the `#` symbol:

```bash
# Database configuration
DB_HOST=localhost
DB_PORT=3306
```

### Special values

The following special values are automatically converted:

```bash
APP_DEBUG=true      # Converted to boolean true
APP_DEBUG=false     # Converted to boolean false
CACHE_DRIVER=null   # Converted to null
```

### Variable interpolation

You can reference other environment variables using `${VAR}` syntax:

```bash
DB_HOST=localhost
DB_URL=mysql://${DB_HOST}:3306/myapp
```

## get_env()

To access environment configurations, call `get_env()` function.

```php
get_env('APP_ENV', 'development');
```

This function takes two arguments. First is the `environment` variable name and second is the `default` value in case it does not find that environment variable.

## set_env()

This function sets an environment variable at runtime.

```php
set_env('APP_VERSION', 'v1.0');
```

<p class="tip">Changes made with <code>set_env()</code> only affect the current request. They are not persisted to the <b>.env</b> file.</p>

## Using in configuration

Environment variables are commonly used in configuration files:

```php
// config/database.php
return [
    'database' => [
        'host' => get_env('DB_HOST', 'localhost'),
        'port' => get_env('DB_PORT', '3306'),
        'name' => get_env('DB_NAME', 'myapp'),
    ],
];
```

This allows you to change configuration based on the environment without modifying code.

## Environment-specific behavior

You can use environment variables to change application behavior:

```php
if (get_env('APP_ENV') === 'production') {
    // Production-specific code
    ini_set('display_errors', 'off');
} else {
    // Development-specific code
    ini_set('display_errors', 'on');
}
```

---