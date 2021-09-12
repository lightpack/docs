# Sending Emails

**Lightpack** facilitates sending emails using any SMTP service provider in a very friendly manner. You can easily send emails in plain `Text` or rich `HTML` format.

The first step is to **configure** your SMTP service provider. 

## Configuring SMTP

Open your [env.php](/environments) file and adjust the SMTP credentials:

```php
/**
 * Mail settings.
 */

'MAIL_DRIVER' => 'smtp',
'MAIL_HOST' => 'smtp.mailtrap.io',
'MAIL_PORT' => 587,
'MAIL_ENCRYPTION' => 'tls',
'MAIL_USERNAME' => null,
'MAIL_PASSWORD' => null,
'MAIL_FROM_ADDRESS' => 'lightpack@example.com',
'MAIL_FROM_NAME' => 'Lightpack',
```

## Creating Mail Class

Once you have setup your `SMTP` credentials, you should now create a mail class. To create a new mail class, fire this command in your terminal from project root:

```terminal
php lucy create:mail TestMail
```

This should have created a `TestMail.php` file in **app/Mails** folder. You can write your mail logic in `execute()` method. Here is a minimal example of sending an email:

```php
public function execute(array $payload = [])
{
    $this->from('joe@example.com')
        ->to('bob@example.com')
        ->subject('Hello Bob')
        ->body('Welcome to Lightpack')
        ->send();
}
```

> Lightpack depends on [PHPMailer](https://github.com/PHPMailer/PHPMailer) for mail sending facility and all the mail classes extend this library. So you can utilize all the possible methods in your mail class as documented by **PHPMailer** project.</p>

## Available Methods

Below we document the most important methods that will help you send emails.

### from()

Use this method to configure the `sender` email address and name.

```php
$this->from('joe@example.com', 'Bob');
```

### to()

Use this method to configure the `recipient` email address and name.

```php
$this->to('joe@example.com', 'Joe');
```




