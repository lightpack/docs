# Models

![ORM Model Overview](_media/orm/orm-model-overview.svg)

## Introduction to ORM Models

Lightpack ORM is an **Active Record** pattern implementation. Each model is a class directly corresponds to a single table in your database, and each instance of a model represents a single row within that table. The model not only holds data, but also encapsulates all the logic required to create, read, update, and delete (CRUD) records.

**Key Fundamentals of Active Record:**
- **Class-to-Table Mapping:** Each model class maps to a database table. For example, a `User` model maps to a `users` table.
- **Object-to-Row Mapping:** Each model instance represents a row in the corresponding table.
- **CRUD Operations:** Models provide methods to perform CRUD operations directly, such as `save()`, `find()`, `update()`, and `delete()`, without needing to write SQL manually.

The following sections will explore how to define models, establish relationships, and utilize the full capabilities of the ORM system.

---


Consider that you have a `products` table in your database.

```table
Table: products
-------------------------------------------------
id, name, size, color, status
-------------------------------------------------
```

Then you should define a `Product` model in <code>app/Models</code> folder.

## Defining Model

Fire this command to generate a model from your terminal inside your project root.

```terminal
php lucy create:model Product --table=products
```

This should have created `Product` model in `app/Models` folder.

```php
class Product extends Model
{
    protected $table = 'products';

    protected $primaryKey = 'id';

    protected $timestamps = false;
}
```

Defining your model in this manner gives you access to a number of utility
methods to deal with records in your `products` table.

## Performing CRUD

Once the model class is defined, performing **CRUD** operations per single database record becomes very easy. You do not need to manually wire-up raw SQL queries for:

- **(C)** creating a new record, 
- **(R)** reading an existing record, 
- **(U)** update an existing record, 
- **(D)** or delete an existing record.

### Create

Set properties on your new model and simply call the inherited method <code>insert()</code> to insert a new record.

```php
// Create the instance
$product = new Product;

// Set product properties
$product->name = 'ACME Shoes';
$product->size = 10;
$product->color = 'black'

// Create new product
$product->insert();
```
#### Last Insert ID

If your table's primary key is an **auto-incrementing** field, you can get the last insert id:

```php
$product->lastInsertId();
```

#### Manual Primary Key

If your table's primary key is a **non auto-incrementing** field, you must override the inherited model attribute `autoIncrements` to false.

```php
class Product extends Model
{
    protected $autoIncrements = false;
}
```

Now when calling `insert()` method, you must pass a unique primary key value for the new record to be created.

```php
// Create the instance
$product = new Product;

// Set product properties
$product->id = 'sku1000';
$product->name = 'ACME Shoes';
$product->size = 10;
$product->color = 'black'

// Create new product
$product->insert();
```


### Read

You can easily fetch a record by its **ID** when constructing the model. 

```php
$product = new Product(23);
```

Now you can access the column values as model properties.

```php
echo $product->title;
echo $product->size;
echo $product->color;
```

### Update

Use the `update()` method to update an existing record in the database table. You first need to instantiate the model using the primary key of the table.

```php
// Get an existing product having id: 23
$product = new Product(23);

// Set product properties to update
$product->name = 'ACME Footwear';
$product->size = 11;
$product->color = 'brown'

// Update the product
$product->update();
```

### Delete

Simply call the <code>delete()</code> passing it the id of the record to be deleted from database.

```php
(new Product)->delete(23);
```

If you already have an existing instance of model, you can call `delete()` method there too.

```php
$product = new Product(23);
$product->delete(); // passing id not required
```

## Timestamps

Consider **products** table:

```table
Table: products
-------------------------------------------------
id, name, price, created_at, updated_at
-------------------------------------------------
```

Setting `$timestamps` property to `true` automatically sets values for **created_at** and **updated_at** columns in your table. So
when you create a new product or update an existing product, you don't have to manually set values for these columns.

<p class="tip">In order for timestamps to work, the table must have <b>created_at</b> and <b>updated_at</b> columns both.</p>

