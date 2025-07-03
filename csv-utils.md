# CSV Utility

Lightpack’s `Csv` utility provides a robust, memory-efficient, and highly flexible interface for reading, writing, streaming, and transforming CSV data. It is designed for both small and very large files, with a focus on practical ETL, data import/export, and reporting workflows.

---

## Features
- **Memory-efficient**: Reads large files with generators, never loads all rows into memory.
- **Flexible mapping**: Rename columns, transform values, or apply callables per column.
- **Type casting**: Automatically cast columns to `int`, `float`, `bool`, or `date` (timestamp).
- **Column exclusion**: Remove sensitive or unwanted columns.
- **Validation**: Row-level validation with three error handling modes (`skip`, `collect`, `fail`).
- **Streaming**: Stream CSV data to output (e.g., HTTP response) with consistent header order.
- **Customizable**: Set delimiter, enclosure, and escape characters.
- **Header order**: Control and cache column order for consistent output.

---

## Basic Usage

### Reading CSV Files

```php
use Lightpack\Utils\Csv;

$csv = new Csv();

// Read with headers (default)
foreach ($csv->read('users.csv') as $row) {
    echo $row['name'];
}

// Read without headers (returns indexed arrays)
foreach ($csv->read('data.csv', false) as $row) {
    echo $row[0];
}
```

### Writing CSV Files

```php
$data = [
    ['name' => 'John', 'age' => 25],
    ['name' => 'Jane', 'age' => 30],
];

// Write with auto-detected headers
$csv->write('users.csv', $data);

// Write with explicit header order
$csv->write('users.csv', $data, ['name', 'age']);

// Write from generator or database cursor
$users = User::query()->cursor();
$csv->write('users.csv', $users, ['id', 'name', 'email']);
```

---

## Data Transformation

### Column Mapping & Value Transformation

- **Rename columns** (e.g., `user_id` → `id`) or **transform values** (e.g., uppercase names):

```php
// Rename columns and apply callables
$csv->map([
    'user_id' => 'id',               // Rename 'user_id' to 'id'
    'user_name' => 'name',           // Rename 'user_name' to 'name'
    'salary' => fn($v) => (float)$v, // Transform 'salary' to float
    'name' => fn($v) => strtoupper($v), // Uppercase names
]);
```

- **Mapping is bidirectional:** When writing, mapped headers are used; when reading, output keys are the mapped names.

### Type Casting

- **Supported types:** `int`, `float`, `bool`, `date` (as UNIX timestamp)
- **Works on read and write**

```php
$csv->casts([
    'age' => 'int',
    'active' => 'bool',
    'joined' => 'date', // Converts to timestamp on read, to Y-m-d H:i:s on write
]);
```

### Excluding Columns

- **Remove unwanted columns:**

```php
$csv->exclude(['password', 'token']);
```

### Chaining Transformations

- All methods are chainable:

```php
$csv->map(['user_id' => 'id'])
    ->casts(['id' => 'int'])
    ->exclude(['password'])
    ->write('users.csv', $data, ['user_id', 'name']);
```

---

## Validation

### Row Validation (with error handling modes)

- **Provide a closure:** Return `true`, `false`, string, or array of errors.
- **Modes:**
  - `'skip'` (default): skip invalid rows
  - `'collect'`: include all rows, collect errors
  - `'fail'`: throw on first invalid row

```php
$csv->validate(function($row) {
    if ($row['age'] < 18) return 'Must be 18 or older';
    return true;
});

$csv->validate(function($row) {
    $errors = [];
    if (!is_numeric($row['salary'])) $errors[] = 'Salary must be numeric';
    if ($row['salary'] < 0) $errors[] = 'Salary cannot be negative';
    return $errors;
}, 'collect');
```

- **With Lightpack Validator:**

```php
use Lightpack\Validation\Validator;

$csv->validate(function($row) {
    $validator = new Validator();
    $validator->field('email')->required()->email();
    $validator->setInput($row);
    $result = $validator->validate();
    return $result->passes() ? true : $result->getErrors();
}, 'collect');
```

- **Get errors:**

```php
$errors = $csv->getErrors();
```

---

## Row Limits and Processing Control

### Maximum Rows (hard limit)

- **Throws if file exceeds limit:**

```php
$csv->max(1000)->read('large.csv');
```

### Processing Limit (soft limit)

- **Only process first N rows:**

```php
$csv->limit(500)->read('large.csv');
```

---

## Streaming Output

