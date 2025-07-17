# Caching

A cache is a store where you put data to enhance application performance by reducing frequent computations or database queries.

For example, if your store displays product categories with counts, querying the database every time is slow. Instead, cache the result and reuse it for future requests.

Lightpack provides a simple, flexible cache library with multiple drivers and a clean API.

## Quick Start

Lightpack comes pre-configured with a default **file-based** cache service. Just use:

```php
cache()->set();
cache()->get();
cache()->has();
cache()->delete();
cache()->flush();
cache()->forever();
cache()->remember();
cache()->rememberForever();
```

<p class="tip">NOTE: Set appropriate write permissions on the <code>storage</code> directory for file cache.</p>

---

### set()

Store an item in the cache.  
**Parameters:** `key`, `value`, `seconds` (expiry in seconds, not minutes).

```php
cache()->set('name', 'Bob', 300); // 5 minutes
```
- Pass `0` as seconds for a “forever” cache (5 years).

### get()

Retrieve an item from cache by key. Returns `null` if not found or expired.

```php
$name = cache()->get('name'); // Bob or null
```

### has()

Check if a key exists in the cache.

```php
if (cache()->has('name')) { ... }
```

### delete()

Remove an item from the cache.

```php
cache()->delete('name');
```

### flush()

Clear all items from the cache store.

```php
cache()->flush();
```

### forever()

Store an item “forever” (actually 5 years).

```php
cache()->forever('site_theme', 'Marble');
```

### remember()

Retrieve an item if present, otherwise compute, store, and return it.

```php
$value = cache()->remember('expensive', 300, function() {
    return computeExpensiveThing();
});
```

### rememberForever()

Like `remember()`, but stores the value “forever”.

```php
$value = cache()->rememberForever('expensive', function() {
    return computeExpensiveThing();
});
```

---

## Manual Configuration

You can manually configure the cache provider:

```php
use Lightpack\Cache\Cache;
use Lightpack\Cache\Drivers\FileDriver;

$driver = new File(DIR_STORAGE . '/cache');
$cache = new Cache($driver);
```

Now you can access cache methods as usual.

```php
$cache->set('name', 'Bob', 5);
$cache->get('name'); // Bob
```

## Available Drivers

Lightpack provides several cache drivers:

- **FileDriver**: Stores cache in files (default).
- **ArrayDriver**: Stores cache in memory (non-persistent, useful for testing).
- **NullDriver**: Disables cache (all operations are no-ops).
- **DatabaseDriver**: Stores cache in a database table (requires schema).
- **RedisDriver**: Stores cache in Redis (supports key prefixing).

You can view `config/cache.php` file for cache related configurations.

### Database Migration

If using database as cache driver, you need to migrate a new table for storing cache entries.

```php
php console create:migration create_table_cache
```

Update and run the migration code:

```php
return new class extends Migration
{
    public function up(): void
    {
        $this->create('cache', function (Table $table) {
            $table->varchar('key', 255)->primary();
            $table->column('value')->type('longtext');
            $table->column('expires_at')->type('int')->attribute('UNSIGNED');
            $table->index('expires_at', 'idx_cache_expiry');
        });
    }

    public function down(): void
    {
        $this->drop('cache');
    }
};
```

---