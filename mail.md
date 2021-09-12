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

## Sending Mail

To send the mail, simply instantiate the call and call its `execute()` method:

```php
(new TestMail)->execute();
```

You can optionally pass it an array as data payload which you can use inside the `execute()` method:

```php
(new TestMail)->execute([
    'to' => 'devs@example.com',
    'from' => 'lightpack@example.com',
]);
```

## HTML Templates

You can create an `HTML` email template inside `app/views` folder and use that as your email body.

For example, create a folder named `mails` inside `app/views` with following two files.

```
./app
    └── views
        └── mails
            ├── welcome.html.php
            ├── welcome.text.php
```

As you can see, we have two files named **welcome.html.php** and **welcome.text.php** because it is recommended to also send plain text version of your email for *non-HTML* compliant inboxes.

### Creating Message

Create your HTML email message markup in **welcome.html.php**.

```php
<!DOCTYPE html>
<html>
    <body>
        <h1>Hello Devs,</h1>
        <p>Welcome to Lightpack PHP web framework.</p>
    </body>
</html>
```

And here is the plain text version in **welcome.text.php**.

```php
Hello Devs, welcome to Lightpack PHP web framework.
```

### Setting View Templates

Now in the mail class, specify those two templates as class properties:

```php
class WelcomeMail extends Mail
{
    protected $this->textView = 'mails/welcome.text';
    
    protected $this->htmlView = 'mails/welcome.html';

    // ...
}
```

**Thats it!!** Now you can simply send the mail:

```php
public function execute(array $payload = [])
{
   $this->from('lightpack@example.com')
        ->to('devs@example.com')
        ->subject('Hello Devs')
        ->send();
}
```

### Passing View Data

In case you need to pass data to your templates, simply set the `data` attribute as an array inside `execute()` method:

```php
public function execute(array $payload = [])
{
    $this->data = [
        'title' => 'Hello Devs',
        'content' => 'Welcome to Lightpack PHP web framework.',
    ];
}
```

Now you can access the data inside your templates with keys as variables:

```php
<!DOCTYPE html>
<html>
    <body>
        <h1><?= $title ?></h1>
        <p><?= $content ?></p>
    </body>
</html>
```

## Available Methods

Below we document the most important methods that will help you send emails.

### from()

Use this method to configure the `sender` email address and name.

```php
$this->from('joe@example.com', 'Bob');
```

**Note:** Use `MAIL_FROM_ADDRESS` and `MAIL_FROM_NAME` to set the global `from` address and name of the email sender. In case when you do not explicitly specify the sender in the mail class using `from()` method, this email will be used.

### to()

Use this method to configure the `recipient` email address and name.

```php
$this->to('joe@example.com', 'Joe');
```

### cc()

Use this method to `cc` a recipient email address and name.

```php
$this->cc('rob@example.com', 'Rob');
$this->cc('john@example.com', 'John');
```

You can `cc` multiple recipients at once by passing it an array of email address and name as **key-value** pairs.

```php
$this->cc([
    'rob@example.com' => 'Rob',
    'john@example.com' => 'John'
]);
```

Note that the name is optional. For example:

```php
$this->cc([
    'rob@example.com'
    'john@example.com'
]);
```

### bcc()

Use this method to `bcc` a recipient email address and name.

```php
$this->bcc('rob@example.com', 'Rob');
$this->bcc('john@example.com', 'John');
```

You can `bcc` multiple recipients at once by passing it an array of email address and name as **key-value** pairs.

```php
$this->bcc([
    'rob@example.com' => 'Rob',
    'john@example.com' => 'John'
]);
```

Note that the name is optional. For example:

```php
$this->bcc([
    'rob@example.com'
    'john@example.com'
]);
```

### replyTo()

Use this method to add a `reply-to` header.

```php
$this->replyTo('support@example.com', 'Support Team');
```

You can `replyTo` multiple recipients at once by passing it an array of email address and name as **key-value** pairs.

```php
$this->replyTo([
    'support1@example.com' => 'Support Team1',
    'support2@example.com' => 'Support Team2'
]);
```

Note that the name is optional. For example:

```php
$this->replyTo([
    'support1@example.com',
    'support2@example.com',
]);
```

### attach()

Use this method to send a file `attachment`.

```php
$this->attach('/path/to/file');
```

You can `attach` multiple files at once by passing it an array of filepath and optional name as **key-value** pairs.

```php
$this->attach([
    '/path/to/file1' => 'filename1',
    '/path/to/file2' => 'filename2',
]);
```

### subject()

Use this method to set the `subject` of the mail.

```php
$this->subject('My Email Subject');
```

### body()

Use this method to set a rich `HTML` email message.

```php
$this->body('<p>Hello Devs</p>');
```

However, it is recommended to create and send [rich HTML](/mail?id=html-templates)  email templates for a better email templates management.

### altBody()

Use this method to set a `plain` text email message.

```php
$this->altBody('Hello Devs');
```

### send()

Use this method to finally send the mail.

```php
$this->send();
```
