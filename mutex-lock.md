# Lock Utility

The **Lock** utility provides a simple way to implement **mutex locks** in your application. It's ideal for ensuring that certain operations run exclusively. It uses Lightpack's **Cache** system with these key features:

- Atomic operations for reliability
- Automatic expiration via TTL
- Works with any cache driver
- No external dependencies

You may create an instance of **Lock** class:

```php
$lock = new Lightpack\Utils\Lock;
```
Or simply call the utility function `lock()` which returns **Lock** class instance. 

The two most important methods it exposes is: `acquire()` and `release()`.

## Basic Usage

```php
// Try to acquire a lock
if (lock()->acquire('daily-report')) {
    try {
        // generate daily report
    } finally {
        // Always release the lock
        lock()->release('daily-report');
    }
}
```

## Lock Duration

By default, locks expire after 60 seconds. You can specify a custom duration:

```php
// Lock for 5 minutes
if (lock()->acquire('backup-process', 300)) {
    // ...
}
```

## Practical Scenarios

### 1. Scheduled Tasks

Prevent multiple instances of a scheduled task from running simultaneously:

```php
class DailyReportJob
{
    public function run()
    {
        // Only one instance can run at a time
        if (lock()->acquire('daily-report') == false) {
            echo "Report generation already in progress\n";
            return;
        }
        
        try {
            // Your report generation logic here
        } finally {
            lock()->release('daily-report');
        }
    }
}
```

### 2. Restricting multiple payment attempts

Implement a simple concurrency limit for API endpoints:

```php
class PaymentController
{
    public function processPayment($orderId)
    {
        $lockKey = "order:{$orderId}:payment";
        
        if (lock()->acquire($lockKey, 30) == false) {
            throw new Exception('Payment already in progress');
        }
        
        try {
            // Process payment
        } finally {
            lock()->release($lockKey);
        }
    }
}
```

### 3. Cache Regeneration

Prevent cache stampede when multiple requests try to regenerate expired cache:

```php
class ProductCache
{
    public function getProduct($id)
    {
        $data = cache()->get("product:{$id}");
        
        if ($data) {
            return $data;
        }
        
        // Prevent multiple regenerations
        if (lock()->acquire("rebuild:product:{$id}", 10) == false) {
            // Wait and try getting from cache again
            sleep(1);
            return $this->cache->get("product:{$id}");
        }
        
        try {
            $data = $this->fetchProductData($id);
            cache()->set("product:{$id}", $data, 3600);
            return $data;
        } finally {
            lock()->release("rebuild:product:{$id}");
        }
    }
}
```

## Best Practices

1. **Always Release Locks**
   - Use try/finally to ensure locks are released
   - Set appropriate TTL to auto-expire stale locks

2. **Lock Naming**
   - Use descriptive lock names
   - Add prefixes for different contexts
   - Include relevant IDs in lock names

3. **Lock Duration**
   - Set TTL slightly longer than expected operation time
   - Consider network latency and processing time
   - Don't set extremely long durations

4. **Error Handling**
   - Always handle lock acquisition failure gracefully
   - Have a fallback plan when locks can't be acquired
   - Log lock-related issues for monitoring

---