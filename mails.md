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
- **log** - Logs emails to `storage/logs/mails.json` (for development)
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

To send the mail, use the `app()` helper to instantiate the mail class and call its `dispatch()` method:

```php
app(TestMail::class)->dispatch();
```

The `app()` helper resolves classes from Lightpack's dependency injection container.

### Using in Controllers

You can also type-hint mail classes directly in controller methods, and Lightpack will auto-inject them:

```php
class UserController
{
    public function sendWelcome(WelcomeMail $mail)
    {
        $mail->dispatch([
            'to' => 'user@example.com',
            'name' => 'John Doe'
        ]);
        
        return response()->json(['message' => 'Email sent!']);
    }
}
```

The container automatically resolves `WelcomeMail` with all its dependencies when the controller method is called.

### Passing Data

You can optionally pass an array as data payload which you can use inside the `dispatch()` method:

```php
app(TestMail::class)->dispatch([
    'to' => 'devs@example.com',
    'from' => 'lightpack@example.com',
]);
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
    'rob@example.com',
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
    'rob@example.com',
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

### template()

Use this method to set both HTML and plain text bodies using a `MailTemplate` instance.

```php
$template = (new MailTemplate)
    ->heading('Welcome!')
    ->paragraph('Thanks for signing up.')
    ->button('Get Started', 'https://example.com');

$this->template($template);
```

This automatically sets both `htmlBody` and `textBody` from the template. See the [MailTemplate](#mailtemplate-programmatic-email-builder) section for full documentation.

### send()

Use this method to finally send the mail.

```php
$this->send();
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

## MailTemplate - Programmatic Email Builder

**MailTemplate** provides a powerful, fluent API for building beautiful, responsive emails programmatically. It handles all the complexity of email client compatibility while giving you a clean, type-safe interface.

### Why MailTemplate?

Building HTML emails is notoriously difficult:

- **Email clients are inconsistent** - What works in Gmail might break in Outlook
- **Inline styles required** - External CSS doesn't work in most email clients
- **Table-based layouts** - Modern CSS flexbox/grid aren't supported
- **Plain text versions** - Many users prefer plain text emails

**MailTemplate** solves all of this:

- ✅ **Email client compatible** - Works in Gmail, Outlook, Apple Mail, etc.
- ✅ **Automatic plain text** - Generates readable plain text from your components
- ✅ **Beautiful default layout** - Professional design out of the box
- ✅ **Programmatic building** - Type-safe, IDE-friendly fluent interface
- ✅ **XSS protection** - Automatic HTML escaping
- ✅ **Customizable** - Override colors, fonts, spacing, and layouts
- ✅ **UTF-8 safe** - Handles international characters and currency symbols

### Quick Example

```php
use Lightpack\Mail\MailTemplate;

class WelcomeMail extends Mail
{
    public function dispatch(array $payload = [])
    {
        $template = (new MailTemplate)
            ->logo(asset()->url('img/logo.svg'), 50)
            ->heading('Welcome to Our Platform!')
            ->paragraph('Thanks for signing up. We\'re excited to have you on board.')
            ->button('Get Started', $payload['actionUrl'], 'primary')
            ->divider()
            ->paragraph('If you have any questions, feel free to reply to this email.')
            ->footer('&copy; 2025 MyApp. All rights reserved.');

        $this->to($payload['email'])
            ->subject('Welcome!')
            ->template($template)  // Sets both HTML and plain text
            ->send();
    }
}
```

### Available Components

MailTemplate provides these building blocks:

#### heading()

Add headings with 3 levels (H1, H2, H3):

```php
$template->heading('Welcome!', 1);  // H1 - largest
$template->heading('Section Title', 2);  // H2 - medium
$template->heading('Subsection', 3);  // H3 - smallest
```

#### paragraph()

Add text paragraphs:

```php
$template->paragraph('This is a paragraph of text.');
```

#### button()

Add call-to-action buttons with color variants:

```php
$template->button('Click Me', 'https://example.com', 'primary');
$template->button('Learn More', 'https://example.com', 'success');
$template->button('Delete', 'https://example.com', 'danger');
```

Available colors: `primary`, `secondary`, `success`, `danger`, `warning`, `info`

#### link()

Add clickable URLs (handles long URLs gracefully):

