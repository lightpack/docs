# Sending Emails

**Lightpack** provides a flexible, driver-based email system that supports multiple mail services. You can easily send emails in plain `Text` or rich `HTML` format using SMTP, third-party APIs (like Resend), or custom drivers.

## Configuration

Configure your mail driver in the `.env` file:

```env
MAIL_DRIVER=smtp
MAIL_FROM_ADDRESS=noreply@example.com
MAIL_FROM_NAME="My Application"

# SMTP Configuration
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=your-username
MAIL_PASSWORD=your-password
MAIL_ENCRYPTION=tls

# Resend Configuration (if using resend driver)
RESEND_API_KEY=re_xxxxxxxxxxxxx
```

## Available Drivers

Lightpack includes these built-in drivers:

- **smtp** - Standard SMTP (uses PHPMailer)
- **array** - For testing (stores emails in memory)
- **log** - Logs emails to file (for development)
- **resend** - Resend API integration (requires `composer require resend/resend-php`)

## Creating Mail Class

Once you have setup your `SMTP` credentials, you should now create a mail class. To create a new mail class, fire this command in your terminal from project root:

```terminal
php console create:mail TestMail
```

This should have created a `TestMail.php` file in **app/Mails** folder. You can write your mail logic in `dispatch()` method. Here is a minimal example of sending an email:

```php
public function dispatch(array $payload = [])
{
    $this->to('bob@example.com')
        ->from('joe@example.com')
        ->subject('Hello Bob')
        ->body('Welcome to Lightpack')
        ->send();
}
```

## Sending Mail

To send the mail, simply instantiate the mail class and call its `dispatch()` method:

```php
(new TestMail)->dispatch();
```

You can optionally pass it an array as data payload which you can use inside the `dispatch()` method:

```php
(new TestMail)->dispatch([
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

Now in the mail class `dispatch()` method, specify those two templates using `htmlView()` and `textView()` methods.

```php
class WelcomeMail extends Mail
{
    public function dispatch(array $payload = [])
    {
        $this->to('bob@example.com')
            ->from('joe@example.com')
            ->subject('Hello Bob')
            ->htmlView('mails/welcome.html')
            ->textView('mails/welcome.text')
            ->send();
    }
}
```

**Thats it!!** Now you can simply send both the versions of the mail.

### Passing View Data

In case you need to pass data to your templates, use the `viewData()` method passing it an array as argument for data. 

```php
public function dispatch(array $payload = [])
{
    $this->viewData([
        'title' => 'Hello Devs',
        'content' => 'Welcome to Lightpack PHP web framework.',
    ]);

    $this->to('bob@example.com')
        ->from('joe@example.com')
        ->subject('Hello Bob')
        ->htmlView('mails/welcome.html')
        ->textView('mails/welcome.text')
        ->send();
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
$this->attach('/path/to/file.pdf');
```

You can optionally specify a custom filename:

```php
$this->attach('/path/to/file.pdf', 'document.pdf');
```

You can `attach` multiple files at once by passing it an array of filepath and optional filename as **key-value** pairs:

```php
$this->attach([
    '/path/to/file1.pdf' => 'invoice.pdf',
    '/path/to/file2.pdf' => 'receipt.pdf',
]);
```

Or without custom filenames:

```php
$this->attach([
    '/path/to/file1.pdf',
    '/path/to/file2.pdf',
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

However, it is recommended to create and send [rich HTML](/mails?id=html-templates)  email templates for a better email templates management.

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

## Switching Drivers

You can switch drivers per-mail using the `driver()` method:

```php
public function dispatch(array $payload = [])
{
    $this->driver('resend')  // Use Resend for this email
        ->to('user@example.com')
        ->subject('Welcome')
        ->body('<h1>Hello!</h1>')
        ->send();
}
```

This is useful when you want to use different services for different types of emails (e.g., transactional vs marketing).

## Testing Emails

Lightpack provides comprehensive testing support for emails.

### Using Array Driver

Set `MAIL_DRIVER=array` in your test environment to capture emails in memory:

```php
class MyTest extends TestCase
{
    use MailAssertionTrait;

    public function testWelcomeEmailIsSent()
    {
        // Send email
        (new WelcomeMail)->dispatch(['to' => 'user@example.com']);

        // Assert email was sent
        $this->assertMailSent()
            ->assertMailSentTo('user@example.com')
            ->assertMailSubject('Welcome')
            ->assertMailContains('Hello');
    }
}
```

### Available Assertions

- `assertMailSent()` - Assert at least one email was sent
- `assertMailNotSent()` - Assert no emails were sent
- `assertMailCount(int $count)` - Assert exact number of emails sent
- `assertMailSentTo(string $email)` - Assert email sent to recipient
- `assertMailSentFrom(string $email)` - Assert email sent from address
- `assertMailSubject(string $subject)` - Assert email has subject
- `assertMailContains(string $text)` - Assert email body contains text
- `assertMailCc(string $email)` - Assert email CC'd to address
- `assertMailBcc(string $email)` - Assert email BCC'd to address
- `assertMailReplyTo(string $email)` - Assert reply-to address
- `assertMailHasAttachment(string $filename)` - Assert has attachment
- `assertMailHasNoAttachments()` - Assert no attachments

All assertions support fluent chaining.

## Creating Custom Drivers

You can easily create custom drivers for any email service:

### 1. Create Driver Class

```php
namespace App\Mail\Drivers;

use Lightpack\Mail\DriverInterface;

class MailgunDriver implements DriverInterface
{
    private $apiKey;
    private $domain;

    public function __construct()
    {
        $this->apiKey = get_env('MAILGUN_API_KEY');
        $this->domain = get_env('MAILGUN_DOMAIN');
    }

    public function send(array $data): bool
    {
        // $data contains:
        // - to: [['email' => 'user@example.com', 'name' => 'User']]
        // - from: ['email' => 'sender@example.com', 'name' => 'Sender']
        // - subject: string
        // - html_body: string
        // - text_body: string
        // - cc, bcc, reply_to: arrays (optional)
        // - attachments: [['path' => '/path', 'filename' => 'name.pdf']]

        // Implement your Mailgun API logic here
        // Return true on success, throw exception on failure
    }
}
```

### 2. Register Driver

In your `MailProvider` or bootstrap file:

```php
app('mail')->registerDriver('mailgun', new MailgunDriver());
```

### 3. Use Custom Driver

```php
// In .env
MAIL_DRIVER=mailgun

// Or per-mail
$this->driver('mailgun')->to('user@example.com')->send();
```

## Data Structure Reference

All drivers receive normalized data in this format:

```php
[
    'to' => [
        ['email' => 'user@example.com', 'name' => 'User Name'],
        ['email' => 'user2@example.com', 'name' => '']
    ],
    'from' => ['email' => 'sender@example.com', 'name' => 'Sender Name'],
    'subject' => 'Email Subject',
    'html_body' => '<h1>HTML content</h1>',
    'text_body' => 'Plain text content',
    'cc' => [['email' => 'cc@example.com', 'name' => 'CC Name']],
    'bcc' => [['email' => 'bcc@example.com', 'name' => '']],
    'reply_to' => [['email' => 'reply@example.com', 'name' => 'Reply Name']],
    'attachments' => [
        ['path' => '/absolute/path/to/file.pdf', 'filename' => 'document.pdf']
    ]
]
```

---