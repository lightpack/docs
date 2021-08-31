# Background Jobs

Ideally some time consuming job should be performed behind the scenes out of the main HTTP request context. For example, sending email to a user blocks the application untill the processing finishes and this may provide a bad experience to your application users. 

What if you could perform time consuming tasks, such as sending emails, in the background without blocking the actual request? 

*Welcome to background job processing.*

While there are highly capable solutions available like **RabbitMQ**, **ZeroMQ**, **ActiveMQ**, **RocketMQ**, and many others, `Lightpack` provides background jobs processing capabilities that is super easy to use and understand. 

Although `Lightpack` will solve background jobs processing needs for most of the applications, it never aims to be a **full-fledged** message queue broker like those mentioned above.

<p class="tip">Lightpack provides a <b>MySQL/MariaDB</b> powered background job processor. Although, some of you might be concerned with its performance, in my experience it has worked fine and scales really well for most of the application needs out there.</p>

<p class="tip">Meanwhile, a <b>beanstalkd</b> powered job processor is in progress which will be integrated in the core framework once done.</p>

## Creating Jobs

Jobs are simply classes that implement `execute()` method. To create a new job class, fire this command in your terminal from project root:

```terminal
php lucy create:job SendMail
```

This should have created a `SendMail.php` class file in `app/Jobs` folder.

