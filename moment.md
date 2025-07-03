# Moment Utility

Lightpack’s `Moment` is a simple, expressive, and timezone-aware date/time utility for PHP. It is designed for clarity, practical use, and covers the most common date/time operations with a clean, chainable API.

---

## Key Features
- **Timezone support:** Default is UTC, but you can set any valid timezone.
- **Flexible formatting:** Chain `format()` to set output format for all methods.
- **Chainable configuration:** All config methods return `$this` for fluent API.
- **Safe error handling:** Throws `InvalidArgumentException` for invalid timezones or date strings.
- **Comprehensive:** Covers today, tomorrow, yesterday, next/last weekday, month ends, travel, diff, human-friendly differences, and more.

---

## Instantiation & Timezone

```php
use Lightpack\Utils\Moment;

$moment = new Moment(); // Defaults to UTC
$moment = new Moment('Asia/Kolkata'); // Custom timezone

// Change timezone on the fly (chainable)
$moment->setTimezone('Europe/Berlin');

// Get current timezone
$tz = $moment->getTimezone(); // e.g. 'Europe/Berlin'
```

---

## Formatting

- Default output: `Y-m-d H:i:s`
- Use `format()` to change output format (chainable)

```php
$moment->format('d-M, Y H:ia')->now();
$moment->format('Y/m/d')->today();
```

---

## Core Methods

### now
Returns current date/time (in set timezone).

**Parameters:**
- `?format` (string, optional): Output format. Defaults to the format set by `format()` or `Y-m-d H:i:s`.

**Examples:**
```php
$moment->now();
$moment->now('d-M, Y');
```

### today, tomorrow, yesterday
Returns the respective date string in the current timezone.

**Parameters:**
- `?format` (string, optional): Output format.

**Examples:**
```php
$moment->today();
$moment->tomorrow('d-M, Y');
$moment->yesterday();
```

### next, last
Returns the date string for the next or last occurrence of a weekday.

**Parameters:**
- `dayName` (string): Full or short weekday name (e.g., 'monday', 'mon').
- `?format` (string, optional): Output format.

**Examples:**
```php
$moment->next('monday');
$moment->last('fri', 'Y-m-d');
```

### thisMonthEnd, nextMonthEnd, lastMonthEnd
Returns the last day of the current, next, or last month.

**Parameters:**
- `?format` (string, optional): Output format.

**Examples:**
```php
$moment->thisMonthEnd();
$moment->nextMonthEnd('Y-m-d');
$moment->lastMonthEnd();
```

### travel
Returns a date/time string after applying a modifier to the current date/time.

**Parameters:**
- `modifier` (string): Any string accepted by `DateTime::modify()` (e.g., '+5 days', '-2 hours', 'noon', 'last day of next month').
- `?format` (string, optional): Output format.

**Examples:**
```php
$moment->travel('+5 days');
$moment->travel('last day of this month', 'd-M, Y');
```

### diff
Returns a `DateInterval` object for the absolute difference between two datetimes.

**Parameters:**
- `datetime1` (string)
- `datetime2` (string)

**Examples:**
```php
$diff = $moment->diff('2021-07-23 14:25:45', '2019-03-14 08:23:12');
$diff->y; // years
$diff->m; // months
$diff->d; // days
$diff->h; // hours
$diff->i; // minutes
$diff->s; // seconds
```

### daysBetween
Returns the number of days between two dates (absolute).

**Parameters:**
- `datetime1` (string)
- `datetime2` (string)

**Examples:**
```php
$days = $moment->daysBetween('2021-07-23', '2021-05-18');
```

### fromNow
Returns a human-friendly difference string (e.g., 'just now', '5 minutes ago', '2 weeks ago').

**Parameters:**
- `datetime` (string, optional): Date/time string to compare to now. Defaults to `'now'`.

**Examples:**
```php
$moment->fromNow('2021-07-20 11:30:45');
// Output: 'x days ago', etc.
```

### create
Returns a `DateTime` object in the current timezone. Throws on invalid input.

**Parameters:**
- `datetime` (string, optional): Date/time string. Defaults to `'now'`.

**Examples:**
```php
$dt = $moment->create();
$dt = $moment->create('2021-07-23');
$dt = $moment->create('2021-07-23 10:00:00');

// Use all native DateTime methods:
$moment->create()->modify('+2 hours')->format('Y-m-d H:i:s');
```

---

## Practical Examples

### Chainable Formatting & Timezone

```php
$moment->format('Y-m-d')->setTimezone('Asia/Kolkata')->now();
```

### Get the next Friday in custom format

```php
$moment->next('friday', 'd/m/Y');
```

### Get human-friendly difference

```php
$moment->fromNow('2023-06-15 10:00:00'); // e.g. 'a year ago'
```

### Calculate days between two dates

```php
$days = $moment->daysBetween('2024-01-01', '2024-01-31'); // 30
```

### Error Handling

```php
try {
    $moment->setTimezone('Invalid/Zone');
} catch (\InvalidArgumentException $e) {
    // Handle invalid timezone
}

try {
    $moment->create('invalid-date');
} catch (\InvalidArgumentException $e) {
    // Handle invalid date string
}
```

---