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

## Defining a Transformer

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
- If you specify an invalid context, a clear error is thrown with available contexts listed.
- Null or missing relations are output as empty arrays or omitted, never causing errors.

---

## Best Practices & Tips
- **Always define a transformer for each model** that will be serialized for API or view output.
- **Use field selection** to minimize payloads and control exposure of sensitive data.
- **Use includes and nested includes** to fetch and structure related data as needed.
- **Leverage contexts** to serve different consumers (API, admin, view) with tailored output.
- **Transform collections and paginated data** directly for consistent, predictable API responses.
- **Combine with eager loading** for best performance—transformers expect related data to be loaded efficiently.

---

## Example: Full-featured Transformation

```php
$project = new Project(2);

$result = (new ProjectTransformer)
    ->includes(['tasks', 'tasks.comments'])
    ->fields([
        'self' => ['id', 'name'],
        'tasks' => ['name'],
        'tasks.comments' => ['content']
    ])
    ->transform($project);

// Output:
// [
//   'id' => 2,
//   'name' => 'Project 2',
//   'tasks' => [
//     [
//       'name' => 'Task 2',
//       'comments' => [
//         ['content' => 'Comment 2'],
//         ['content' => 'Comment 3']
//       ]
//     ],
//     [ 'name' => 'Task 3', 'comments' => [] ]
//   ]
// ]
```

---

Lightpack model transformers give you precise, powerful, and expressive control over your model output, making it easy to build robust APIs and clean views with minimal effort.

---