- **Stream large datasets directly to output (e.g., HTTP response):**
- **Headers are written only once; column order is cached.**

```php
$csv->map(['Name' => 'name', 'Email' => 'email']);

// First stream call writes headers
$csv->stream([
    ['name' => 'John', 'email' => 'john@example.com'],
], ['Name', 'Email']);

// Subsequent calls append rows (no headers)
$csv->stream([
    ['name' => 'Jane', 'email' => 'jane@example.com'],
]);
```

---

## Header Order and Consistency

- **Specify header order for write/stream:**

```php
$csv->map([
    'User ID' => 'id',
    'Full Name' => 'name',
    'Email Address' => 'email',
    'Age' => 'age',
]);

$csv->write('users.csv', $data, [
    'Email Address', // first column
    'Age',           // second
    'Full Name',     // third
    'User ID',       // last
]);
```

- **Order is preserved on subsequent stream calls.**

---

## Error Handling

- **Throws `RuntimeException` for:**
  - Unreadable files
  - Unwritable directories
  - Exceeding `max()`
  - Invalid arguments (e.g., negative limits)
- **Validation errors:**
  - Collected via `getErrors()` if `'collect'` mode
  - Skipped or thrown otherwise

---

## Advanced Usage & Tips

- **Generators and cursors:** Use generators for both reading and writing to handle huge files efficiently.
- **Custom delimiter, enclosure, escape:**

```php
$csv->setDelimiter(';')
    ->setEnclosure("'")
    ->setEscape('\\');
```

- **Streaming to HTTP:** Use `php://output` as the file path in `write()` for direct HTTP output, or use `stream()` for chunked output.
- **Chaining:** All configuration and transformation methods are chainable.
- **Header caching:** Once headers are set (by write or stream), order is preserved for all subsequent rows.

---

## Practical Examples

### Export with Transformations and Column Order

```php
$data = [
    ['id' => 1, 'name' => 'JOHN', 'age' => 25],
    ['id' => 2, 'name' => 'JANE', 'age' => 30],
];

$csv->map([
    'user_id' => 'id',
    'user_name' => 'name',
])
->casts(['age' => 'int'])
->write('users.csv', $data, ['user_id', 'user_name', 'age']);
// Output columns: user_id,user_name,age
```

### Import, Validate, and Collect Errors

```php
$csv->validate(function($row) {
    $errors = [];
    if ($row['quantity'] <= 0) $errors[] = 'Quantity must be positive';
    if ($row['price'] <= 0) $errors[] = 'Price must be positive';
    return $errors;
}, 'collect')
->casts([
    'quantity' => 'int',
    'price' => 'float',
])
->read('orders.csv');

foreach ($csv->getErrors() as $error) {
    Log::error($error);
}
```

---

### Generate Reports

```php
$csv = new Csv();

// Get sales data
$sales = ProductSale::query
    ->select('product_id', 'quantity', 'price', 'created_at')
    ->all()
    ->toArray();

// Generate sales report
$csv->map([
        'created_at' => fn($date) => date('Y-m-d', strtotime($date)),
        'total' => fn($row) => $row['quantity'] * $row['price']
    ])
    ->casts([
        'quantity' => 'int',
        'price' => 'float',
        'total' => 'float'
    ])
    ->write('sales_report.csv', $sales);
```

## ETL (Extract, Transform, Load) Operations

The CSV utility class is powerful enough to handle ETL operations, making it perfect for data pipeline processing.

### Basic ETL Example

```php
use Lightpack\Utils\Csv;

$csv = new Csv();

// Extract: Get users from database
$users = User::query()
    ->where('status', 'active')
    ->all()
    ->toArray();

// Transform: Clean and format data
$csv->map([
        // Format dates
        'created_at' => fn($date) => date('Y-m-d', strtotime($date)),
        // Calculate full name
        'full_name' => fn($row) => $row['first_name'] . ' ' . $row['last_name'],
        // Format currency
        'salary' => fn($amount) => number_format($amount, 2)
    ])
    ->casts([
        'id' => 'int',
        'age' => 'int',
        'is_active' => 'bool'
    ])
    ->except(['password', 'remember_token'])
    ->validate(function($row) {
        $errors = [];
        if ($row['age'] < 18) $errors[] = 'Must be adult';
        if ($row['salary'] < 0) $errors[] = 'Invalid salary';
        return $errors;
    }, 'skip');

// Load: Export to CSV
$csv->write('processed_users.csv', $users);
```

---