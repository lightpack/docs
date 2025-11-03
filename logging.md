# Logging

**Lightpack** ships with a PSR-3 compatible logger that is already configured as a service for you. You can access the logger instance as service using `logger()` which gives an instance of `Lightpack\Logger\Logger`. 

This class exposes following logging methods:

```php
logger()->log();
logger()->info();
logger()->alert();
logger()->debug();
logger()->notice();
logger()->warning();
logger()->critical();
logger()->emergency();
```

## Log Drivers

**Lightpack** ships with three log drivers:

- **`file`** - Size-based rotation (default)
- **`daily`** - Daily log files with date-based retention
- **`null`** - Disables logging

The default driver is `file`. You can change the driver by setting `LOG_DRIVER` in your `.env` file.

### Configuration

To customize log driver settings, run the following command to create a `config/logs.php` configuration file:

```cli
php console create:config --support=logs
```

### File Driver (Size-Based Rotation)

The **file** driver rotates logs based on file size. When a log file reaches the maximum size, it's rotated with a timestamp and old logs are automatically cleaned up.

```env
LOG_DRIVER=file
```

**How it works:**
- Logs to: `storage/logs/lightpack.log`
- Rotates when file reaches `max_file_size` (default: 10MB)
- Keeps last `max_log_files` rotated files (default: 10)

**Example:**
```
storage/logs/
  ├── lightpack.log              (current, 8MB)
  ├── lightpack.20241103120000.log (10MB)
  └── lightpack.20241102180000.log (10MB)
```

### Daily Driver (Date-Based Rotation)

The **daily** driver creates a new log file each day and automatically removes logs older than the specified retention period.

**Configuration:**

```env
LOG_DRIVER=daily
```

**How it works:**
- Logs to: `storage/logs/lightpack-YYYY-MM-DD.log`
- Creates new file each day automatically
- Keeps logs for `days_to_keep` days (default: 7)
- Old logs deleted automatically

**Example:**
```
storage/logs/
  ├── lightpack-2024-11-03.log  (today)
  ├── lightpack-2024-11-02.log  (yesterday)
  └── lightpack-2024-11-01.log  (2 days ago)
```

### Null Driver (Disable Logging)

To **disable** logging completely, set the driver to `null`:

```env
LOG_DRIVER=null
```

This is useful for testing environments where you don't want log files created.

## Available Methods

All logging methods follow the PSR-3 standard and accept two parameters:

1. **`$message`** (string) - The log message
2. **`$context`** (array, optional) - Additional structured data

### log()

The generic logging method that accepts a log level as the first parameter.

```php
logger()->log('error', 'Database connection failed', [
    'host' => 'localhost',
    'port' => 3306,
]);
```

### emergency()

System is unusable. Requires immediate attention.

```php
logger()->emergency('Database server is down', [
    'error_code' => 2002,
    'attempted_at' => time(),
]);
```

**Use when:** Complete system failure, data loss, security breach.

### alert()

Action must be taken immediately.

```php
logger()->alert('Disk space critical: 95% full', [
    'disk' => '/dev/sda1',
    'available' => '500MB',
]);
```

**Use when:** Critical issues that need immediate attention but system still functioning.

### critical()

Critical conditions that should be addressed quickly.

```php
logger()->critical('Payment gateway timeout', [
    'gateway' => 'stripe',
    'transaction_id' => 'txn_123',
    'amount' => 99.99,
]);
```

**Use when:** Application component failures, critical business logic errors.

### error()

Runtime errors that don't require immediate action but should be logged and monitored.

```php
logger()->error('Failed to send email', [
    'recipient' => 'user@example.com',
    'subject' => 'Welcome Email',
    'smtp_error' => 'Connection refused',
]);
```

**Use when:** Recoverable errors, failed operations, exceptions.

### warning()

Exceptional occurrences that are not errors.

```php
logger()->warning('API rate limit approaching', [
    'current_usage' => 950,
    'limit' => 1000,
    'resets_at' => '2024-11-03 15:00:00',
]);
```

**Use when:** Deprecated API usage, poor use of API, things that aren't necessarily wrong.

### notice()

Normal but significant events.

```php
logger()->notice('User password changed', [
    'user_id' => 123,
    'ip_address' => '192.168.1.1',
]);
```

**Use when:** Significant business events, security-relevant actions.

### info()

Log events for informational purposes.

```php
logger()->info('User logged in', [
    'user_id' => 123,
    'username' => 'john_doe',
]);
```

**Use when:** General informational messages, tracking user actions.

### debug()

Detailed debug information for development.

```php
logger()->debug('Cache miss for key', [
    'key' => 'user_profile_123',
    'ttl' => 3600,
]);
```

**Use when:** Development debugging, detailed diagnostic information.

## Context Data

The `$context` array allows you to log structured data alongside your message. This is particularly useful for debugging and log analysis.

```php
logger()->error('Payment failed', [
    'user_id' => 123,
    'amount' => 99.99,
    'currency' => 'USD',
]);
```

---