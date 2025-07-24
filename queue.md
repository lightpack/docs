# Lightpack Background Jobs: Complete Guide

Ideally, a time consuming job should be performed behind the scenes out of the main HTTP request context. For example, sending email to a user blocks the application untill the processing finishes and this may provide a bad experience to your application users. 

What if you could perform time consuming tasks, such as sending emails, in the background without blocking the actual request? 

**Welcome to background job processing.**

While there are highly capable solutions available like **RabbitMQ**, **ZeroMQ**, **ActiveMQ**, **RocketMQ**, and many others, `Lightpack` provides background jobs processing capabilities that is super easy to use and understand. 

Although `Lightpack` will solve background jobs processing needs for most of the applications, it never aims to be a **full-fledged** message queue broker like those mentioned above.

> **Lightpack Jobs** provides robust, extensible, and developer-friendly background job processing for PHP apps. Supports MySQL/MariaDB, Redis, synchronous, and null engines out of the box.

## Supported Engines
- **Database:** MySQL/MariaDB-backed persistent queue
- **Redis:** High-performance, production-grade queue (sorted sets, atomic ops, delayed jobs)
- **Sync:** Executes jobs immediately (for synchronous execution)
- **Null:** Discards jobs (for tests/dev)

View `config/jobs.php` for desired job queue related configurations.

## Jobs Table Migration (Database Engine)

If using the **database** engine, you need a `jobs` table. 

**Create the migration:**

```cli
php console create_table_jobs
```

**Update the migration logic:**

```php
return new class extends Migration
{
    public function up(): void
    {
        $this->create('jobs', function (Table $table) {
            $table->id();
            $table->varchar('handler', 255);
            $table->varchar('queue', 55)->index();
            $table->text('payload');
            $table->varchar('status', 55)->index();
            $table->column('attempts')->type('int');
            $table->column('exception')->type('longtext')->nullable();
            $table->createdAt();
            $table->datetime('scheduled_at')->default('CURRENT_TIMESTAMP')->index();
            $table->datetime('failed_at')->nullable();
        });
    }

    public function down(): void
    {
        $this->drop('jobs');
    }
};
```

**Run the migration:**

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
        // Your job logic, e.g. send email using $this->payload
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

You can delay job processing by a specified amount of time by setting the `$delay` property:

```php
class SendMail
{
    protected $delay = '+30 seconds';
}
```

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

## Processing Jobs

Once you have dispatched your job its time to run them. Fire this command from the terminal in your project root:

```terminal
php console process:jobs
```

This will hang your terminal prompt and will wait for any jobs to process. If a job is processed successfully or faile, you should see a terminal message accordingly.

### Worker Options
- `--sleep=N` (default 5): Seconds to sleep between polling
- `--queues=emails,default`: Comma-separated queue names
- `--cooldown=N`: Max seconds to run before exiting

Example:
```cli
php console process:jobs --sleep=2 --queues=emails,default --cooldown=600
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

## Production

In  **production** environment, you should run and monitor job processing by using a process monitoring solution like **supervisor**.

First, you will have to install `supervisor`:

```terminal
sudo apt-get install supervisor
```

Let us assume that your project root path is `/var/www/lightpack-app`.

Create a file named `lightpack-worker.conf` in `/etc/supervisor/conf.d` directory with following contents:

```text
[program:lightshop-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/lightpack-app/console process:jobs
autostart=true
autorestart=true
stopasgroup=true
user=www-data
numprocs=4
redirect_stderr=true
stdout_logfile=/var/www/lightpack-app/worker.log
```

Finally, fire these commands to start supervisor:

```terminal
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start lightshop-worker:*
```

---