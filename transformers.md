# Model Transformers in Lightpack ORM

Model transformers are a powerful feature in Lightpack ORM, designed to help you convert your models (and collections of models) into clean, structured arrays for API responses, view rendering, or any custom output. They empower you to:
- Select and filter fields dynamically
- Include and nest related models
- Transform collections and paginated data
- Support multiple output contexts (e.g., API, view)
- Keep presentation logic out of your models

---

## What is a Transformer?
A transformer is a dedicated class that defines how a model (or collection of models) should be represented as an array. Transformers encapsulate field selection, relation inclusion, and output formatting logic.

**Why use transformers?**
- Centralize and DRY up your serialization logic
- Enforce consistent API and view output
- Support dynamic field selection and relation includes
- Easily adapt output for different consumers (API, admin, etc.)

---

## How Transformer Resolution Works

Transformers in Lightpack ORM are designed to make your API and view output flexible, maintainable, and secure. Understanding how the framework selects and applies transformers is key to leveraging their full power.

### Resolution Process

When you call the `transform()` method on a model (or collection), Lightpack follows a clear, predictable process:

1. **Check the `$transformer` Property**
   - This property tells Lightpack which transformer class to use.
   - It can be:
     - A **string**: The class name of a single transformer (e.g., `UserTransformer::class`).
     - An **array**: A map of context names (like `'api'`, `'view'`) to transformer classes.
     - `null`: No transformer defined—calling `transform()` will throw an error.

2. **Context Support**
   - If you want different outputs for different consumers (API, admin, etc.), use an array mapping contexts to transformer classes.
   - At runtime, specify the context via the `context` option:
   ```php
   $product->transform(['context' => 'api']);
   $product->transform(['context' => 'view']);
   ```

3. **Fields and Includes**
   - You can pass a `fields` array to include only specific fields for the main model or for related models.
   - You can pass an `includes` array to include related data (including nested relations).
   - These options are forwarded directly to the transformer instance before transformation.
   ```php
   $user->transform([
       'fields' => ['self' => ['name', 'email']],
       'includes' => ['profile', 'roles'],
   ]);
   ```

## Defining a Transformer

---

The `transform()` method on any model is the canonical way to convert it to an array for API or view output. This method:

- Looks up the transformer class from the model’s `$transformer` property.
- If `$transformer` is a string, it instantiates that transformer.
- If `$transformer` is an array, it expects a `context` key in the options array (e.g., `'api'`, `'view'`) and selects the transformer for that context.
- If no transformer is defined, or the context is invalid, it throws a clear, descriptive exception.

### Examples
```php
// Single transformer
class User extends Model {
    protected $transformer = UserTransformer::class;
}

// Multiple contexts
class Product extends Model {
    protected $transformer = [
        'api' => ProductApiTransformer::class,
        'view' => ProductViewTransformer::class,
    ];
}

// Usage
$user = new User(1);
$userArray = $user->transform(); // Uses UserTransformer

$product = new Product(1);
$productApi = $product->transform(['context' => 'api']);
$productView = $product->transform(['context' => 'view']);
```

#### Passing Fields and Includes
You can pass `fields` and `includes` options to control the output:
```php
$user->transform([
    'fields' => ['self' => ['name', 'email']],
    'includes' => ['profile', 'roles'],
]);
```
These options are forwarded to the transformer instance and determine which fields and relations are included in the output.

---

A transformer is a PHP class that extends `Lightpack\Database\Lucid\Transformer` and implements a `data($model)` method. This method returns an array representation of the model, controlling which fields are exposed and how relations are included.

### Basic Transformer Example
```php
use Lightpack\Database\Lucid\Transformer;

class ProjectTransformer extends Transformer
{
    protected function data($model): array
    {
        return [
            'id' => $model->id,
            'name' => $model->name,
        ];
    }
}
```

### Using Fields and Includes in Transformers
You don't have to manually filter fields or include relations in your `data()` method—Lightpack handles this based on the `fields()` and `includes()` options passed at runtime. Your `data()` method should return the full set of possible fields for the model.

### Contextual Transformers
You can define multiple transformers for the same model to support different output contexts (e.g., API, view, admin):

```php
class ProductApiTransformer extends Transformer
{
    protected function data($model): array
    {
        return [
            'name' => $model->name,
            'price' => $model->price,
        ];
    }
}

class ProductViewTransformer extends Transformer
{
    protected function data($model): array
    {
        return [
            'id' => $model->id,
            'name' => $model->name,
            'price' => $model->price,
            'color' => $model->color,
        ];
    }
}
```

