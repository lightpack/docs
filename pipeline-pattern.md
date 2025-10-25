# Pipeline Pattern

The **Pipeline** pattern provides a clean way to pass data through a series of operations (pipes). Each pipe transforms the data and passes it to the next pipe. It's ideal for multi-step processes like order processing, data validation, file uploads, and request handling.

Key features:
- Sequential data transformation
- Automatic dependency injection
- Mix closures, instances, and class names
- Clean, testable code
- Single responsibility per pipe

You can create a pipeline using the `pipeline()` helper function:

```php
$result = pipeline($data)
    ->through([
        ValidateData::class,
        TransformData::class,
        SaveData::class,
    ])
    ->run();
```

## Basic Usage

### Simple Transformation

```php
$result = pipeline(10)
    ->through([
        fn($n) => $n * 2,  // 10 * 2 = 20
        fn($n) => $n + 5,  // 20 + 5 = 25
    ])
    ->run();

// $result = 25
```

### String Processing

```php
$result = pipeline('hello world')
    ->through([
        fn($str) => strtoupper($str),           // HELLO WORLD
        fn($str) => str_replace(' ', '_', $str), // HELLO_WORLD
    ])
    ->run();

// $result = 'HELLO_WORLD'
```

## Using Pipe Classes

Instead of closures, you can use dedicated pipe classes for better organization and reusability.

### Creating a Pipe Class

```php
<?php

namespace App\Pipes;

class UpperCasePipe
{
    public function __invoke($value)
    {
        return strtoupper($value);
    }
}
```

### Using the Pipe

```php
use App\Pipes\UpperCasePipe;

$result = pipeline('hello')
    ->through([
        UpperCasePipe::class,  // Resolved from container
    ])
    ->run();

// $result = 'HELLO'
```

## Dependency Injection

Pipe classes support automatic dependency injection through the container.

### Pipe with Dependencies

```php
<?php

namespace App\Pipes\Order;

use App\Services\InventoryService;

class ValidateInventory
{
    protected $inventoryService;
    
    // Dependencies automatically injected
    public function __construct(InventoryService $inventoryService)
    {
        $this->inventoryService = $inventoryService;
    }
    
    public function __invoke($order)
    {
        foreach ($order->items as $item) {
            if (!$this->inventoryService->isAvailable($item)) {
                throw new \Exception('Product out of stock');
            }
        }
        
        return $order;
    }
}
```

### Using It

```php
$order = pipeline($order)
    ->through([
        ValidateInventory::class,  // InventoryService injected automatically
    ])
    ->run();
```

## Real-World Examples

### Order Processing Pipeline

```php
<?php

namespace App\Controllers;

use App\Pipes\Order\ValidateInventory;
use App\Pipes\Order\ApplyDiscounts;
use App\Pipes\Order\CalculateTax;
use App\Pipes\Order\ProcessPayment;
use App\Pipes\Order\ReserveInventory;
use App\Pipes\Order\SendConfirmationEmail;

class OrderController
{
    public function checkout($request)
    {
        $order = new Order;
        $order->customer_id = $request->customer_id;
        $order->save();
        
        $order = pipeline($order)
            ->through([
                ValidateInventory::class,
                ApplyDiscounts::class,
                CalculateTax::class,
                ProcessPayment::class,
                ReserveInventory::class,
                SendConfirmationEmail::class,
            ])
            ->run();
            
        return response()->json($order);
    }
}
```

### File Upload Pipeline

```php
$file = pipeline($request->file('document'))
    ->through([
        ValidateFileType::class,
        ValidateFileSize::class,
        ScanForVirus::class,
        OptimizeFile::class,
        UploadToS3::class,
        CreateDatabaseRecord::class,
    ])
    ->run();
```

### Data Validation Pipeline

```php
$data = pipeline($request->all())
    ->through([
        SanitizeInput::class,
        ValidateEmail::class,
        CheckDuplicateEmail::class,
        ValidatePassword::class,
        NormalizePhone::class,
    ])
    ->run();

$user = User::create($data);
```

### Lead Scoring Pipeline (CRM)

