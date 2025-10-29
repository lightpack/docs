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

## Sticky Forms and Errors

If the form validation fails for the current request, you would want to:
- Repopulate fields with the user's previous input ("sticky" forms)
- Show validation error messages next to each field

When you call `validateRequest()` method on the **validator** instance, Lightpack automatically sets the current request input data and validation error messages in the active session.

Lightpack provides two helpers:
- `old('field')` — Returns the previous value for a field
- `error('field')` — Returns the validation error message for a field


### old()

`old(string $key, $default = '', bool $escape = true)`

**What it does:**
Returns the previously submitted value for a form field, or a default if not present.

**When to use:**
- To repopulate form fields after a validation error, so users don’t lose their input.
- Especially useful in large forms or when validation fails.

**Example:**
```php
<input name="email" value="<?= old('email') ?>">
```
If the user submitted the form and it failed validation, their email input will be preserved.

### error()

`error(string $key)`

**What it does:**
Returns the validation error message for a specific field, if any.

**When to use:**
- To show users what went wrong with their input after a failed form submission.
- Place near each form field for clear feedback.

**Example:**
```php
<input name="email" value="<?= old('email') ?>">
<span class="error"><?= error('email') ?></span>
```
If validation fails for `email`, the error message appears next to the field.

### Example

Below is an example showing usage of above two helper functions.

**Controller**

```php
public function register(Request $request)
{
    $validator = validator()
        ->field('username')->required()->min(3)
        ->field('email')->required()->email()
        ->field('password')->required()->min(8);

    $validator->validateRequest();

    if ($validator->fails()) {
        return redirect()->back();
    }
    
    // ... proceed with registration
}
```

**View**

```php
<form method="POST">
    <?= csrf_input() ?>

    <label>Username</label>
    <input name="username" value="<?= old('username') ?>">
    <span class="error"><?= error('username') ?></span>

    <label>Email</label>
    <input name="email" value="<?= old('email') ?>">
    <span class="error"><?= error('email') ?></span>

    <label>Password</label>
    <input name="password" type="password">
    <span class="error"><?= error('password') ?></span>

    <button type="submit">Register</button>
</form>
```

---

## Form Requests

The `FormRequest` class in Lightpack provides a powerful, expressive, and reusable way to handle HTTP request validation and authorization in your applications. It encapsulates validation logic, error handling, and request data preparation, making your controllers clean and focused.

---

### Key Features

- **Thin Controllers:** Move all validation logic to FormRequest classes.
- **Reusable Rules:** Centralize and reuse request validation across controllers.
- **Automatic Validation:** Requests are validated before reaching your controller logic.
- **AJAX & JSON Support:** Automatically returns JSON error responses for AJAX/JSON requests.
- **Custom Hooks:** Easily customize data preparation and error handling with overridable methods.
- **Seamless Integration:** Works with Lightpack’s DI container, session, and redirect systems.
- **Session Flash**: Validation errors and current request input are automatically flashed to the session for easy display in views.
- Use **old()** and **error()** methods to work with sticky forms and displaying error messages.

---

### How It Works

1. **Extend FormRequest:** Create your own request classes by extending `Lightpack\Http\FormRequest`.
2. **Define Rules:** Implement the `rules()` method to return your validation rules.
   - **Rule Resolution:** Your `rules()` method is called via the container.
   - You can typehint dependencies you would like to get injected by the framework.
3. **Automatic Bootstrapping:** Lightpack boots your FormRequest, runs validation, and handles errors or passes control to your controller.
4. On successful validation, controller method executes further.
5. **On Failure:**
  - **AJAX/JSON:** Responds with HTTP 422 and a JSON error structure.
  - **Standard Request:** Redirects back with errors in the session.
  - Custom hooks (`beforeSend`, `beforeRedirect`) are available for advanced control.

### Example Usage

#### 1. Create a FormRequest

```php
php console create:request RegisterUserRequest
```

Then implement the `rules()` method. For example:

```php
namespace App\Requests;

use Lightpack\Http\FormRequest;

class RegisterUserRequest extends FormRequest
{
    protected function rules()
    {
         $this->validator
            ->field('name')
            ->required()
            ->max(255);

         $this->validator
            ->field('email')
            ->required()
            ->email()
            ->custom(new UniqueEmailRule, UniqueEmailRule::MESSAGE);

         $this->validator
            ->field('password')
            ->required()
            ->min(6)
            ->max(25);

         $this->validator
            ->field('confirm_password')
            ->required()
            ->same('password');
    }
}
```

