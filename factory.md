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
- `times(int $count)` — Specify how many entities to generate (fluent interface).
- `make(array $overrides = [])` — Generate a single entity array, or a batch if `times()` was called.
- Field overrides: pass an array to `make()` to override default values.

**Examples:**
```php
// Generate a single user entity
$user = (new UserFactory)->make();

// Generate a batch of 3 user entities
$users = (new UserFactory)->times(3)->make();

// Generate a single user entity overriding email
$user = (new UserFactory)->make([
    'email' => 'custom@example.com'
]);
```

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
// Save a single model
$model = (new ProductFactory)->save();

// Save a batch of models with name override
$models = (new ProductFactory)
    ->times(2)
    ->save(['name' => 'Batch Name']);
```

---
