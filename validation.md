# Lightpack Validation

Lightpack provides robust data validation support. This can be helpful when validating form inputs or data to be stored in database.

## Quick Start

In your controller's method, you can typehint `Lightpack\Validation\Validator` as dependency or you can call the `validator()` helper function.

```php
$validator = validator();

$validator
    ->field('username')->required()->string()->min(3)->max(50)
    ->field('email')->required()->email()
    ->field('age')->int()->between(18, 100);

$validator->validate($_POST);

if ($validator->fails()) {
    $errors = $validator->getErrors();
}
```

---

## How Validation Works

1. **Define fields and rules** using the fluent API.
2. **Call `validate($input)`** to run validation.
3. **Check results** with `passes()`, `fails()`, `getErrors()`, or `getError('field')`.

---

## Available Rules

### String Rules
- `required()` — Field must be present and not empty
- `string()` — Must be a string
- `min($n)` / `max($n)` — Length constraints
- `length($n)` — Exact length
- `alpha()` / `alphaNum()` — Only letters or letters/numbers
- `regex($pattern)` — Custom regex
- `slug()` — URL-friendly string

### Numeric Rules
- `numeric()` — Any number
- `int()` / `float()` — Integer or float
- `between($min, $max)` — Value range
- `min($n)` / `max($n)` — Value constraints

### Date/Time Rules
- `date($format = null)` — Valid date (optionally with format)
- `before($date, $format = null)` / `after($date, $format = null)` — Date comparison

### Boolean Rules
- `bool()` — Must be boolean

### Array Rules
- `array($min = null, $max = null)` — Must be array, with optional length
- `in($values)` / `notIn($values)` — Value must (not) be in list
- `unique()` — All values must be unique

### Comparison Rules
- `same($field)` — Must match another field
- `different($field)` — Must not match another field

### File & Image Rules
- `file()` — Valid file upload
- `fileSize($size)` — Max file size (e.g. '2M', '500K')
- `fileType($types)` — Allowed MIME types
- `fileExtension($exts)` — Allowed extensions
- `files($min = null, $max = null)` — Multiple files
- `image($options)` — Image validation (width, height, ratio)

### Custom & Advanced Rules
- `custom($fn, $message)` — Custom closure for validation
- `transform($fn)` — Preprocess value before validation
- `nullable()` — Field can be null or empty

---

## Wildcards & Nested Validation

Validate arrays of objects or deeply nested data with wildcards:

```php
$validator
    ->field('users.*.email')->required()->email()
    ->field('users.*.roles')->array()->in(['admin', 'user']);

$input = [
    'users' => [
        ['email' => 'john@example.com', 'roles' => ['admin']],
        ['email' => 'jane@example.com', 'roles' => ['user']],
    ]
];

$validator->validate($input);
```

---

## File & Image Validation

```php
$validator->field('avatar')
    ->file()
    ->fileSize('1M')
    ->fileType(['image/jpeg', 'image/png'])
    ->image([
        'min_width' => 100,
        'max_width' => 1000,
        'min_height' => 100,
        'max_height' => 1000,
        'ratio' => '1:1'
    ]);
```

Multiple files:

```php
$validator->field('photos')
    ->files(1, 5)
    ->fileSize('2M')
    ->fileType(['image/jpeg', 'image/png']);
```

---

## Custom Rules & Transformers

### Register a custom rule globally

```php
$validator->addRule('uppercase', function($value) {
    return strtoupper($value) === $value;
}, 'Must be uppercase');

$validator->field('code')->uppercase();
```

### Custom rule per field

```php
$validator->field('code')->custom(function($value) {
    return preg_match('/^CODE-\\d{6}$/', $value);
}, 'Invalid code format');
```

### Transform values before validation

```php
$validator->field('tags')
    ->transform(fn($v) => explode(',', $v))
    ->array();
```

### Class-Based Custom Rules

For advanced scenarios, you can use **invokable classes** as custom validation rules. This is ideal for business logic that needs dependencies, database access, or configuration.

**Example: Unique Email Rule**

```php
namespace App\Rules;

use App\Models\User;

class UniqueEmailRule
{
    public const MESSAGE = 'Email already exists';

    public function __construct(
        private ?int $excludeId = null
    ) {}

    public function __invoke(string $email): bool
    {
        return User::query()
            ->where('email', '=', $email)
            ->whereIf($this->excludeId, 'id', '!=', $this->excludeId)
            ->notExists();
    }
}
```

**Usage:**

```php
$validator->field('email')
    ->required()
    ->email()
    ->custom(new UniqueEmailRule, UniqueEmailRule::MESSAGE);
```

- The rule can accept constructor arguments (e.g., `$excludeId` for updates).
- The validator will call the class as an invokable (`__invoke`) object.
- This pattern keeps business logic clean, testable, and reusable.

---

## Error Handling & Messages

- `getErrors()` — All errors as `[field => message]`
- `getError('field')` — First error for a field
- `getFieldErrors('field')` — All errors for a field

### Custom error messages

```php
$validator->field('age')
    ->numeric()
    ->message('Age must be a number')
    ->between(18, 100)
    ->message('Age must be between 18 and 100');
```

---

## Advanced: Password Strength

```php
$validator->field('password')
    ->required()
    ->between(8, 32)
    ->hasUppercase()
    ->hasLowercase()
    ->hasNumber()
    ->hasSymbol();
```

---

## Example: User Registration

```php
$validator
    ->field('username')->required()->string()->min(3)->max(50)->alphaNum()
    ->field('email')->required()->email()
    ->field('password')->required()->between(8, 32)->hasUppercase()->hasLowercase()->hasNumber()->hasSymbol()
    ->field('avatar')->nullable()->image([
        'max_width' => 1000,
        'max_height' => 1000,
        'max_size' => '1M'
    ]);
```

---

## Advanced: Nested & Conditional Validation

```php
$validator
    ->field('user.profile.name')->required()
    ->field('user.profile.age')->required()->int()->custom(fn($v) => $v >= 18, 'Must be 18 or older');

$validator
    // Only one address can be primary
    ->field('user.addresses')->custom(function($addresses) {
        $primaryCount = 0;
        foreach ($addresses as $address) {
            if ($address['is_primary']) {
                $primaryCount++;
            }
        }
        return $primaryCount === 1;
    }, 'Only one address can be marked as primary');
```

---