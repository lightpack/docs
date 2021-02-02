# Caching

A cache is a store where you put some data to enhance application performance by reducing data computation. 

For example, suppose an online shopping store website displays all its categories with products count. Querying the database
to aggregate categories with products count
everytime the store's pages are displayed can become a huge performance bottleneck. So rather than querying that data on every page
request, you should store the query data in a cache to access it later on every page request. 

While caching in itself can introduce its own challenges, `Lightpack` provides you a cache library that can take some pain away when working with cache data.

## Quick start

`Lightpack` comes pre-configured with a default **file** based cache service which you can start using without much changes required. 

Following is the list of cache methods available:

```php
app('cache')->set();
app('cache')->get();
app('cache')->has();
app('cache')->delete();
app('cache')->flush();
```

<p class="tip">NOTE: You will have to set appropriate write permissions on <code>storage</code> directory in order to work with file cache.</p>

### set()

To store an item in the cache, call `set()` method. It takes item `key`, item
`value` and item expiry time in `minutes`.

This example stores in cache an item with key `name` and value `Bob` for `5 minutes`.

```php
app('cache')->set('name', 'Bob', 5);
```

### get()

To get an item from cache, call `get()` method passing it the item key. It returns
`null` if the item does not exist in the cache.

```php
app('cache')->get('name'); // Bob
```

### has()

To check if an item is in the cache, use `has()` method. This method return a `boolean` true or false. Pass it the item key to check for the item existence.

```php
app('cache')->has('name');
```

### delete()

To delete an item from the cache, call `delete()` method passing it the
the item key.

```php
app('cache')->delete('name');
```