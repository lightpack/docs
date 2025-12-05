# Faker System

Lightpack's **Faker** provides a lightweight fake data generator for tests, seeding, and development.

---

## Getting Started

```php
use Lightpack\Faker\Faker;

$faker = new Faker(); // Uses the default 'en' locale
```

### Basic Usage

```php
$name = $faker->name();         // 'John Smith'
$email = $faker->email();       // 'johnsmith@example.com'
$city = $faker->city();         // 'London'
$sentence = $faker->sentence(); // 'People over you have are we you come up.'
$uuid = $faker->uuid();         // 'e7b8...-...'
```

---

## Locale System

Faker supports locale-based data via simple PHP arrays. You can:
- Use built-in locales (e.g., `en`)
- Provide your own locale file
- Inject locale data as an array at runtime

### Using a Custom Locale File

```php
$faker = new Faker('custom', '/path/to/custom-locale.php');
```

A locale file returns an array of data lists:
```php
<?php
return [
    'firstNames' => ['Jean'],
    'lastNames' => ['Dupont'],
    'domains' => ['monsite.fr'],
    'cities' => ['Paris'],
    'states' => ['Île-de-France'],
    'countries' => ['France'],
    'streets' => ['Rue de la Paix'],
    'companies' => ['Acme Corp'],
    'jobTitles' => ['Engineer'],
    'productNames' => ['Widget'],
    'phonePrefixes' => ['+33'],
    'words' => ['the', 'be', 'to', 'of', 'and', ...], // Common English words
    // ...
];
```

> **Note:** If a locale key is missing, Faker automatically falls back to the English (`en`) locale data for that key.

### Injecting Locale Data Directly

```php
$faker = new Faker('custom');
$faker->setLocaleData([
    'firstNames' => ['Anna'],
    'lastNames' => ['Smith'],
    'domains' => ['example.test'],
    // ...
]);
```

---

## Faker API Methods

All methods are explicit, chainable, and deterministic (with seeding):

### Identity & Contact
- `firstName(): string` — First name only
- `lastName(): string` — Last name only
- `name(): string` — Full name (first + last)
- `email(): string` — Email address (derived from name + domain)
- `username(): string` — Username (format: `firstname.lastname##`)
- `phone(): string` — Phone number (prefix + 8 digits)
- `address(): string` — Full address (number, street, city, state, country)
- `city(): string` — City name
- `state(): string` — State name
- `country(): string` — Country name

### Business & Commerce
- `company()` — Company name
- `jobTitle()` — Job title
- `productName()` — Product name
- `price(float $min = 1, float $max = 9999, string $currency = '$'): string` — Formatted price string

### Internet & Tech
- `domainName(): string` — Domain name
- `url(): string` — Random URL with domain
- `userAgent(): string` — Browser user agent string (Chrome, Safari, Firefox, Edge)
- `ipv4(): string` — Random IPv4 address
- `ipv6(): string` — Random IPv6 address
- `hexColor(): string` — Random hex color code (e.g., '#A3F2B1')
- `uuid(): string` — RFC 4122 version 4 UUID
- `slug(int $words = 3): string` — URL-friendly slug

### Numbers & Types
- `number(int $min = 0, int $max = PHP_INT_MAX): int` — Random integer
- `float(float $min = 0, float $max = 1, int $decimals = 2): float` — Random float
- `bool(): bool` — Random boolean
- `enum(array $values)` — Random value from array

### Text
- `sentence(int $words = 8): string` — Random sentence using common English words
- `paragraph(int $sentences = 3): string` — Random paragraph (each sentence has 7-15 words)

### Dates & Time
- `date(string $format = 'Y-m-d'): string` — Random date (between 10 years ago and now)
- `datetime(string $format = 'Y-m-d H:i:s'): string` — Random datetime with time (last 1 year)
- `dob(int $minAge = 18, int $maxAge = 65): string` — Date of birth in 'Y-m-d' format
- `age(int $min = 0, int $max = 100): int` — Random age

### Security & Finance
- `password(int $length = 12): string` — Random password with letters, numbers, and special characters
- `creditCardNumber(): string` — Plausible card number (Visa/MasterCard/Amex format, **not Luhn-valid**)
- `otp(int $digits = 6): string` — Numeric one-time password code

### Geo
- `latitude(): float` — Random latitude (-90 to 90, 6 decimals)
- `longitude(): float` — Random longitude (-180 to 180, 6 decimals)
- `zipCode(): string` — Random zip/postal code (various formats)

### Utilities
- `arrayOf(string $method, int $count, ...$args): array` — Generate array of fake values
- `seed(int $seed): void` — Set deterministic seed (uses `mt_srand()`)

---

## UniqueFaker: Generating Unique Values

For unique fake data (e.g., unique emails), use `unique()`:

```php
$uniqueFaker = $faker->unique();
$email1 = $uniqueFaker->email();
$email2 = $uniqueFaker->email(); // Always different from $email1
```

**Important Details:**
- Attempts up to **100 times** to generate a unique value
- Throws `RuntimeException` if unable to generate unique value after 100 attempts
- Tracks all generated values in memory to ensure uniqueness
- Supports all Faker methods via `__call()` magic method

**Example of exhaustion:**
```php
$unique = $faker->unique();
// This will eventually throw RuntimeException after 100 failed attempts
for ($i = 0; $i < 1000; $i++) {
    $unique->enum(['A', 'B']); // Only 2 possible values
}
```

---

## Deterministic Seeding

You can make all fake data deterministic for repeatable tests:

```php
$faker = new Faker();
$faker->seed(1234);
echo $faker->name(); // Always the same output for the same seed
```

---

## Practical Usage Examples

### Generate a Batch of Fake Users
```php
$faker = new Faker();
$users = $faker->arrayOf('name', 5);
// [ 'John Smith', 'Priya Sharma', ... ]
```

### Unique Emails for Seeding
```php
$unique = $faker->unique();
$emails = [];
for ($i = 0; $i < 10; $i++) {
    $emails[] = $unique->email();
}
// All emails are unique
```

### Generate User Profile
```php
$user = [
    'first_name' => $faker->firstName(),
    'last_name' => $faker->lastName(),
    'email' => $faker->email(),
    'username' => $faker->username(),
    'password' => $faker->password(16),
    'phone' => $faker->phone(),
    'address' => $faker->address(),
    'registered_at' => $faker->datetime(),
];
```

### Generate Blog Post
```php
$post = [
    'title' => ucwords($faker->sentence(6)),
    'slug' => $faker->slug(5),
    'content' => $faker->paragraph(5),
    'author' => $faker->name(),
    'published_at' => $faker->datetime('Y-m-d H:i:s'),
];
```

---