#### 2. Use in Controller

Typehint the request class as dependency in your controller's method:

```php
public function register(RegisterUserRequest $request)
{
   // If validation passes, you reach here!

    $data = $request->all();

   // Proceed with user registration...
}
```

### Overridable Hooks

You can customize the request lifecycle by overriding these methods:

- `protected function data()`: Prepare or mutate request data before validation.
- `protected function beforeSend()`: Run logic before sending a JSON error response.
- `protected function beforeRedirect()`: Run logic before redirecting on validation failure.

Override any of the following in your FormRequest:

```php
protected function data()
{
   // Manipulate request input data before validation
}

protected function beforeSend()
{
   // Add custom headers or logging before JSON error response
}

protected function beforeRedirect()
{
   // Custom logic before redirecting on failure
}
```

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
- `email()` — Valid email address
- `url()` — Valid URL
- `ip()` / `ip('v6')` - Valid IPv4 or IPv6 address

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
- `unique()` — All array values must be unique

### Comparison Rules
- `same($field)` — Must match another field
- `different($field)` — Must not match another field

### Conditional Rules
- `requiredIf($field, $value)` — Required when another field has specific value
- `requiredWith($fields)` — Required when any of the specified fields are present
- `requiredWithout($fields)` — Required when none of the specified fields are present
- `requiredUnless($field, $value)` — Required unless another field has specific value

### Database Rules
- `dbUnique($table, $columns, $ignoreId = null, $idColumn = 'id')` — Check database uniqueness
- `exists($table, $columns = null, $where = [])` — Value must exist in database table

### File & Image Rules
- `file()` — Valid file upload
- `fileSize($size)` — Max file size (e.g. '2M', '500K')
- `fileType($types)` — Allowed MIME types
- `fileExtension($exts)` — Allowed extensions
- `files($min = null, $max = null)` — Multiple files
- `image($options)` — Image validation (width, height, ratio)

### Password Strength Rules
- `hasUppercase()` — Must contain at least one uppercase letter
- `hasLowercase()` — Must contain at least one lowercase letter
- `hasNumber()` — Must contain at least one numeric digit
- `hasSymbol()` — Must contain at least one special character

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

**Wildcard Behavior:**
- Wildcards validate each item in an array
- Supports deeply nested paths: `companies.*.employees.*.email`
- Empty arrays: If you need to validate that an array has items, validate the parent field with `array($min)` before using wildcards
- Non-array values: Will fail with "Field must be an array" error

**Example with array validation:**
```php
$validator
    ->field('items')->required()->array(1)  // Must have at least 1 item
    ->field('items.*')->required()->numeric()->min(1);  // Each item validated
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

## Example: Password Strength

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

## Nested & Conditional Validation

### Nested Data Validation

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

### Conditional Validation

**`requiredIf()` - Required when another field has specific value:**

```php
// Reason required when status is rejected
$validator
    ->field('status')->required()->in(['pending', 'approved', 'rejected'])
    ->field('reason')->requiredIf('status', 'rejected')->min(20);

// Company name required for business accounts
$validator
    ->field('account_type')->required()->in(['personal', 'business'])
    ->field('company_name')->requiredIf('account_type', 'business');

// Works with nested fields
$validator
    ->field('user.type')->required()
    ->field('company_details.name')->requiredIf('user.type', 'business');
```

**`requiredWith()` - Required when other fields are present:**

```php
// Phone required if country code is provided
$validator
    ->field('country_code')->string()
    ->field('phone')->requiredWith('country_code')->numeric();

// Address fields work together
$validator
    ->field('city')->string()
    ->field('state')->string()
    ->field('address')->requiredWith(['city', 'state']);
```

**`requiredWithout()` - Required when other fields are NOT present:**

```php
// Email required if phone is not provided (at least one contact method)
$validator
    ->field('email')->requiredWithout('phone')->email()
    ->field('phone')->requiredWithout('email')->numeric();

// Billing address required if not using saved address
$validator
    ->field('billing_address')->requiredWithout('use_saved_address');
