# Environments

You may want to change how your application behaves on a production server than on a local
development machine. For example, you should display all PHP errors when developing your
app but you should turn off error display to end users when your application goes live.

## .env

**Lightpack** ships with a sample configuration file where some pre-defined environment configurations are available for your reference. 

Copy and paste the contents of `.env.example` file into `env.php` file in the same folder. There
you can define as many environment configurations as you wish.

You can also generate the **.env** file from command line itself.

```terminal
php lucy create:env
```

<p class="tip">You should not commit your environment specific config files to your version control system.</p>

## get_env()

To access environment configurations, call `get_env()` function.

```php
get_env('APP_ENV', 'development');
```

This function takes two arguments. First is the `environment` variable name and second is the `default` value in case it does not find that environment variable.

## set_env()

This function sets an environment variable.

```php
set_env('APP_VERSION', 'v1.0');
```

---