## Refetch

Sometimes, the data in your existing model instance can become outdated—especially if changes are made to the database from somewhere else in your application. 

For an example, consider the scenario where the `timestamps` attribute in model class definition is set to **true**.

```php
class Product extends Model
{
    protected $timestamps = true;
}
```

In such case, performing an `insert()` or `update()` automatically sets **created_at** and **updated_at** columns. This happens as a side-effect of the framework's ORM implementation as convinience. 

```php
/**
 * created_at, updated_at column is set automatically
 */
$product->insert(); 
```

Now if you try this:

```php
echo $product->created_at; // null
echo $product->updated_at; // null
```

To ensure you are working with the latest data for the current record from the database, you can call `refetch()` method.

```php
$product = $product->refetch();
```

`$product` will contain the latest data. If the record was deleted or the primary key isn’t set, it will return `null`.

## Attribute Casting
Attribute casting converts model attributes to a specific type (like integer, boolean, array, or date) when you access them, and back to a database-friendly format when you save them.

### Supported Cast Types

| Cast Type           | Description & Example                                      |
|---------------------|-----------------------------------------------------------|
| `int` / `integer`   | Converts to integer: `'123'` → `123`                      |
| `float` / `double`  | Converts to float: `'123.45'` → `123.45`                  |
| `string`            | Converts to string: `123` → `'123'`                       |
| `bool` / `boolean`  | Converts to boolean: `'1'`, `1`, `'true'` → `true`        |
| `array` / `json`    | Converts JSON string to array and vice versa               |
| `date`              | Converts to `Y-m-d` string or from `DateTime`             |
| `datetime`          | Converts to `DateTime` object or from string              |
| `timestamp`         | Converts to Unix timestamp (int or string)                |

### Example Usage

Suppose your `User` model has a `settings` column that stores JSON, and a `created_at` column for timestamps. You can define casts like this:

```php
class User extends Model
{
    protected $casts = [
        'settings'   => 'array',
        'created_at' => 'datetime',
        'active'     => 'bool',
        'score'      => 'int',
    ];
}
```

Now, whenever you access `$user->settings`, you’ll get an array. `$user->created_at` will be a `DateTime` object, and so on.

### How Casting Works
- **On retrieval:** The value is converted to the specified type automatically.
- **On save:** The value is converted back (uncast) to a database-friendly format.
- **Null values:** Always remain `null`—no conversion is performed.
- **Unknown types:** The value is returned as-is.

### Common Pitfalls
- Make sure your database column type matches the cast (e.g., don’t cast a string column as an array unless it stores JSON).
- Invalid input (like malformed JSON or dates) will throw exceptions—handle these in your code if needed.

---

## Cloning a Model

There may be times when you want to create a new record in your database that’s almost identical to an existing one—without re-entering all the data. The `clone` method makes this easy: it creates a new model instance with the same attribute values as the original, but leaves out the primary key and timestamps, so you can safely save it as a new record.

This is especially useful for duplicating templates, copying products, or quickly creating similar entries.

**How it works:**  
- The new instance copies all attributes from the original, except for the primary key (`id`), `created_at`, and `updated_at` fields.
- You can also specify additional fields to exclude if needed.
- You can only clone an existing record (one that already exists in the database).

**Example:**  
Let’s say you want to duplicate a product but change its name:

```php
$original = new Product(23); // Load an existing product
$copy = $original->clone();  // Create a new instance with the same data

$copy->name = 'New Product Name';
$copy->insert(); // Save as a new product in the database
```

If you want to exclude more fields from being copied, just pass them as an array:

```php
$copy = $original->clone(['description', 'price']);
```

If you try to clone a model that doesn’t exist in the database, you’ll get an error.

## Tracking Unsaved Changes

When working with models, you may want to know if you’ve made changes that haven’t been saved to the database yet. Lightpack models make this easy with two helpful methods:

