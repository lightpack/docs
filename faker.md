# Faker System

Lightpack's **Faker** provides a lighweight fake data generator for tests, seeding, and development.

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
$sentence = $faker->sentence(); // 'Lorem ipsum dolor sit amet.'
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
    // ...
];
```

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
- `name()` — Full name
- `email()` — Email address
- `username()` — Username
- `phone()` — Phone number
- `address()` — Full address
- `city()`, `state()`, `country()` — Location fields

### Business & Commerce
- `company()` — Company name
- `jobTitle()` — Job title
- `productName()` — Product name
- `price($min, $max, $currency)` — Price string

### Internet & Tech
- `url()` — URL
- `ipv4()`, `ipv6()` — IP addresses
- `hexColor()` — Color code
- `uuid()` — RFC 4122 UUID
- `slug($words)` — URL slug

### Numbers & Types
- `number($min, $max)` — Integer
- `float($min, $max, $decimals)` — Float
- `bool()` — Boolean
- `enum($values)` — Random value from array

### Text
- `sentence($words)` — Sentence
- `paragraph($sentences)` — Paragraph
- `words` (locale data) — Word list used internally by methods like `sentence()`, `paragraph()`, and `slug()`. Not a direct method, but can be customized in your locale.

### Dates & Time
- `date($format)` — Date string
- `dob($minAge, $maxAge)` — Date of birth
- `age($min, $max)` — Age

### Security & Finance
- `password($length)` — Password
- `creditCardNumber()` — Card number (plausible, not valid)
- `otp($digits)` — One-time code

### Geo
- `latitude()` — Latitude
- `longitude()` — Longitude
- `zipCode()` — Zip/postal code

### Utilities
- `arrayOf($method, $count, ...$args)` — Generate array of fake values
- `seed($seed)` — Set deterministic seed

---

## UniqueFaker: Generating Unique Values

For unique fake data (e.g., unique emails), use `unique()`:

```php
$uniqueFaker = $faker->unique();
$email1 = $uniqueFaker->email();
$email2 = $uniqueFaker->email(); // Always different from $email1
```
- Guarantees uniqueness up to the pool size (throws if exhausted)
- Supports all Faker methods

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

---