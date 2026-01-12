# Lightpack Background Jobs: Complete Guide

Ideally, a time consuming job should be performed behind the scenes out of the main HTTP request context. For example, sending email to a user blocks the application until the processing finishes and this may provide a bad experience to your application users. 

What if you could perform time consuming tasks, such as sending emails, in the background without blocking the actual request? 

**Welcome to background job processing.**

While there are highly capable solutions available like **RabbitMQ**, **ZeroMQ**, **ActiveMQ**, **RocketMQ**, and many others, `Lightpack` provides background jobs processing capabilities that is super easy to use and understand. 

Although `Lightpack` will solve background jobs processing needs for most of the applications, it never aims to be a **full-fledged** message queue broker like those mentioned above.

> **Lightpack Jobs** provides robust, extensible, and developer-friendly background job processing for PHP apps. Supports MySQL/MariaDB, Redis, synchronous, and null engines out of the box.

## Supported Engines
- **database:** MySQL/MariaDB-backed persistent queue
- **redis:** High-performance, production-grade queue (sorted sets, atomic ops, delayed jobs)
- **sync:** Executes jobs immediately (for synchronous execution)
- **null:** Discards jobs (for tests/dev)

You can switch the queue engine by altering `JOB_ENGINE` key in **.env** file.

## Database Migration

If using the **database** engine, you need a `jobs` table. 

Create schema migration file:

```cli
php console create:migration --support=jobs
```

Run migration:

```cli
php console migrate:up
```

---

## Creating Jobs

Jobs are PHP classes extending `Lightpack\Jobs\Job` and implementing a `run()` method. 

To create a new job class, fire this command in your terminal from project root:

```terminal
php console create:job SendMail
```

This should have created a `SendMail.php` class file in `app/Jobs` folder. You can implement your job logic in the `run()` method.



```php
use Lightpack\Jobs\Job;

class SendMail extends Job {
    public function run() {
        // Access payload data
        $to = $this->payload['to'];
        $message = $this->payload['message'];
        
        // Your job logic - send email
    }
}
```

## Dispatching Jobs

Once you have implemented your job class, you can **dispatch** them by simply invoking its `dispatch()` method:

```php
(new SendMail)->dispatch();
```

You can optionally pass it an array as payload:

```php
$payload = [
    'to' => 'bob@example.com',
    'message' => 'Hello Bob'
];

(new SendMail)->dispatch($payload);
```

## Advanced Job Features
- **Queue:** Set `$queue` property (default: 'default')
- **Delay:** Set `$delay` property (strtotime string, e.g. '+30 seconds')
- **Attempts:** Set `$attempts` property (default: 1)
- **Retry After:** Set `$retryAfter` property (strtotime string, e.g. '+1 minute')

Example:
```php
class SendMail extends Job {
    protected $queue = 'emails';
    protected $delay = '+1 minute';
    protected $attempts = 3;
    protected $retryAfter = '+10 seconds';
}
```

### Queue

You can specify a queue for a job by setting the `$queue` property:

```php
class SendMail
{
    protected $queue = 'emails';
}
```

### Delay

You can delay job processing by a specified amount of time in two ways:

**Option 1: Property-based (class-level default)**
```php
class SendMail extends Job
{
    protected $delay = '+30 seconds';
}

(new SendMail)->dispatch($payload); // Will be delayed by 30 seconds
```

**Option 2: Method-based (runtime, per-instance)**
```php
// Delay a specific job instance
(new SendMail)->delay('+1 hour')->dispatch($payload);

// Dynamic delays for batch processing
for ($i = 0; $i < 100; $i++) {
    (new SendMail)
        ->delay('+' . ($i * 5) . ' seconds')
        ->dispatch($emails[$i]);
}
```

The `delay()` method accepts any `strtotime()` compatible string (e.g., `'+30 seconds'`, `'+1 hour'`, `'+2 days'`).

### Attempts

You can specify the number of attempts a job should be retried by setting the `$attempts` property:

```php
class SendMail
{
    protected $attempts = 3;
}
```

### Retry After

You can specify the time after which a failed job should be retried by setting the `$retryAfter` property:

```php
class SendMail
{
    protected $retryAfter = '+1 minute';
}
```

## Rate Limiting

Rate limiting controls how many jobs can execute within a time window. Use this when jobs arrive **unpredictably** and you need to respect external service limits.

Lightpack supports rate limiting out of the box. Implement the `rateLimit()` method in your job class to enable rate limiting.

