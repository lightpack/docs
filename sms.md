
# Lightpack SMS

> **Lightpack SMS** offers a clean, extensible, and framework-native way to send SMS messages via multiple providers.

## Supported Providers

Out of the box:

- **TwilioProvider:** Real SMS via Twilio API.
- **LogSmsProvider:** Writes SMS to logs (for dev/test).
- **NullSmsProvider:** Discards SMS (for test/mocking).

For **Twilio** support, you will need to install its PHP SDK:  
  `composer require twilio/sdk`

View the config file `config/sms.php` to see available configuration options for SMS feature.

## Sending an SMS

```php
sms()->send('+1234567890', 'Hello from Lightpack!');
```

## Extending with Custom Providers

Create your own provider by implementing `SmsProviderInterface`:

```php
namespace App\Sms\Providers;

use Lightpack\Sms\SmsProviderInterface;

class MyCustomProvider implements SmsProviderInterface
{
    public function send(string $phone, string $message, array $options = []): bool
    {
        // Your custom SMS logic
        return true;
    }
}
```

Add to your config:

```php
'drivers' => [
    'mycustom' => [
        'provider' => App\Sms\Providers\MyCustomProvider::class,
        // ...your config
    ],
]
```

---

## Error Handling & Logging

- **TwilioProvider:** Catches exceptions, logs errors, returns `false` on failure.
- **LogSmsProvider:** Always returns `true`, logs all messages.
- **NullSmsProvider:** Always returns `true`, does nothing.
- **Best practice:** Always check the boolean return value of `send()` and act accordingly.

---


## FAQ & Troubleshooting

**Q: How do I switch providers?**  
A: Change the `default` driver in your config or instantiate the desired provider.

**Q: Can I queue SMS for async sending?**  
A: Yes! Use Lightpack’s jobs system to queue jobs that call `sms()->send()`.

**Q: How do I test SMS sending?**  
A: Use LogSmsProvider or NullSmsProvider in your test environment.

**Q: What if Twilio credentials are invalid?**  
A: TwilioProvider will log the error and return `false`.

---