```php
$template->link('https://example.com/very/long/url');
$template->link('https://example.com', 'Click here');  // Custom text
```

#### divider()

Add horizontal dividers to separate sections:

```php
$template->divider();
```

#### alert()

Add alert boxes with different types:

```php
$template->alert('Important notice', 'info');
$template->alert('Success message', 'success');
$template->alert('Warning message', 'warning');
$template->alert('Error message', 'danger');
```

#### bulletList()

Add bullet point lists:

```php
$template->bulletList([
    'First item',
    'Second item',
    'Third item',
]);
```

#### keyValueTable()

Add key-value tables (perfect for order details, user info, etc.):

```php
$template->keyValueTable([
    'Order ID' => '#12345',
    'Date' => '2025-01-15',
    'Total' => '$99.99',
    'Status' => 'Confirmed',
]);
```

#### table()

Add multi-column tables with headers:

```php
$template->table(
    ['Name', 'Email', 'Status'],  // Headers
    [
        ['John Doe', 'john@example.com', 'Active'],
        ['Jane Smith', 'jane@example.com', 'Pending'],
    ]
);
```

#### code()

Add code blocks:

```php
$template->code('composer require lightpack/framework');
```

#### image()

Add images:

```php
$template->image('https://example.com/image.png', 'Alt text', 300);
$template->image('https://example.com/logo.png', 'Logo', null, 'center');
```

#### html()

Add raw HTML content without escaping. This is useful when you need:

**Use Cases:**
- HTML entities like `&copy;`, `&trade;`, `&reg;`, `&#8377;` (₹)
- Pre-rendered HTML from a trusted source
- Complex formatting that other components don't support
- Content from your own templates or database (that you control)

```php
// HTML entities
$template->html('&copy; 2025 MyApp. All rights reserved.');

// Currency symbols
$template->html('Price: &#8377; 725.04');  // Indian Rupee

// Complex formatting
$template->html('<p style="color: #666;">Custom <strong>formatted</strong> text</p>');
```

**Warning:** Content is NOT escaped for XSS protection. Never use with user-provided input:

```php
// ❌ DANGEROUS - XSS vulnerability
$template->html($userInput);

// ✅ SAFE - Use paragraph() instead (auto-escaped)
$template->paragraph($userInput);
```

### Header & Footer

#### logo()

Add a logo to the email header:

```php
$template->logo('https://example.com/logo.svg', 120);
```

For multi-tenant SaaS with varying logo sizes:

```php
// Auto-adjusts for any logo shape/size
$template->logo($client->logo_url);

// Custom dimensions
$template->logo($client->logo_url, 150, null, 80);  // width, height, max-height

// Force exact size
$template->logo($client->logo_url, 120, 50);  // May distort aspect ratio
```

**Parameters:**
- `$url` - Logo URL
- `$width` - Width in pixels (default: 120)
- `$height` - Height in pixels (null = auto, maintains aspect ratio)
- `$maxHeight` - Maximum height constraint (default: 60, prevents tall logos)

#### footer()

Add footer text (supports HTML entities):

```php
$template->footer('&copy; 2025 MyApp. All rights reserved.');
```

#### footerLinks()

Add footer links:

```php
$template->footerLinks([
    'Privacy' => 'https://example.com/privacy',
    'Terms' => 'https://example.com/terms',
    'Contact' => 'https://example.com/contact',
    'Unsubscribe' => 'https://example.com/unsubscribe',
]);
```

### Customization

#### Custom Colors

```php
$template = new MailTemplate([
    'colors' => [
        'primary' => '#FF5733',
        'success' => '#10B981',
    ],
]);

// Or set after creation
$template->setColors(['primary' => '#FF5733']);
```

Available color keys: `primary`, `secondary`, `success`, `danger`, `warning`, `info`, `text`, `textLight`, `background`, `white`, `border`

#### Custom Fonts

Customizing fonts in emails is tricky because email clients have limited support. Here's what you need to know:

**Font Family:**
Always provide fallbacks because custom fonts often don't work in email clients:

```php
$template = new MailTemplate([
    'fonts' => [
        // Web-safe fonts with fallbacks (recommended)
        'family' => 'Georgia, "Times New Roman", serif',
        
        // Or system fonts
        'family' => '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Arial, sans-serif',
    ],
]);

// Or set after creation
$template->setFonts(['family' => 'Georgia, serif']);
```