> Rate limiting depends on Lightpack's Cache system. You should configure your cache driver in `.env`. Learn more about cache drivers in the [Caching](caching.md) section.

### Setting Rate Limit

```php
class SendEmailJob extends Job
{
    public function rateLimit(): ?array
    {
        // 10 emails per second
        return ['limit' => 10, 'seconds' => 1];
    }
    
    public function run()
    {
        // Send email logic
    }
}
```

### Supported Time Units

For better readability, you can use multiple time units:

```php
// Seconds (for high-frequency operations)
public function rateLimit(): ?array
{
    return ['limit' => 2, 'seconds' => 1]; // 2 per second
}

// Minutes (common for API calls)
public function rateLimit(): ?array
{
    return ['limit' => 6, 'minutes' => 5]; // 6 login attempts per 5 minutes
}

// Hours (for moderate limits)
public function rateLimit(): ?array
{
    return ['limit' => 100, 'hours' => 1]; // 100 API calls per hour
}

// Days (for daily quotas)
public function rateLimit(): ?array
{
    return ['limit' => 1, 'days' => 1]; // 1 newsletter per day
}
```

**Important:** You must specify a time unit. Omitting it will throw an `InvalidArgumentException`.

### Using Custom Key

Use custom keys to rate limit per user, tenant, or any other dimension:

```php
class SendPaymentReminderJob extends Job
{
    public function rateLimit(): ?array
    {
        $userId = $this->payload['user_id'];
        
        return [
            'limit' => 3,
            'hours' => 1,
            'key' => 'payment-reminder:user:' . $userId
        ];
    }
    
    public function run()
    {
        // Send payment reminder to specific user
    }
}
```

This ensures each user can receive max 3 payment reminders per hour, independently.

### Conditional Rate Limiting

Skip rate limiting based on conditions:

```php
class SendEmailJob extends Job
{
    public function rateLimit(): ?array
    {
        // No rate limit for admin users
        if ($this->payload['is_admin'] ?? false) {
            return null;
        }
        
        // otherwise limit to 10 attempts per minute
        return ['limit' => 10, 'minutes' => 1];
    }
}
```

### Jitter: Preventing Thundering Herd

When multiple jobs are rate-limited simultaneously, they may all retry at the same time, causing a spike in queue processing. Lightpack automatically adds **jitter** (random delay variation) to prevent this "thundering herd" problem.

**Default Behavior:**
- 20% jitter is added automatically to all rate-limited job delays
- Example: 60-second window → jobs retry between 60-72 seconds
- Spreads retry load across time instead of all at once

**How It Works:**
```php
class SendEmailJob extends Job
{
    public function rateLimit(): ?array
    {
        return ['limit' => 14, 'seconds' => 1];
        // Jobs retry after 1.0-1.2 seconds (20% jitter)
    }
}
```

**Disabling Jitter:**
```php
public function rateLimit(): ?array
{
    return [
        'limit' => 14,
        'seconds' => 1,
        'jitter' => 0, // No jitter - exact 1 second delay
    ];
}
```

**Custom Jitter:**
```php
public function rateLimit(): ?array
{
    return [
        'limit' => 14,
        'seconds' => 1,
        'jitter' => 0.5, // 50% jitter - retry between 1.0-1.5 seconds
    ];
}
```

**When Jitter Helps:**
- ✅ Multiple workers processing jobs
- ✅ High-concurrency scenarios (many jobs rate-limited at once)
- ✅ Prevents queue spikes
- ✅ Smoother resource utilization

**When to Disable Jitter:**
- ❌ Single worker setups (no thundering herd possible)
- ❌ Testing/debugging (need deterministic timing)
- ❌ Time-sensitive jobs (need exact retry timing)

### Additional Notes

It is important to understand few of the nuances of rate limiting. Below we document some detailed explanations to help you make informed decisions.


#### Rate Limiting and Attempts Counter

Rate-limited jobs DO increment the attempts counter. This is an important design decision:

- **Rate limiting** = Waiting for API quota/slots → **increments attempts**
- **Job failure** = Exception thrown during execution → **increments attempts**

**Why both increment attempts:**
- Prevents infinite loops if jobs are perpetually rate-limited
- Natural protection against misconfigured rate limits
- Jobs eventually fail rather than cycling forever

**Example:**
```php
class SendEmailJob extends Job
{
    protected $attempts = 10; // Set higher for rate-limited jobs
    
    public function rateLimit(): ?array
    {
        return ['limit' => 2, 'seconds' => 1];
    }
    
    public function run()
    {
        // Send email
    }
}
```