| Method     | What it does                                 | Example Output      |
|------------|----------------------------------------------|--------------------|
| isDirty()  | Checks if the model (or a specific field) has unsaved changes | true / false       |
| getDirty() | Lists all fields that have unsaved changes   | ['name', 'email']  |

### Why is this useful?
- **Save only when needed:** Avoid unnecessary database writes by saving only if something has changed.
- **User feedback:** Warn users if they try to leave a page with unsaved changes.
- **Debugging:** See exactly what’s different before saving.

### How to use

**Check if anything changed:**
```php
$user = User::find(1);
$user->name = 'New Name';

if ($user->isDirty()) {
    // There are unsaved changes
}
```

**Check if a specific field changed:**
```php
if ($user->isDirty('name')) {
    // The 'name' field was modified
}
```

**See which fields changed:**
```php
$dirty = $user->getDirty(); // e.g., ['name', 'email']
```

### Typical scenarios
- Prevent saving when nothing has changed.
- Warn users before they leave a form with unsaved edits.
- Highlight changed fields in a review step.

### Example: Send Email Verification When Email Changes

Suppose you want to automatically send an email verification request whenever a user updates their profile and changes their email address. With `isDirty('email')`, you can easily detect this:

```php
$user = new User(23);
$user->name = 'John Doe';

if ($user->isDirty('email')) { // false
    $user->email_verified_at = null;
}

// Update user changes
$user->update();

if($user->email_verified_at == null) {
    // send verification mail
}
```

In above example, only user's name was changed, so before saving the profile, `$user->isDirty('email')` check will be false.This way, you only send the verification request if the email was actually updated—no need to compare values manually!

> Once the `insert()` or `update()` method is called on the model instance, the ORM clears all the dirty attributes. So `isDirty()` method returns **false** and `getDirty()` method returns **empty** array after model persistence.

## Query Builders

**Lightpack** models are capable [query builders](/db-query-builder) too. 

To get a query builder on a model, call the static method `query()`:

```php
$productQuery = Product::query();
```

Now you can access all the methods on [query builders](/query-builder). Below are some example for using query builder on a model.

**Fetch all products**
```php
$products = Product::query()->all();
```

**Fetch all active products**
```php
$products = Product::query()->where('active', '=', '1')->all();
```

**Fetch products with matching ids**
```php
$products = Product::query()->whereIn('id', [1,2,3])->all();
```

**Fetch all products with at least one order**

```php
$products = Product::query()->has('orders')->all();
```

The above is same as:

```php
$products = Product::query()->has('orders', '>', 0)->all();
// or
$products = Product::query()->has('orders', '>=', 1)->all();
```

**Fetch products with no orders**

```php
$products = Product::query()->has('orders', '=', 0)->all();
```

**Fetch products with at least 2 orders**

```php
$products = Product::query()->has('orders', '>', 1)->all();
// or
$products = Product::query()->has('orders', '>=', 2)->all();
```

**Fetch products with atmost 2 orders**

```php
$products = Product::query()->has('orders', '<', 3)->all();
// or
$products = Product::query()->has('orders', '<=', 2)->all();
```

**Callbacks as query constraints**

You can even pass a callback as **4th** parameter to `has()` method to add more constraints on relationship. For example, suppose you want to fetch **products** with atleast **2 paid orders**.

```php
$products = Product::query()->has('orders', '>=', 2, function($q) {
    $q->where('paid', '=', true);
});
```

## Query Filters

Query filters provide a clean way to filter database records using model scopes. They allow you to encapsulate common query constraints and apply them dynamically.

### Defining Filter Scopes

Create filter scopes by adding methods prefixed with `scope` to your model:

```php
use Lightpack\Database\Lucid\Model;

class User extends Model
{
    protected $table = 'users';

    protected function scopeStatus($query, $value)
    {
        $query->where('status', $value);
    }

    protected function scopeType($query, $value)
    {
        $query->where('type', $value);
    }

    protected function scopeRole($query, $value)
    {
        $query->where('role', $value);
    }

    protected function scopeSearch($query, $value)
    {
        $query->where('name', 'LIKE', "%{$value}%");
    }
}
```