```

**`requiredUnless()` - Required unless another field has specific value:**

```php
// Shipping address required unless same as billing
$validator
    ->field('same_as_billing')->bool()
    ->field('shipping_address')->requiredUnless('same_as_billing', true);

// Reason required unless status is approved
$validator
    ->field('status')->required()->in(['approved', 'rejected', 'pending'])
    ->field('reason')->requiredUnless('status', 'approved')->min(10);
```

### Database Uniqueness Validation

Use `dbUnique()` to check if values are unique in the database:

**Single Column Uniqueness:**

```php
// Email must be unique in users table
$validator
    ->field('email')
    ->required()
    ->email()
    ->dbUnique('users', 'email');

// Username must be unique
$validator
    ->field('username')
    ->required()
    ->alphaNum()
    ->dbUnique('users', 'username');
```

**Composite Uniqueness:**

```php
// Email must be unique per organization
$validator
    ->field('email')
    ->required()
    ->email()
    ->dbUnique('users', ['email', 'organization_id']);

// Slug must be unique per category
$validator
    ->field('slug')
    ->required()
    ->slug()
    ->dbUnique('posts', ['slug', 'category_id']);

// SKU must be unique per warehouse
$validator
    ->field('sku')
    ->required()
    ->dbUnique('inventory', ['sku', 'warehouse_id']);
```

**Ignoring Records (For Updates):**

```php
// Ignore current user when updating email
$validator
    ->field('email')
    ->required()
    ->email()
    ->dbUnique('users', 'email', ignoreId: $userId);

// Ignore current post when updating slug
$validator
    ->field('slug')
    ->required()
    ->slug()
    ->dbUnique('posts', ['slug', 'category_id'], ignoreId: $postId);
```

**Custom ID Column:**

```php
// For tables using UUID or custom primary keys
$validator
    ->field('code')
    ->required()
    ->dbUnique('products', 'code', ignoreId: $uuid, idColumn: 'uuid');
```

---

### Database Existence Validation

Use `exists()` to verify that values exist in the database (e.g., foreign key validation):

**Single Column Check:**

```php
// Category ID must exist in categories table
$validator
    ->field('category_id')
    ->required()
    ->int()
    ->exists('categories', 'id');

// User ID must exist
$validator
    ->field('user_id')
    ->required()
    ->exists('users', 'id');

// Email must exist (for login/password reset)
$validator
    ->field('email')
    ->required()
    ->email()
    ->exists('users', 'email');
```

**With Additional Conditions:**

```php
// Category must exist AND be active
$validator
    ->field('category_id')
    ->required()
    ->exists('categories', 'id', where: ['status' => 'active']);

// User must exist and not be banned
$validator
    ->field('user_id')
    ->required()
    ->exists('users', 'id', where: [
        'status' => 'active',
        'banned' => false
    ]);

// Product must exist in specific warehouse
$validator
    ->field('product_id')
    ->required()
    ->exists('products', 'id', where: ['warehouse_id' => $warehouseId]);
```

**Composite Column Check:**

```php
// Check if SKU exists in specific warehouse
$validator
    ->field('sku')
    ->required()
    ->exists('inventory', ['sku', 'warehouse_id']);

// Validates: SELECT COUNT(*) FROM inventory 
//            WHERE sku = ? AND warehouse_id = ?

// Input data must contain both fields:
$validator->setInput([
    'sku' => 'PROD-001',
    'warehouse_id' => 5
]);
```

**Default Column (uses field name):**

```php
// If no column specified, uses the field name
$validator
    ->field('id')
    ->required()
    ->exists('categories'); // Checks 'id' column automatically
```

**Practical Examples:**

```php
// Order form validation
$validator
    ->field('customer_id')
    ->required()
    ->exists('customers', 'id', where: ['status' => 'active'])
    
    ->field('product_id')
    ->required()
    ->exists('products', 'id', where: ['in_stock' => true])
    
    ->field('shipping_method')
    ->required()
    ->exists('shipping_methods', 'code', where: ['enabled' => true]);

// Assignment validation
$validator
    ->field('user_id')
    ->required()
    ->exists('users', 'id', where: ['role' => 'employee'])
    
    ->field('project_id')
    ->required()
    ->exists('projects', 'id', where: ['status' => 'active']);
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