**Scenario:**
- Dispatch 10 emails with `$attempts = 3`
- Jobs 1-2 execute immediately (attempts: 0)
- Jobs 3-10 are rate-limited and released (attempts: 1)
- After 1 second, jobs 3-4 execute (attempts: 1)
- Jobs 5-10 rate-limited again (attempts: 2)
- After 1 second, jobs 5-6 execute (attempts: 2)
- Jobs 7-10 rate-limited again (attempts: 3)
- **Jobs 7-10 fail permanently** (max attempts reached)

**Best Practices:**
- Set higher `$attempts` for rate-limited jobs (e.g., 10-20 instead of 3)
- Monitor rate-limited jobs to tune limits appropriately
- Consider if rate limiting is the right solution for your use case

#### When to Use Rate Limiting vs Manual Delays

Rate limiting is not always the best solution. Below are some scenarios to help you decide when to use rate limiting and when to use manual delays.

**Use Rate Limiting When:**
- ✅ Jobs arrive **unpredictably** (user signups, webhook events, form submissions)
- ✅ You **don't control** when jobs are dispatched
- ✅ External API has **strict limits** you must respect
- ✅ Need **per-user or per-tenant** throttling

**Use Manual Delays (`delay()` method) When:**
- ✅ You **know the volume** upfront (batch processing, cron jobs)
- ✅ You **control dispatch timing** (scheduled tasks)
- ✅ Want to **spread load** evenly over time

**Example Decision:**
```php
// ❌ BAD: Batch processing 1000 emails with rate limiting
for ($i = 0; $i < 1000; $i++) {
    (new SendEmailJob)->dispatch($emails[$i]);
    // Rate limiting will cause many to fail after max attempts
}

// ✅ GOOD: Batch processing with manual delays
for ($i = 0; $i < 1000; $i++) {
    (new SendEmailJob)
        ->delay('+' . ($i * 2) . ' seconds') // 2 seconds apart
        ->dispatch($emails[$i]);
}

// ✅ GOOD: User-triggered emails with rate limiting
class SendVerificationEmailJob extends Job
{
    public function rateLimit(): ?array
    {
        return ['limit' => 100, 'minutes' => 1]; // API limit
    }
}
// Users trigger this unpredictably - rate limiting handles it
```

#### Batch Processing with Manual Delays

For batch processing where you know the volume upfront, use the `delay()` method instead of rate limiting:

**Calculating Delays:**
```
If API allows X requests per Y seconds:
Delay between jobs = Y / X seconds

Examples:
- 100 per minute → 60/100 = 0.6 seconds apart
- 10 per second → 1/10 = 0.1 seconds apart
- 1000 per hour → 3600/1000 = 3.6 seconds apart
```

**Implementation:**
```php
// Example: Email provider allows 100 emails per minute
// Calculation: 60 seconds / 100 emails = 0.6 seconds per email

$delayPerEmail = 60 / 100; // 0.6 seconds

for ($i = 0; $i < 1000; $i++) {
    (new SendEmailJob)
        ->delay('+' . ($i * $delayPerEmail) . ' seconds')
        ->dispatch($emails[$i]);
}

// Job 0: dispatches immediately
// Job 1: dispatches after 0.6 seconds
// Job 2: dispatches after 1.2 seconds
// Job 3: dispatches after 1.8 seconds
// ... and so on
```

**Why This is Better for Batch Processing:**
- ✅ No attempts counter wasted on rate limiting
- ✅ Predictable execution timeline
- ✅ Efficient - jobs execute exactly when scheduled
- ✅ No risk of jobs failing due to rate limit cycles

#### Real-World Rate Limiting Examples

**Example 1: User Verification Emails (unpredictable, API-limited)**
```php
class SendVerificationEmailJob extends Job
{
    public function rateLimit(): ?array
    {
        // Email provider allows 100 emails per minute
        return ['limit' => 100, 'minutes' => 1];
    }
    
    public function run()
    {
        // Users sign up unpredictably throughout the day
        // Rate limiting ensures we never exceed API limits
    }
}
```

**Example 2: SMS OTP (user-triggered, cost control)**
```php
class SendOtpSmsJob extends Job
{
    public function rateLimit(): ?array
    {
        // SMS provider allows 10 per second
        return ['limit' => 10, 'seconds' => 1];
    }
    
    public function run()
    {
        // Users request OTP codes unpredictably
        // Rate limiting prevents exceeding SMS provider limits
    }
}
```

