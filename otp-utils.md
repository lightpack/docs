# OTP Utilities

Lightpack’s `Otp` utility provides a fluent, flexible API for generating one-time passwords (OTPs) and codes for authentication, verification, or any scenario where you need a secure, random code.

---

## Overview

- **Fluent API:** Chain methods to configure code length, type, and charset.
- **Types Supported:** Numeric, alphabetic, alphanumeric, or custom charset.
- **Length:** 1–32 characters (defaults to 6 if invalid).

---

## Basic Usage

```php
use Lightpack\Utils\Otp;

$otp = new Otp();
$code = $otp->generate(); // 6-digit numeric code by default
```

---

## Fluent Configuration

### Set Code Length
```php
$otp->length(4); // 4 characters
```

### Set Code Type
- **Numeric:** Only digits 0–9
- **Alpha:** Uppercase A–Z
- **Alnum:** Uppercase A–Z and digits 0–9
- **Custom:** Any charset you provide

```php
$otp->type('numeric'); // Default, e.g. 123456
$otp->type('alpha');   // e.g. ABCDEF
$otp->type('alnum');   // e.g. 1A2B3C
$otp->type('custom')->charset('XYZ123'); // e.g. 1Z2XY
```

### Full Example
```php
$otp = (new Otp)
    ->length(8)
    ->type('alnum');

$code = $otp->generate(); // e.g. 1A2B3C4D
```

---

## Custom Charset

You can fully control the character set:
```php
$otp = (new Otp)
    ->length(5)
    ->type('custom')
    ->charset('ABC123');

$code = $otp->generate(); // e.g. 1B2CA
```

---

## Edge Cases & Defaults

- If you set an invalid length (<1 or >32), the OTP will default to 6 characters.
- If you set a custom type but do not provide a charset, it defaults to uppercase letters and digits.

---