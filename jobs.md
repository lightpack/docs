# Background Jobs

Ideally some time consuming job should be performed behind the scenes out of the main HTTP request context. For example, sending email to a user blocks the application untill the processing finishes and this may provide a bad experience to your application users. 

What if you could perform time consuming tasks, such as sending emails, in the background without blocking the actual request? 

*Welcome to background job processing.*

While there are highly capable solutions available like **RabbitMQ**, **ZeroMQ**, **ActiveMQ**, **RocketMQ**, and many others, `Lightpack` provides background jobs processing capabilities that is super easy to use and understand. 

Although `Lightpack` will solve background jobs processing needs for most of the applications, it never aims to be a **full-fledged** message queue broker like those mentioned above.

<p class="tip">Lightpack provides a <b>MySQL/MariaDB</b> powered background job processor. Although, some of you might be concerned with its performance, in my experience it has worked fine and scales really well for most of the application needs out there.</p>

<p class="tip">Meanwhile, a <b>beanstalkd</b> powered job processor is in progress which will be integrated in the core framework once done.</p>

## Jobs Table

Currently, jobs processing is powered by `MySQL/MariaDB` database. So you will need to migrate a `jobs` table in your app database. So please create this table in your database:

```SQL
CREATE TABLE jobs (
    id int NOT NULL AUTO_INCREMENT,
    name varchar(55) COLLATE utf8_unicode_ci NOT NULL,
    payload text COLLATE utf8_unicode_ci NOT NULL,
    status varchar(55) COLLATE utf8_unicode_ci NOT NULL,
    scheduled_at datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
    created_at datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    KEY status (status,scheduled_at)
) ENGINE=InnoDB
```

## Creating Jobs

Jobs are simply classes that implement `execute()` method. To create a new job class, fire this command in your terminal from project root:

```terminal
php lucy create:job SendMail
```

This should have created a `SendMail.php` class file in `app/Jobs` folder. You can implement your job logic in the `execute()` method.

## Dispatching Jobs

Once you have implemented your job class, you can **dispatch** them by simply invoking its `dispatch()` method passing it a payload as an array as shown:

```php
(new SendMail)->dispatch();
```

This will push the job into the database for processing in background and will not block the request.

## Processing Jobs

Once you have dispatched your jobm its time to run them. Fire this command from the terminal in your project root:

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