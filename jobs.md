# Background Jobs

Ideally some time consuming job should be performed behind the scenes out of the main HTTP request context. For example, sending email to a user blocks the application untill the processing finishes and this may provide a bad experience to your application users. 

What if you could perform time consuming tasks, such as sending emails, in the background without blocking the actual request? 

**Welcome to background job processing.**

While there are highly capable solutions available like **RabbitMQ**, **ZeroMQ**, **ActiveMQ**, and many others, `Lightpack` provides background jobs processing capabilities that is super easy to use and understand. 

Although `Lightpack` will solve background jobs processing needs for most of the applications, it never aims to be a **full-fledged** message queue broker like those mentioned above.

`