```php
$lead = pipeline($lead)
    ->through([
        ScoreByCompanySize::class,    // +20 if > 100 employees
        ScoreByIndustry::class,       // +15 if tech
        ScoreByEngagement::class,     // +10 per email open
        ScoreByJobTitle::class,       // +25 if C-level
        AssignSalesRep::class,        // Auto-assign if score > 80
    ])
    ->run();
```

## Mixing Pipe Types

You can mix class names, instances, and closures in the same pipeline:

```php
$result = pipeline($data)
    ->through([
        ValidateData::class,              // Class name (DI)
        new TransformData($config),       // Instance
        fn($data) => $data['result'],     // Closure
        ProcessData::class,               // Class name (DI)
    ])
    ->run();
```

## Conditional Pipes

You can conditionally add pipes based on your business logic by building the pipes array dynamically:

### Basic Conditional Pipes

```php
// Build pipes array conditionally
$pipes = [
    ValidateInventory::class,
];

if ($order->hasCoupon()) {
    $pipes[] = ApplyDiscount::class;
}

if ($order->isInternational()) {
    $pipes[] = CalculateInternationalTax::class;
} else {
    $pipes[] = CalculateTax::class;
}

$pipes[] = ProcessPayment::class;

// Run pipeline with conditional pipes
$result = pipeline($order)
    ->through($pipes)
    ->run();
```

### Order Type Based Pipes

```php
$pipes = [ValidateInventory::class];

// Different pipes for different order types
switch ($order->type) {
    case 'wholesale':
        $pipes[] = ApplyWholesaleDiscount::class;
        // No tax for wholesale
        break;
        
    case 'international':
        $pipes[] = ApplyStandardDiscount::class;
        $pipes[] = CalculateInternationalTax::class;
        $pipes[] = CheckCustomsRestrictions::class;
        break;
        
    default:
        $pipes[] = ApplyStandardDiscount::class;
        $pipes[] = CalculateTax::class;
}

$pipes[] = ProcessPayment::class;

$result = pipeline($order)->through($pipes)->run();
```

### User Role Based Pipes

```php
$pipes = [ValidateData::class];

// Add pipes based on user role
if ($user->isAdmin()) {
    $pipes[] = SkipApproval::class;
} else {
    $pipes[] = RequireApproval::class;
    $pipes[] = NotifyManager::class;
}

$pipes[] = SaveData::class;

$result = pipeline($data)->through($pipes)->run();
```

### Feature Flag Based Pipes

```php
$pipes = [ProcessOrder::class];

// Add pipes based on feature flags
if (feature('new_discount_engine')) {
    $pipes[] = ApplyNewDiscounts::class;
} else {
    $pipes[] = ApplyLegacyDiscounts::class;
}

if (feature('fraud_detection')) {
    $pipes[] = CheckFraud::class;
}

$pipes[] = CompleteOrder::class;

$result = pipeline($order)->through($pipes)->run();
```

### Using Helper Method

For complex conditional logic, extract to a helper method:

```php
class OrderPipeline
{
    public static function getPipes(Order $order): array
    {
        $pipes = [ValidateInventory::class];
        
        // Discount logic
        if ($order->hasCoupon()) {
            $pipes[] = ApplyDiscount::class;
        }
        
        // Tax logic
        if ($order->isInternational()) {
            $pipes[] = CalculateInternationalTax::class;
            $pipes[] = CheckCustomsRestrictions::class;
        } else {
            $pipes[] = CalculateTax::class;
        }
        
        // Payment
        $pipes[] = ProcessPayment::class;
        
        // Post-processing
        if ($order->requiresShipping()) {
            $pipes[] = CalculateShipping::class;
            $pipes[] = NotifyWarehouse::class;
        }
        
        return $pipes;
    }
}

// Usage
$result = pipeline($order)
    ->through(OrderPipeline::getPipes($order))
    ->run();
```

This approach keeps your controller clean while maintaining flexibility.

## Error Handling

Pipes can throw exceptions to stop the pipeline:

```php
class ValidateInventory
{
    public function __invoke($order)
    {
        if (!$this->hasStock($order)) {
            throw new OutOfStockException('Insufficient inventory');
        }
        
        return $order;
    }
}
```

Usage with error handling:

