# Environments

You may want to change how your application behaves on a production server than on a local
development machine. For example, you should display all PHP errors when developing your
app but you should turn off error display to end users when your application goes live.

This is what environments are for. You can configure your application as per your current 
environment which for example can be: `development`, `staging`, `production`, etc.

**Lightpack** ships with a configuration file where you put your environment based configurations. Copy and paste the contents of `env.example.php` file into `env.php` file in the same folder. There
you can define as many environment configurations as you wish.

For example, you can set `APP_ENV` configuration to `production` from `development`.

Now to access these configurations, call `get_env()` function.

```php
get_env(APP_ENV, 'development');
```

This function takes two arguments. First is the `environment` variable name and second is the `default` value in case it does not find that environment variable.

<p class="tip">Tip: You only need to change the environment specific configuration items and not the whole configuration.</p>

<p class="tip">Tip: Ideally you should not commit your environment specific config files to your version control system.</p>