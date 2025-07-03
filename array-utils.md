# Array Utility Documentation

Lightpack's Array utility provides a powerful set of methods for working with arrays, featuring dot notation access, wildcard matching, array transformations, and hierarchical tree building.

## Basic Usage

```php
use Lightpack\Utils\Arr;

$arr = new Arr();
```

Or simply call `arr()` utility function.

It exposes following methods to efficiently workk with arrays:

- get()
- set()
- has()
- delete()
- tree()
- transpose()

### Basic Operations

```php
$data = [
    'user' => [
        'profile' => [
            'name' => 'John Doe',
            'email' => 'john@example.com'
        ]
    ]
];

// Check if key exists
$exists = arr()->has('user.profile.name', $data);  // true

// Get value
$name = arr()->get('user.profile.name', $data);    // 'John Doe'
$age = arr()->get('user.age', $data, 25);          // 25 (default value)

// Set value
arr()->set('user.profile.age', 30, $data);

// Delete value
arr()->delete('user.profile.email', $data);
```

## Dot Notation

### Basic Dot Notation

```php
$data = [
    'users' => [
        'active' => [
            'john' => ['age' => 25],
            'jane' => ['age' => 30]
        ]
    ]
];

// Access nested data
$johnsAge = arr()->get('users.active.john.age', $data);    // 25

// Set nested data
arr()->set('users.active.bob.age', 35, $data);

// Check nested path
if (arr()->has('users.active.jane', $data)) {
    // Path exists
}

// Delete nested data
arr()->delete('users.active.john', $data);
```

### Wildcard Matching

```php
$data = [
    'users' => [
        'active' => [
            'john' => ['age' => 25],
            'jane' => ['age' => 30]
        ],
        'inactive' => [
            'bob' => ['age' => 35]
        ]
    ]
];

// Get all ages using wildcard
$ages = arr()->get('users.*.*.age', $data);
// [25, 30, 35]

// Get all active user ages
$activeAges = arr()->get('users.active.*.age', $data);
// [25, 30]
```

## Array Transformations

### Building Hierarchical Trees

```php
// Flat array of categories
$categories = [
    ['id' => 1, 'parent_id' => 0, 'name' => 'Electronics'],
    ['id' => 2, 'parent_id' => 1, 'name' => 'Phones'],
    ['id' => 3, 'parent_id' => 1, 'name' => 'Laptops'],
    ['id' => 4, 'parent_id' => 2, 'name' => 'iPhone'],
    ['id' => 5, 'parent_id' => 2, 'name' => 'Android'],
];

// Convert to hierarchical tree
$tree = arr()->tree($categories);

/* Result:
[
    [
        'id' => 1,
        'parent_id' => 0,
        'name' => 'Electronics',
        'children' => [
            [
                'id' => 2,
                'parent_id' => 1,
                'name' => 'Phones',
                'children' => [
                    ['id' => 4, 'parent_id' => 2, 'name' => 'iPhone'],
                    ['id' => 5, 'parent_id' => 2, 'name' => 'Android']
                ]
            ],
            ['id' => 3, 'parent_id' => 1, 'name' => 'Laptops']
        ]
    ]
]
*/
```

```php
// Custom key names
$tree = arr()->tree($items, 0, 'category_id', 'parent_category_id');
```

### Array Transposition

```php
// Column-based data
$data = [
    'name' => ['John', 'Jane', 'Bob'],
    'age' => [25, 30, 35],
    'city' => ['New York', 'London', 'Paris']
];

// Convert to row-based data
$rows = arr()->transpose($data);

/* Result:
[
    ['name' => 'John', 'age' => 25, 'city' => 'New York'],
    ['name' => 'Jane', 'age' => 30, 'city' => 'London'],
    ['name' => 'Bob', 'age' => 35, 'city' => 'Paris']
]
*/

// Transpose specific keys only
$rows = arr()->transpose($data, ['name', 'age']);
```

---