### Using Filters

Apply filters using the static `filters()` method:

```php
// Fetch all users
$users = User::filters(['status' => 'active'])->all();

// Fetch active users
$users = User::filters(['status' => 'active'])->all();

// Combine multiple filters
$users = User::filters([
    'status' => 'active',
    'role' => 'admin',
    'search' => 'john'
])->all();
```

### Type hint scope parameters

You can type hint `$query` and `$value` parameters for better code clarity:

```php
class User extends Model
{
    protected function scopeTags(Query $query, array|string $value)
    {
        if (is_string($value)) {
            $value = explode(',', $value);
        }
        $query->whereIn('tag', $value);
    }
}
```

```php
// Usage
$users = User::filters([
    'tags' => 'php,mysql,redis'
])->all();
```

```php
// Or with array
$users = User::filters([
    'tags' => ['php', 'mysql', 'redis']
])->all();
```

## Global Scope

Global scopes let you automatically apply common query conditions to all queries on a model—ensuring consistent, safe, and DRY data access. This is especially powerful for multi-tenant applications, or any scenario where you want to transparently filter data for all operations.

### What is a Global Scope?
A global scope is a rule that is always applied to every query for a model, whether you’re fetching, updating, deleting, or counting records. This helps prevent accidental data leaks and reduces repetitive code.

### How to Define a Global Scope
To add a global scope, override the inherited method `globalScope()` in your model. Any conditions you add to the `$query` will be automatically included in all queries for that model.

**Example: Restricting by Tenant**

```php
class TenantModel extends Model
{
    public function globalScope(Query $query)
    {
        // Only show records for tenant_id = 1
        $query->where('tenant_id', 1);
    }
}
```
Now, any model inheriting from `TenantModel` will always include `WHERE tenant_id = 1` in its queries—no matter what operation you perform.

### Why is this Powerful?
- **Security:** Prevents users from accessing data outside their tenant.
- **Consistency:** No risk of forgetting to add the filter in a query.
- **Simplicity:** Write your code as if you’re working with a single-tenant table.

### Real-World Example
Suppose you have a `users` table with a `tenant_id` column. By using a global scope, you can ensure that all queries only affect users belonging to the current tenant:

```php
class User extends TenantModel
{
    protected $table = 'users';
}
```
Now, all of these will only affect tenant 1:
```php
User::query()->all();
User::query()->count();
User::query()->where('active', 1)->all();
User::query()->delete();
User::query()->update(['active' => 0]);
```

### Best Practices
- Define global scopes for any rule that should always apply (tenancy, soft deletes, published status).
- Be careful: global scopes apply to **all** queries, including destructive ones like `delete()` and `update()`.

---

## Model Hooks

Lightpack Lucid models provide a set of protected lifecycle hook methods that allow you to inject custom logic before and after key persistence operations—**without global events, observers, or magic**. These hooks allow to extend model behavior making your code organized and discoverable

### Available Hook Methods

| Hook                | When is it called?                                 |
|---------------------|---------------------------------------------------|
| `beforeInsert()`    | Before `insert()`                                  |
| `afterInsert()`     | After `insert()`                                   |
| `beforeUpdate()`    | Before `update()`                                  |
| `afterUpdate()`     | After `update()`                                   |
| `beforeDelete()`    | Before `delete()`                                  |
| `afterDelete()`     | After `delete()`                                   |

---

### How and Why to Use Hooks

- **Validation:** Enforce business rules before DB changes
- **Mutation:** Mutate/transform data (e.g., hash, normalize)
- **Side Effects:** Trigger actions after DB changes (e.g., notifications, cache)
- **Prevention:** Abort operation by throwing exceptions
- **Audit/Logging:** Record changes or actions

---

### Practical Examples for Each Hook

