# Lightpack Webhook Receiver

A robust, flexible, and production-ready webhook processing framework for PHP/Lightpack projects. Easily receive, verify, store, and process webhooks from any provider (Stripe, GitHub, Slack, etc.)

---

## Features
- **Provider-agnostic:** Plug in any webhook provider with simple config.
- **Signature verification:** Securely verify authenticity of incoming requests.
- **Idempotency:** Prevent duplicate event processing using provider+event ID.
- **Payload & header storage:** Audit, debug, and replay all webhook events.
- **Status tracking:** Track event lifecycle (`pending`, `processed`, `failed`).
- **Extensible handlers:** Easily add provider-specific logic via custom classes.

---


## Event Storage Schema

| Column       | Type     | Description                             |
|--------------|----------|-----------------------------------------|
| id           | bigint   | Primary key                             |
| provider     | varchar  | Provider name (e.g., 'stripe')          |
| event_id     | varchar  | Unique event ID (nullable)              |
| payload      | text     | Full event payload (array, JSON-cast)   |
| headers      | text     | All request headers (array, JSON-cast)  |
| status       | varchar  | Event status (`pending`, `processed`, `failed`) |
| received_at  | datetime | Timestamp of event receipt              |

---

## Migration

Run this command to generate a migration file:

```cli
php console create:migration create_table_webhook_events
```

Use the following code for the up() and down() methods:

```php
public function up(): void
{
    $this->create('webhook_events', function (Table $table) {
        $table->id();
        $table->varchar('provider', 64);
        $table->varchar('event_id', 128)->nullable();
        $table->column('payload')->type('text');
        $table->column('headers')->type('text')->nullable();
        $table->varchar('status', 32)->default('pending');
        $table->datetime('received_at')->default('CURRENT_TIMESTAMP');
        $table->unique(['provider', 'event_id']);
    });
}

public function down(): void
{
    $this->drop('webhook_events');
}
```

---

## Configuration

In your config (e.g., `config/webhook.php`):

```php
return [
    // example: stripe
    'stripe' => [
        'secret' => 'your-stripe-webhook-secret',
        'algo' => 'hmac', // or 'static' for static secrets
        'id' => 'id', // field in payload for event ID
        'handler' => App\Webhooks\StripeWebhookHandler::class,
    ],
    // example: github
    'github' => [
        'secret' => 'your-github-secret',
        'algo' => 'hmac',
        'id' => 'delivery', // GitHub's event ID is in a header
        'handler' => App\Webhooks\GitHubWebhookHandler::class,
    ],
];
```

- `secret`: Your provider's webhook signing secret.
- `algo`: Signature verification algorithm (`hmac` or `static`).
- `id`: Field name in payload (or header) for unique event ID.
- `handler`: Custom handler class for provider-specific logic.

---

## Usage



1. **Define your webhook route:**

Add following route definition in `routes/web.php` file. The corresponding `WebhookController` is already implemented by the framework.

```php
route()->post('/webhook/:provider', WebhookController::class, 'handle');
```

2. **Implement custom handlers as needed:**

`app/Webhooks/StripeWebhookHandler.php`

```php
use Lightpack\Webhook\BaseWebhookHandler;

class StripeWebhookHandler extends BaseWebhookHandler
{
    public function verifySignature(): static
    {
        // Stripe-specific signature logic
        return $this;
    }

    public function handle(): Response
    {
        // Custom event processing
        return response()->json(['status' => 'ok'], 200);
    }
}
```


**Example: Custom Handler for Header-Based Event ID**

In cases where the webhook receiver gets the event id in request header, you need to override `storeEvent()` method.

```php
class GitHubWebhookHandler extends BaseWebhookHandler
{
    public function storeEvent(?string $eventId): WebhookEvent
    {
        // GitHub's delivery ID is in a header
        $eventId = $this->request->header('X-GitHub-Delivery');

        return parent::storeEvent($eventId);
    }
}
```

---

## Security & Idempotency

- **Signature verification**: All requests are verified using the configured secret and algorithm before processing.
- **Idempotency**: Duplicate events (same provider + event ID) are ignored, ensuring safe retries.
- **Missing event ID**: If a provider does not send an event ID, events are stored with `event_id = null` and idempotency is not enforced.

---

## Best Practices & Tips

- **Always specify the correct event ID field** for each provider in your config.
- **For providers with event IDs in headers** (e.g., GitHub), extend your handler to extract the ID from headers.
- **If a provider does not send an event ID**, be aware that idempotency is not enforced—duplicate events may be processed.
- **Extend BaseWebhookHandler** for provider-specific signature verification and event processing.
- **Log and monitor** webhook event statuses for production visibility.

---

## Notes

**What if my provider doesn't send a unique event ID?**

The event will be stored with `event_id = null`, and idempotency will not be enforced. You may want to generate your own unique key if needed.

**Can I process multiple providers with different logic?**

Yes! Simply specify a different handler class per provider in your config.

**How do I debug webhook failures?**

All events (including failed ones) are stored with full payload and headers. Check the `webhook_events` table for details.

---