**Available Font Keys:**
- `family` - Font family with fallbacks
- `sizeBase` - Base font size (default: 15px)
- `sizeSmall` - Small text size (default: 13px)
- `sizeLarge` - Large text size (default: 17px)
- `sizeH1`, `sizeH2`, `sizeH3` - Heading sizes

**Example - Larger, More Readable Fonts:**

```php
$template->setFonts([
    'family' => 'Georgia, serif',
    'sizeBase' => '16px',    // Larger body text
    'sizeH1' => '24px',      // Larger headings
]);
```

**Important:** Web fonts (Google Fonts, custom fonts) are NOT supported in most email clients (Gmail, Outlook). Stick to web-safe fonts:
- **Serif:** Georgia, "Times New Roman", Times
- **Sans-serif:** Arial, Helvetica, Verdana, Tahoma
- **Monospace:** "Courier New", Courier

### Complete Example

Here's a complete order confirmation email:

```php
class OrderConfirmationMail extends Mail
{
    public function dispatch(array $payload = [])
    {
        $order = $payload['order'];
        
        $template = (new MailTemplate)
            ->logo(asset()->url('img/logo.svg'), 50)
            ->heading('Order Confirmation')
            ->paragraph("Thank you for your order, {$order['customer_name']}!")
            ->divider()
            ->heading('Order Details', 2)
            ->keyValueTable([
                'Order ID' => $order['id'],
                'Date' => $order['date'],
                'Total' => $order['total'],
                'Payment Method' => $order['payment_method'],
            ])
            ->divider()
            ->heading('Items Ordered', 2)
            ->table(
                ['Product', 'Quantity', 'Price'],
                $order['items']
            )
            ->divider()
            ->alert('Your order will ship within 2-3 business days.', 'info')
            ->button('Track Order', $order['tracking_url'], 'primary')
            ->divider()
            ->paragraph('Questions? Reply to this email or contact our support team.')
            ->footer('&copy; 2025 MyStore. All rights reserved.')
            ->footerLinks([
                'Privacy' => 'https://mystore.com/privacy',
                'Terms' => 'https://mystore.com/terms',
                'Support' => 'https://mystore.com/support',
            ]);

        $this->to($order['customer_email'])
            ->subject("Order Confirmation #{$order['id']}")
            ->template($template)
            ->send();
    }
}
```

### Reusing Common Branding

For applications that send multiple email types with consistent branding, create a base mail class:

```php
namespace App\Mails;

use Lightpack\Mail\Mail;
use Lightpack\Mail\MailTemplate;

abstract class AppMail extends Mail
{
    protected function template(): MailTemplate
    {
        return (new MailTemplate)
            ->logo(asset()->url('img/logo.svg'), 50)
            ->footer('&copy; 2025 MyApp. All rights reserved.')
            ->footerLinks([
                'Privacy' => url('/privacy'),
                'Terms' => url('/terms'),
                'Contact' => url('/contact'),
            ]);
    }
}
```

Then extend it in your specific emails:

```php
class WelcomeMail extends AppMail
{
    public function dispatch(array $payload = [])
    {
        $template = $this->template()  // Gets logo, footer, links
            ->heading('Welcome!')
            ->paragraph('Thanks for signing up!')
            ->button('Get Started', $payload['actionUrl']);

        $this->to($payload['email'])
            ->subject('Welcome!')
            ->template($template)
            ->send();
    }
}
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
        app(WelcomeMail::class)->dispatch([
            'to' => 'user@example.com',
            'subject' => 'Welcome',
            'body' => '<h1>Hello!</h1>',
        ]);

        // Assert email was sent
        $this->assertMailSent()
            ->assertMailSentTo('user@example.com')
            ->assertMailSubject('Welcome')
            ->assertMailContains('Hello');
    }
}
```

You can view all the available assertions in the [MailAssertionTrait](/testing?id=asserting-emails).

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

### 2. Use Custom Driver

```php
class WelcomeMail extends Mail
{
    public function dispatch(array $payload = [])
    {
        $this->driver('mailgun')
            ->to('user@example.com')
            ->subject('Welcome')
            ->body('<h1>Hello</h1>')
            ->send();
    }
}
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