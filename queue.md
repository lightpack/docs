# Background Jobs

Ideally, a time consuming job should be performed behind the scenes out of the main HTTP request context. For example, sending email to a user blocks the application untill the processing finishes and this may provide a bad experience to your application users. 

What if you could perform time consuming tasks, such as sending emails, in the background without blocking the actual request? 

*Welcome to background job processing.*

While there are highly capable solutions available like **RabbitMQ**, **ZeroMQ**, **ActiveMQ**, **RocketMQ**, and many others, `Lightpack` provides background jobs processing capabilities that is super easy to use and understand. 

Although `Lightpack` will solve background jobs processing needs for most of the applications, it never aims to be a **full-fledged** message queue broker like those mentioned above.

<p class="tip">Lightpack provides a <b>MySQL/MariaDB/Redis</b> powered background job processor.</p>

## Jobs Table

Currently, jobs processing is powered by `MySQL/MariaDB` database. So you will need to migrate a `jobs` table in your app database. So please create this table in your database:

```sql
CREATE TABLE jobs (
    id int NOT NULL AUTO_INCREMENT,
    handler varchar(255) NOT NULL,
    queue varchar(55) NOT NULL,
    payload text NOT NULL,
    status varchar(55) NOT NULL,
    attempts int NOT NULL,
    exception longtext NULL,
    created_at datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
    scheduled_at datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
    failed_at datetime NULL,
    PRIMARY KEY (id),
    index status (status),
    index scheduled_at (scheduled_at),
    index queue (queue)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
```

## Creating Jobs

Jobs are simply classes that implement `execute()` method. To create a new job class, fire this command in your terminal from project root:

```terminal
php lucy create:job SendMail
```

This should have created a `SendMail.php` class file in `app/Jobs` folder. You can implement your job logic in the `run()` method.

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

This will push the job into the database for processing in background and will not block the request.

## Processing Jobs

Once you have dispatched your job its time to run them. Fire this command from the terminal in your project root:

```terminal
php lucy process:jobs
```

This will hang your terminal prompt and will wait for any jobs to process. If a job is processed successfully, you should see a terminal message something like this for example:

```terminal
✔ Job processed successfully: 123
```

If the job throws an exception that can be caught and fails, you should see a terminal message something like this for example:

```terminal
✖ Error dispatching job: 123 - Recipient email address is missing
```

**NOTE:** All jobs that failed processing will have status `failed` in the `jobs` table.

## Delaying Jobs

By default, a job is processed as soon as it is available for processing. However, you can delay job processing by a specified amount of time. For that, set the `$delay` property with any `strtotime` compatible string value. 

For example, following job will be processed after a delay of `30 seconds`. 

```php
class SendMail
{
    protected $delay = '+30 seconds';
}
```

## Supervising Jobs

When testing your jobs locally, it fine to inspect them in terminal but in production, the processing of jobs should be **deamonized**. What this means is that the job **worker** should keep running in the background as a system process and in case it stops, it should start automatically.

There are various process monitoring solutions available like **upstart**, **systemd**, **supervisor**. Here is a solution using `supervisor` as job process monitor.

First, you will have to install `supervisor`:

```terminal
sudo apt-get install supervisor
```

Let us assume that your project root path is `/var/www/lightpack-app`.

Create a file named `lightpack-worker.conf` in `/etc/supervisor/conf.d` directory with following contents:

```text
[program:lightshop-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/lightpack-app/lucy process:jobs
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