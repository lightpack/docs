# Factory

A **factory** in Lightpack is a dedicated class that encapsulates the logic for generating consistent, customizable arrays or model instances—typically for testing, seeding, or rapid prototyping. Factories let you define default data structures, produce batches of fake or default data, and override fields as needed.

---

## Why Use Factories?

Factories are one of the most powerful tools for building reliable, maintainable applications—especially when it comes to testing and prototyping. Here’s why they matter in Lightpack:

- **Eliminate Repetition:** Without factories, every test or seed script would need to manually construct arrays or models—over and over, with subtle inconsistencies and lots of boilerplate.
- **Consistency Everywhere:** Factories guarantee that your test data always follows the same structure and rules. Change it in one place, and every test, seeder, or script benefits instantly.
- **Rapid Prototyping:** Need 50 users, 100 products, or a complex data tree? Factories let you spin up realistic data in a single line, so you can focus on logic, not busywork.
- **Safer Refactoring:** When your data structure changes, update the factory—not hundreds of scattered test cases.
- **Batch Operations Made Easy:** Factories make it trivial to generate large datasets for performance testing, UI demos, or stress tests.
- **Override Only What Matters:** Want to test an edge case? Override just one field—no need to duplicate the entire data definition.
- **Readable, Intentional Tests:** Your tests and seeders become clear and expressive, showing intent (“create a user with email X”) instead of low-level details.

---

## Factory Base Class

You define a new factory class by extending `Factory` and implementing the `template()` method:

```php
class UserFactory extends Factory
{
    protected function template(): array
    {
        $faker = new Faker();
        
        return [
            'name' => $faker->name(),
            'email' => $faker->unique()->email(),
            'address' => $faker->address(),
            'created_at' => $faker->date('Y-m-d H:i:s'),
        ];
    }
}
```

### Core Methods
- `times(int $count): static` — Specify how many entities to generate. Returns `$this` for method chaining.
- `make(array $overrides = []): array` — Generate a single entity array, or an array of arrays if `times()` was called. **Resets the batch count after use.**
- **Field overrides:** Pass an array to `make()` to override default values. In batch mode, the same overrides apply to **all** generated items.

**Examples:**
```php
// Generate a single user entity (returns array)
$user = (new UserFactory)->make();
// Result: ['name' => 'John Doe', 'email' => 'john@example.com', ...]

// Generate a batch of 3 user entities (returns array of arrays)
$users = (new UserFactory)->times(3)->make();
// Result: [['name' => ...], ['name' => ...], ['name' => ...]]

// Generate a single user entity overriding email
$user = (new UserFactory)->make([
    'email' => 'custom@example.com'
]);
// Result: ['name' => 'John Doe', 'email' => 'custom@example.com', ...]

// Batch with overrides - same override applies to ALL items
$users = (new UserFactory)->times(3)->make(['email' => 'same@example.com']);
// All 3 users will have 'same@example.com' as email
```

> **Important:** The batch count is automatically reset to `null` after calling `make()`, so you can reuse the factory instance.

---

## ModelFactory: For Persisted Models

`ModelFactory` extends `Factory` and adds support for creating and saving model instances.

- Implement both `template()` (attributes) and `model()` (model class name).
- Use `save()` to create and persist a model or batch of models.

In this example we create a `ProductFactory` class for `Product` model.

```php
class ProductFactory extends ModelFactory
{
    protected function template(): array
    {
        return [
            'sku' => 'DUMMY_1000',
            'name' => 'Dummy Product',
        ];
    }
    
    protected function model(): string
    {
        return Product::class;
    }
}
```

**Usage**

```php
// Save a single model (returns Product instance)
$product = (new ProductFactory)->save();
// $product is a Product model instance, already inserted

// Save a batch of models (returns array of Product instances)
$products = (new ProductFactory)->times(2)->save();
// $products is an array of 2 Product model instances

// Save with overrides - same override applies to all in batch
$products = (new ProductFactory)
    ->times(2)
    ->save(['name' => 'Batch Name']);
// Both products will have 'name' => 'Batch Name'
```

> **Note:** `save()` returns **Model instances**, not arrays. Use `make()` if you only need the data arrays without persisting.

---
