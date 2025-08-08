# Providers

Suppose in one of your controllers you want to send an email and for that you create a **Mailer** library that requires some initial setup before you start sending mails.

```php
class ReportController
{
    public function sendReport()
    {
        // Configure mailer service provider
        $mailer = new Mailer('smtp.example.org', 25);
        $mailer->setUsername('your username');
        $mailer->setPassword('your password');
        
        // Send mail
        $mailer->sendMessage([
            'to'      => 'analyst@example.com',
            'from'    => 'admin@example.com',
            'body'     => 'Here is your report',
            'subject' => 'Latest financial report',
        ]);
    }
}
```

What is wrong with the above code? **Nothing if it actually works.**

But imagine there a number of controller methods where you use this **Mailer** service class. This will soon become a nightmare to maintain such code because you will keep configuring **Mailer** instance before sending mails. **Right?**

What if you could configure your **Mailer** service once and keep using that instance wherever you wanted in your application?

Although there are a couple of possible solutions to do that, `Lightpack` supports the concept of **provider** which are classes to configure your **services**.

<p class="tip"><b>Providers are classes where you register your services in IoC container for easy access.</b></p>

## Creating Provider

From your terminal you can fire this command to create a `Mailerprovider`.

```terminal
php console create:provider MailerProvider
```

This should have generated `MailerProvider` class in `app/Providers` directory. 

Inside that class you will find a `register()` method:

```php
public function register(Container $container)
{
    $container->register('mailer', function ($container) {
        //
    });
}
```

In this method you can configure `Mailer` service as shown:

```php
public function register(Container $container)
{
    $container->register('mailer', function ($container) {

        $mailer = new Mailer('smtp.example.org', 25);
        $mailer->setUsername('your username');
        $mailer->setPassword('your password');

        return $mailer;

    });
}
```

## Registering Provider

You need to register the provider class in order to access the service.

Open `boot/providers.php` file and just add your provider class at the **end** of the providers array.

```php
<?php

/**
 * ------------------------------------------------------------
 * List service providers here.
 * ------------------------------------------------------------
 */

return [
    'providers' => [
        App\Providers\MailerProvider::class,
    ],
];
```

## Using Provider

Now you can access this service using the alias **mailer** by calling `app('mailer')` function in your controller.

```php
class ReportController
{
    public function sendReport()
    {
        app('mailer')->sendMessage([
            'to'      => 'analyst@example.com',
            'from'    => 'admin@example.com',
            'body'     => 'Here is your report',
            'subject' => 'Latest financial report',
        ]);
    }
}
```

As you can see that in **providers** you bind services in container, so you should read more about [containers](/containers) in Lightpack.