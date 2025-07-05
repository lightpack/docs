# Lightpack Captcha System

Lighpack makes it painless to add **CAPTCHA** protection to your apps—whether you need a simple image challenge, Google reCAPTCHA, Cloudflare Turnstile, or a no-op for testing.

- **Purpose:** Prevent automated bots from abusing forms, APIs, or other user-facing endpoints.
- **Where to Use:** Registration, login, comments, password reset, or any action vulnerable to spam or brute-force.

---

## Supported Drivers

| Driver        | Class                       | Use Case                                      |
|---------------|-----------------------------|-----------------------------------------------|
| `null`        | `NullCaptcha`               | Testing/dev, disables CAPTCHA (always passes) |
| `native`      | `NativeCaptcha`             | Simple image CAPTCHA (no external API)        |
| `recaptcha`   | `GoogleReCaptcha`           | Google reCAPTCHA v2 widget                    |
| `turnstile`   | `CloudflareTurnstile`       | Cloudflare Turnstile widget                   |

> See config/captcha.php file configuration details.

---

## CaptchaInterface

All drivers implement:

```php
// Render the challenge (HTML/image)
public function generate(): string;

// Validate user captcha response
public function verify(): bool;     
```

---

## Usage Patterns

### Rendering a CAPTCHA
```php
echo captcha()->generate();
```

### Verifying User Response
```php
// In your form handler:
if (captcha()->verify()) {
    // Passed
} else {
    // Failed
}
```

## Driver Details

### NullCaptcha (Testing/Dev)
- **No challenge, always passes.**
- Use for local/dev/testing only.

```php
echo captcha()->generate(); // ''

captcha()->verify(); // always true
```

### NativeCaptcha (Image-based)
- **No external API.** Generates PNG image with random text.
- Stores captcha verification answer in session.
- Configurable: font, width, height.

```php
// Configure image captcha settings
captcha()
    ->font('/path/to/font.ttf')
    ->width(200)
    ->height(60);

// In the form view
echo '<img src="' . captcha()->generate() . '" />';

// In form handler
if (captcha()->verify()) { 
    // Passed
}
```

### GoogleReCaptcha
- **Google-hosted widget.** 
- Requires site/secret keys in config.
- Renders widget, verifies via Google API.

```php
// Outputs Google recaptcha
echo captcha()->generate(); 

// In form handler
if (captcha()->verify()) {
    // passed
}
```

### CloudflareTurnstile
- **Cloudflare-hosted widget**.
- Requires site/secret keys in config.
- Renders widget, verifies via Cloudflare API.

```php
// Outputs <div class="cf-turnstile" ...>
echo captcha()->generate(); 

// In form handler
if (captcha()->verify()) {
    // passed
}
```

---