### Associating Transformers with Models
To use a default transformer, set the `$transformer` property on your model:

```php
class Project extends Model
{
    protected $table = 'projects';
    protected $transformer = ProjectTransformer::class;
}
```

To support multiple contexts, use an array mapping context names to transformer classes:

```php
class Product extends Model
{
    protected $transformer = [
        'api' => ProductApiTransformer::class,
        'view' => ProductViewTransformer::class,
    ];
}
```

When transforming, specify the context:
```php
$product->transform(['context' => 'api']); // Uses ProductApiTransformer
$product->transform(['context' => 'view']); // Uses ProductViewTransformer
```

If an invalid context is provided, Lightpack will throw a descriptive exception listing available contexts.

---

## Basic Usage

### Transforming a Single Model
```php
$project = new Project(1);
$transformer = new ProjectTransformer();

$result = $transformer->transform($project);
// [ 'id' => 1, 'name' => 'Project 1' ]
```

### Field Selection
Select only specific fields for the main model ("self") or for relations:
```php
$result = $transformer
    ->fields(['self' => ['name']])
    ->transform($project);
// [ 'name' => 'Project 1' ]
```

### Including Relations
Include related models by name:
```php
$result = $transformer
    ->includes('tasks')
    ->fields([
        'self' => ['name'],
        'tasks' => ['id', 'name'],
    ])
    ->transform($project);
// [ 'name' => 'Project 1', 'tasks' => [ [ 'id' => 1, 'name' => 'Task 1' ] ] ]
```

---

## Nested Relations & Deep Includes
You can include nested relations using dot notation:
```php
$result = $transformer
    ->includes('tasks.comments')
    ->fields([
        'self' => ['name'],
        'tasks' => ['name'],
        'tasks.comments' => ['content']
    ])
    ->transform($project);
// [ 'name' => 'Project 1', 'tasks' => [ [ 'name' => 'Task 1', 'comments' => [ [ 'content' => 'Comment 1' ] ] ] ] ]
```

You can also include multiple paths at once:
```php
$result = $transformer
    ->includes(['tasks', 'tasks.comments'])
    ->fields([
        'self' => ['id', 'name'],
        'tasks' => ['name'],
        'tasks.comments' => ['content']
    ])
    ->transform($project);
```

---

## Handling Missing or Null Relations
- If a relation is missing or not included, it is omitted from the output.
- If a relation is included but null or empty, it appears as an empty array or null, as appropriate.

---

## Transforming Collections
Transformers can handle collections of models seamlessly:
```php
$projects = Project::query()->all();
$result = $transformer
    ->includes(['tasks'])
    ->fields([
        'self' => ['name'],
        'tasks' => ['name']
    ])
    ->transform($projects);
// [ [ 'name' => 'Project 1', 'tasks' => [ ... ] ], ... ]
```

---

## Model-integrated Transformation
You can call `transform()` directly on a model or collection, passing options:
```php
$result = $project->transform([
    'includes' => ['tasks.comments'],
    'fields' => [
        'self' => ['name'],
        'tasks' => ['name'],
        'tasks.comments' => ['content'],
    ],
]);
```

---

## Pagination Support
If you have a paginated collection, you can transform it for API output:
```php
$pagination = new Pagination($projects, $total, $perPage, $currentPage);
$result = $pagination->transform([
    'includes' => ['tasks'],
    'fields' => [
        'self' => ['name'],
        'tasks' => ['name'],
    ],
]);
// Output includes 'data', 'meta', and 'links' keys
```

---

## Dynamic Includes: Comma and Dot Notation
- **Comma notation**: Direct relations (`includes(['tasks', 'comments'])`)—relations are included at the top level.
- **Dot notation**: Nested relations (`includes('tasks.comments')`)—relations are nested under their parent.

You can mix and match as needed.

---

## Contextual Transformers
Transformers can support multiple output contexts (e.g., API, view) for the same model:

```php
$product = new Product(1);

// API context
$result = $product->transform(['context' => 'api']);
// Uses ProductApiTransformer

// View context
$result = $product->transform(['context' => 'view']);
// Uses ProductViewTransformer
```

If an invalid context is provided, a clear exception is thrown listing available contexts.

---

## Error Handling & Robustness
- If you include a non-existent relation, it is simply ignored in the output.
- If you call `transform()` on a model without a transformer, you get a clear exception: `No transformer defined for model: ModelClass`.
- If you specify an invalid context, a descriptive exception is thrown: `Invalid transformer context 'admin' for ModelClass. Available contexts: api, view`.
- Null or missing relations are output as empty arrays or omitted, never causing errors.

---