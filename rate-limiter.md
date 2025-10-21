# Rate Limiting

Rate limiting (throttling) helps control access rates to any resource - from HTTP routes to API calls to background jobs. Lightpack provides a simple, efficient utility for **rate limiting** actions—such as login attempts, API requests, or any operation you want to restrict to a certain number of times within a time window.

## Use Cases

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

## Checking Status

### Get Current Hits

```php
$hits = limiter()->getHits('login:user:123'); // returns int|null

if ($hits !== null) {
    echo "You've made {$hits} attempts";
}
```

### Get Remaining Attempts

```php
$remaining = limiter()->getRemaining('login:user:123', 5);
echo "You have {$remaining} attempts remaining";
```

## How It Works

- On the first attempt, the window is started and the hit count is set to 1.
- Each subsequent allowed attempt increments the count, but the window's TTL is preserved.
- If the max is reached, `attempt()` returns `false` until the window expires.
- Once expired, the count resets automatically.

### Example: IP-Based Rate Limiting

```php
// Allow 3 requests per 10 seconds from an IP
$key = 'ip:' . request()->ip();

if (!limiter()->attempt($key, 3, 10)) {
    return response('Too many requests', 429);
}

// Process request
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
```

### `getRemaining(string $key, int $max): int`
Get remaining attempts for a key. Returns `0` if rate limited, `$max` if no attempts yet.

```php
$remaining = limiter()->getRemaining('login:user:123', 5);

if ($remaining > 0) {
    echo "You have {$remaining} attempts remaining";
} else {
    echo "Rate limit exceeded. Please try again later.";
}
```

---

## Best Practices

### 1. Use Descriptive Keys

```php
// Good
'login:email:user@example.com'
'api:user:123:endpoint:/posts'
'upload:user:456:type:image'

// Bad
'user123'
'limit'
'check'
```

### 2. Choose Appropriate Windows

```php
// Login attempts: 5 per 5 minutes
limiter()->attempt('login:' . $email, 5, 300);

// API calls: 1000 per hour
limiter()->attempt('api:' . $userId, 1000, 3600);

// Sensitive operations: 3 per day
limiter()->attempt('delete:' . $userId, 3, 86400);

// Real-time operations: 10 per second
limiter()->attempt('websocket:' . $userId, 10, 1);
```

### 3. Provide User Feedback

```php
$key = 'action:user:' . $userId;
$max = 10;

if (!limiter()->attempt($key, $max, 3600)) {
    $hits = limiter()->getHits($key);
    $remaining = limiter()->getRemaining($key, $max);
    
    return response()->json([
        'error' => 'Rate limit exceeded',
        'limit' => $max,
        'current' => $hits,
        'remaining' => $remaining,
        'reset_in' => '1 hour'
    ], 429);
}
```

### 4. Different Limits for Different Users

```php
$user = auth()->user();

// Premium users get higher limits
$max = $user->isPremium() ? 10000 : 1000;
$window = 3600; // 1 hour

if (!limiter()->attempt('api:user:' . $user->id, $max, $window)) {
    return response('Rate limit exceeded', 429);
}
```

---

## Distributed Systems

When running multiple servers/workers, the rate limiter works correctly because it uses a shared cache backend:

**With Redis (Recommended for Production):**
```php
// config/cache.php
return [
    'driver' => 'redis',
    // ...
];
```

**How it works:**
- All servers share the same Redis instance
- Rate limits are enforced globally across all servers
- No coordination needed between servers

**Example: 3 servers, 100 req/min limit**
```
Server 1: User makes 40 requests → Redis counter = 40
Server 2: User makes 35 requests → Redis counter = 75
Server 3: User makes 25 requests → Redis counter = 100
Server 1: User makes 1 request  → BLOCKED (limit reached)
```

---