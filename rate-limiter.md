# Rate Limiting

Rate limiting (throttling) helps control access rates to any resource - from HTTP routes to API calls to background jobs. Lightpack provides a simple, efficient utility for **rate limiting** actions—such as login attempts, API requests, or any operation you want to restrict to a certain number of times within a time window.. Use cases include:

- API request throttling
- Login attempt limits
- Job processing controls
- Resource usage limits
- Database write throttling

The **rate limiter** uses Lightpack's Cache system, supporting multiple drivers:
- Database (default)
- Redis
- File
- Array (for testing)
- Null (for disabled limiting)

> The configured cache driver becomes the backend for rate limiter. You do not need to separately configure the rate limiter.

## Usage

You may create an instance of `Limiter` class:

```php
$limiter = new Lightpack\Utils\Limiter;
```

Or simply call the utility function `limiter()` which returns **Limiter** class instance.

Call the `attempt()` method of **Limiter** class instance to rate limit a block of code or an operation to be executed.

`attempt(string $key, int $max, int $seconds)`

- The **first** argument is a unique key (e.g., user ID, IP address, or action).
- The **second** argument is the max allowed attempts.
- The **third** argument is the window (in seconds).

**Example:** Rate limit user login to max 5 attempts per 60 seconds

```php
$loginAllowed = limiter()->attempt('login:user:123', 5, 60);
```

```php
if($loginAllowed) {
    // Allowed: perform the action
} else {
    // Rate limit exceeded: block or notify
}
```

**Key Generation**

Keys can be anything unique to what you're limiting:

```php
// Examples
'ip:127.0.0.1'           // IP-based
'user:123'               // User-based
'api:endpoint:/users'     // API endpoint
'upload:user:123'        // Resource usage
'jobs:processor:1'       // Job processing
```

## Checking Attempts

You can check how many attempts have been made in the current window:

```php
$hits = limiter()->getHits('login:user:123'); // returns int|null

if ($hits !== null) {
    $remaining = 5 - $hits; // If max is 5
    echo "You have {$remaining} attempts remaining";
}
```

## How It Works

- On the first attempt, the window is started and the hit count is set to 1.
- Each subsequent allowed attempt increments the count, but the window's TTL is preserved.
- If the max is reached, `attempt()` returns `false` until the window expires.
- Once expired, the count resets automatically.

## Examples

```php
// Allow 3 requests per 10 seconds from an IP
if ($limiter->attempt(request()->ip(), 3, 10)) {
    // process request
} else {
    // too many requests
}

```

## HTTP Rate Limiting

You can rate limit HTTP routes using the rate limit filter.

**Syntax:** `limit:max,minutes`
- `max` - Maximum requests allowed
- `minutes` - Time window in minutes

```php
// Allow 60 requests per 1 minute
route()
    ->get('/api', ApiController::class)
    ->filter(['limit:60,1']);

// Allow 1000 requests per 60 minutes (1 hour)
route()->group(['filter' => ['limit:1000,60']], function() {
    route()->get('/api/users', 'UserController::class');
    route()->get('/api/posts', 'PostController::class');
});

// Allow 10 uploads per 60 minutes
route()
    ->post('/upload', UploadController::class)
    ->filter(['auth', 'limit:10,60']);
```

**When rate limited, clients receive:**

```http
HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1616721340
Retry-After: 60

Too many requests. Please try again in 1 minute.
```

---

## Job Rate Limiting

Rate limit background jobs to comply with external API limits.

**Scenario:** Email provider allows 2 requests per second, 50 emails per request.

```php
class SendEmailBatch extends Job
{
    protected $attempts = 5;
    protected $retryAfter = '+30 seconds';

    public function run()
    {
        $batchId = $this->payload['batch_id'];
        
        // Rate limit: 2 requests per second
        // Key 'email-api' is shared across all workers
        if (limiter()->attempt('email-api', 2, 1) === false) {
            // Rate limit hit - throw to retry later
            throw new \Exception('Rate limit reached');
        }
        
        try {
            // Send batch of 50 emails to provider
            $this->sendEmailBatch($batchId);
        } catch (\Exception $e) {
            // Job will retry after 30 seconds
            throw $e;
        }
    }
}
```

**How it works:**
1. Multiple workers process email batches concurrently
2. Each worker checks rate limit before calling API
3. If limit reached (2 req/sec), job throws exception
4. Job automatically retries after 30 seconds
5. Rate limit resets after 1 second

**Result:** Perfect compliance with API limits across distributed workers!

---

## Available Methods

### `attempt(string $key, int $max, int $seconds): bool`
Attempt an action. Returns `true` if allowed, `false` if rate limited.

```php
if (limiter()->attempt('action:user:123', 5, 60)) {
    // Allowed
} else {
    // Rate limited
}
```

### `getHits(string $key): ?int`
Get current hit count for a key. Returns `null` if key doesn't exist.

```php
$hits = limiter()->getHits('action:user:123');
$remaining = 5 - ($hits ?? 0);
```

---