### beforeInsert()
Called before inserting a new record:
```php
protected function beforeInsert()
{
    // Example: Hash password before storing
    if (!empty($this->password)) {
        $this->password = password_hash($this->password, PASSWORD_DEFAULT);
    }
    // Example: Set created_by
    $this->created_by = Auth::userId();
}
```

### afterInsert()
Called after inserting a new record:
```php
protected function afterInsert()
{
    // Example: Send welcome email
    Mailer::sendWelcome($this->email);
    // Example: Log creation
    Audit::log('Created user: ' . $this->id);
}
```

### beforeUpdate()
Called before updating an existing record:
```php
protected function beforeUpdate()
{
    // Example: Prevent email change
    if ($this->isDirty('email')) {
        throw new \RuntimeException('Email cannot be changed.');
    }
    // Example: Update audit fields
    $this->updated_by = Auth::userId();
}
```

### afterUpdate()
Called after updating an existing record:
```php
protected function afterUpdate()
{
    // Example: Invalidate related cache
    Cache::forget('user_' . $this->id);
    // Example: Notify admin
    Notification::admin('User updated: ' . $this->id);
}
```

### beforeDelete()
Called before deleting a record:
```php
protected function beforeDelete()
{
    // Example: Prevent deletion if related orders exist
    if ($this->orders()->count() > 0) {
        throw new \RuntimeException('Cannot delete user with orders.');
    }
    // Example: Archive data
    ArchiveService::archive($this->toArray());
}
```

### afterDelete()
Called after deleting a record:
```php
protected function afterDelete()
{
    // Example: Remove from search index
    SearchIndex::remove('users', $this->id);
    // Example: Log deletion
    Audit::log('Deleted user: ' . $this->id);
}
```

---

### More Realistic Scenarios
- **beforeInsert:** Generate a UUID PK for non-auto-increment models
- **beforeUpdate:** Prevent updates to immutable fields (e.g., SSN)
- **afterInsert:** Trigger onboarding workflow
- **afterUpdate:** Sync changes to external APIs
- **beforeDelete:** Clean up dependent child records (manual cascade)
- **afterDelete:** Notify other systems of deletion

---

### Best Practices & Gotchas
- **Keep hooks focused:** Only put logic relevant to that model and operation
- **Throw exceptions to abort:** Any exception will prevent the operation
- **Avoid heavy side effects in hooks:** For long-running tasks, consider queueing
- **No global events:** All logic must be per-model, explicit, and discoverable
- **Avoid external dependencies if possible:** Keep hooks self-contained

---

## Cast Into Array

To convert loaded models into **array**, use `toArray()` method.

```php
$product = new Product(23);
$productArray = $product->toArray();
```

```php
$products = Product::query()->limit(10)->all();
$productsArray = $products->toArray();
```

## Hidden Attributes

Sometimes, you don’t want certain model attributes to show up when converting your models to arrays or serializing them (for example, when returning JSON responses from an API). The `$hidden` property on your model lets you easily hide sensitive or irrelevant fields from output.

### Why Hide Attributes?
- **Security:** Prevent leaking sensitive data (like passwords, tokens, internal IDs).
- **Clean Output:** Remove fields that aren’t needed by the client or API consumer.
- **Consistency:** Ensure only intended data is exposed in API responses or exports.

### How to Use

Just define the `$hidden` property as an array of attribute names in your model:

```php
class User extends Model
{
    protected $hidden = [
        'password',
        'remember_token',
        'internal_notes',
    ];
}
```

Now, when you call `toArray()` or serialize the model (e.g., for JSON), these fields will be automatically excluded:

```php
$user = new User(23);
$userArray = $user->toArray();
// 'password', 'remember_token', and 'internal_notes' will NOT appear in $userArray
```

This also applies to collections:

```php
$users = User::query()->all();
$usersArray = $users->toArray(); // All hidden fields are excluded for every user
```

### Best Practices
- Always hide sensitive fields like passwords and tokens.
- Only include what’s necessary for your consumers—less is more.
- Hidden attributes only affect serialization/array conversion—they are still accessible in your code.