```php
try {
    $order = pipeline($order)
        ->through([
            ValidateInventory::class,
            ProcessPayment::class,
        ])
        ->run();
} catch (OutOfStockException $e) {
    return response()->json(['error' => $e->getMessage()], 400);
}
```

## Testing Pipes

Each pipe can be tested independently:

```php
<?php

namespace Tests\Pipes;

use Tests\TestCase;
use App\Pipes\Order\ValidateInventory;
use App\Models\Order;
use App\Models\Product;

class ValidateInventoryTest extends TestCase
{
    public function testValidatesSuccessfully()
    {
        // Arrange
        $product = new Product;
        $product->stock = 100;
        $product->save();
        
        $order = new Order;
        $order->save();
        $order->items()->create([
            'product_id' => $product->id,
            'quantity' => 10,
        ]);
        
        // Act
        $pipe = new ValidateInventory($this->inventoryService);
        $result = $pipe($order);
        
        // Assert
        $this->assertInstanceOf(Order::class, $result);
    }
    
    public function testThrowsExceptionWhenOutOfStock()
    {
        // Test exception case
        $this->expectException(OutOfStockException::class);
        
        // ... test code
    }
}
```

## Best Practices

### Keep Pipes Small

Each pipe should have a single responsibility:

```php
// ✅ Good - single responsibility
class ValidateInventory { ... }
class ProcessPayment { ... }
class SendEmail { ... }

// ❌ Bad - doing too much
class ProcessOrderAndSendEmailAndUpdateInventory { ... }
```

### Make Pipes Reusable

Design pipes to work in different contexts:

```php
// ✅ Good - reusable
class ValidateEmail
{
    public function __invoke($data)
    {
        if (!filter_var($data['email'], FILTER_VALIDATE_EMAIL)) {
            throw new ValidationException('Invalid email');
        }
        return $data;
    }
}

// Can be used in multiple pipelines:
// - User registration
// - Profile update
// - Contact form
```

### Use Dependency Injection

Let the container handle dependencies:

```php
// ✅ Good - uses DI
class ProcessPayment
{
    public function __construct(PaymentGateway $gateway)
    {
        $this->gateway = $gateway;
    }
}

// ❌ Bad - creates dependencies
class ProcessPayment
{
    public function __invoke($order)
    {
        $gateway = new PaymentGateway(); // Hard to test
    }
}
```

### Return the Passable

Always return the data being passed through:

```php
// ✅ Good
public function __invoke($order)
{
    $order->validated = true;
    return $order;  // Return it!
}

// ❌ Bad
public function __invoke($order)
{
    $order->validated = true;
    // Forgot to return - breaks pipeline!
}
```

## When to Use Pipeline

**Use Pipeline for:**
- ✅ Multi-step processes (order checkout, file upload)
- ✅ Data transformation chains
- ✅ Validation workflows
- ✅ Sequential operations that need to be reordered
- ✅ Operations that benefit from testing in isolation

**Don't use Pipeline for:**
- ❌ Simple single-step operations
- ❌ Operations that don't transform data
- ❌ When a simple function call is clearer

## Configuration-Driven Pipelines

You can make pipelines configurable:

```php
// config/pipelines.php
return [
    'order_processing' => [
        'standard' => [
            ValidateInventory::class,
            ApplyDiscounts::class,
            CalculateTax::class,
            ProcessPayment::class,
        ],
        'wholesale' => [
            ValidateInventory::class,
            ApplyWholesaleDiscounts::class,
            ProcessPayment::class,  // No tax for wholesale
        ],
    ],
];

// Usage
$pipes = config("pipelines.order_processing.{$order->type}");

$order = pipeline($order)
    ->through($pipes)
    ->run();
```

This allows you to change business logic without code changes.

## API Reference

### pipeline($passable)

Create a new pipeline instance.

**Parameters:**
- `$passable` (mixed) - The data to pass through the pipeline

**Returns:** `Pipeline` instance

### through(array $pipes)

Set the pipes to pass data through.

**Parameters:**
- `$pipes` (array) - Array of class names, instances, or closures

**Returns:** `Pipeline` instance (chainable)

### run()

Execute the pipeline and return the result.

**Returns:** The transformed passable

**Example:**
```php
$result = pipeline($data)
    ->through([...])
    ->run();
```