**Example 3: Webhook Delivery (event-driven, external service)**
```php
class DeliverWebhookJob extends Job
{
    public function rateLimit(): ?array
    {
        $webhookUrl = $this->payload['webhook_url'];
        
        // Limit per webhook endpoint to avoid overwhelming recipient
        return [
            'limit' => 50,
            'minutes' => 1,
            'key' => 'webhook:' . md5($webhookUrl)
        ];
    }
    
    public function run()
    {
        // Events trigger webhooks unpredictably
        // Rate limiting protects recipient servers
    }
}
```

**Example 4: Third-Party API Calls (strict API limits)**
```php
class FetchDataFromApiJob extends Job
{
    public function rateLimit(): ?array
    {
        // External API allows 1000 requests per hour
        return ['limit' => 1000, 'hours' => 1];
    }
    
    public function run()
    {
        // Various parts of app trigger API calls
        // Rate limiting ensures we stay within quota
    }
}
```

## Processing Jobs

Once you have dispatched your job it's time to run them. Fire this command from the terminal in your project root:

```terminal
php console jobs:run
```

This will hang your terminal prompt and will wait for any jobs to process. If a job is processed successfully or failed, you should see a terminal message accordingly.

### Worker Options
- `--sleep=N` (default 5): Seconds to sleep between polling
- `--queue=emails,default`: Comma-separated queue names (note: singular)
- `--cooldown=N` (default 0): Total runtime in seconds before worker stops (0 = unlimited)

**Cooldown Explained:**
Cooldown is the **total runtime** (not idle time). After running for the specified seconds, the worker stops gracefully. This is useful for:
- Preventing memory leaks by restarting workers periodically
- Picking up new code after deployment (worker restarts with fresh code)
- Works with Supervisor's auto-restart feature

Example:
```cli
# Worker runs for 10 minutes of total runtime, then stops (Supervisor will restart it)
php console jobs:run --sleep=2 --queue=emails,default --cooldown=600
```

### Signal Handling
The worker supports UNIX signals for graceful shutdown and reload.

## Custom Hooks

To run custom logic after a job succeeds or fails (after all retries are exhausted), implement `onSuccess()` and/or `onFailure()` in your job class:

```php
class SendMail extends Job {
    public function run() {
        // ... job logic ...
    }

    public function onSuccess() {
        // Called after successful processing
    }

    public function onFailure() {
        // Called after all attempts fail
    }
}
```

The framework will call these methods automatically if they exist.

## Retrying Failed Jobs

When jobs fail after exhausting all retry attempts, they are marked as **failed** in the queue. You can retry these failed jobs using the `jobs:retry` command.

### Retry All Failed Jobs

```cli
php console jobs:retry
```

This will reset all failed jobs and queue them for processing again.

### Retry a Specific Failed Job

```cli
php console jobs:retry <job_id>
```

Example:
```cli
# For database engine (numeric IDs)
php console jobs:retry 123

# For Redis engine (string IDs)
php console jobs:retry job_abc123xyz
```

### Retry Failed Jobs from a Specific Queue

```cli
php console jobs:retry --queue=<queue_name>
```

Example:
```cli
# Retry all failed jobs from the 'emails' queue
php console jobs:retry --queue=emails

# Retry all failed jobs from the 'notifications' queue
php console jobs:retry --queue=notifications
```

## Production

In **production** environment, you should run and monitor job processing by using a process monitoring solution like **supervisor**.

First, you will have to install `supervisor`:

```terminal
sudo apt-get install supervisor
```

Let us assume that your project root path is `/var/www/lightpack-app`.

Create a file named `lightpack-worker.conf` in `/etc/supervisor/conf.d` directory with following contents:

```text
[program:lightpack-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/lightpack-app/console jobs:run --cooldown=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=www-data
numprocs=4
redirect_stderr=true
stdout_logfile=/var/www/lightpack-app/worker.log
stopwaitsecs=60
```

**Configuration Notes:**
- `--cooldown=3600`: Workers stop after 1 hour of total runtime (not idle time), then Supervisor restarts them. This prevents memory leaks and picks up new code on restart.
- `autorestart=true`: Supervisor automatically restarts workers after they stop
- `stopwaitsecs=60`: Gives workers 60 seconds to finish current job before force-killing
- `numprocs=4`: Runs 4 worker processes in parallel

Finally, fire these commands to start supervisor:

```terminal
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start lightpack-worker:*
```

---