# Taxonomies System

Lightpack’s Taxonomies system provides a robust, hierarchical, and framework-native way to add category, menu, or any tree-like structure to your Lucid models. It supports many-to-many polymorphic relationships, deep trees, and flexible querying.

---

## Migration

The Taxonomies system requires two tables:
- `taxonomies`: Stores taxonomy nodes (id, name, slug, type, parent_id, sort_order, meta, timestamps).
- `taxonomy_models`: Pivot table connecting taxonomies to any model (taxonomy_id, model_id, model_type).

Create schema migration file:

```cli
php console create:migration --support=taxonomies
```

Run migration:

```cli
php console migrate:up
```

---

## Adding Taxonomies to Your Model

Add `TaxonomyTrait` to any model to make it taxonomy-aware:

```php
use Lightpack\Taxonomies\TaxonomyTrait;

class Post extends Model {
    use TaxonomyTrait;
}
```

---

## TaxonomyTrait API

### taxonomies()
Returns a pivot query for the model’s taxonomies.
```php
$post->taxonomies()->all(); // Get all taxonomy nodes for this post
```

### attachTaxonomies()
Attach one or more taxonomy nodes by ID:
```php
$post->attachTaxonomies([1, 2, 3]);
```

### detachTaxonomies()
Detach one or more taxonomy nodes by ID:
```php
$post->detachTaxonomies([2]);
```

### syncTaxonomies()
Replace all taxonomies on the model with the given IDs:
```php
$post->syncTaxonomies([3]);
```

### filters()
Query scope to filter models by taxonomy IDs:
```php
// All posts with taxonomy 1 or 2
$posts = Post::filters(['taxonomies' => [1, 2]])->all();
```

---

## Hierarchy & Tree Operations

Lightpack's `Taxonomy` model provides a rich set of methods for working with hierarchical data. Each method is designed for clarity, performance, and practical developer use. Below are detailed explanations and usage for each helper:

### parent()

Returns a query for the parent taxonomy node, if any.

```php
// fetch parent node
$parent = $taxonomy->parent()->one();

if ($parent) {
    echo $parent->name;
}
```

- **Notes:** Returns `null` if the node is a root.

### children()


Returns a query for all direct child nodes, ordered by `sort_order` then `id`.

```php
$children = $taxonomy->children()->all();

foreach ($children as $child) {
    echo $child->name;
}
```

- **Notes:** Returns an empty collection if there are no children.

### siblings()

Returns all taxonomy nodes at the same level (same parent), excluding the current node, ordered by `sort_order` then `id`.

```php
$siblings = $taxonomy->siblings();

foreach ($siblings as $sibling) {
    echo $sibling->name;
}
```

- **Notes:** Root nodes' siblings are other roots.

### ancestors()

Returns all ancestor nodes from the root down to the immediate parent.

```php
$ancestors = $taxonomy->ancestors();

foreach ($ancestors as $ancestor) {
    echo $ancestor->name;
}
```

- **Notes:** Returns an empty collection for root nodes.

### descendants()

Returns all descendant nodes (children, grandchildren, etc.) in depth-first order.

```php
$descendants = $taxonomy->descendants();
foreach ($descendants as $descendant) {
    echo $descendant->name;
}
```

- **Notes:** Returns an empty collection if there are no descendants.

### tree()

Returns this node and all descendants as a nested array structure.

```php
$tree = $taxonomy->tree();
print_r($tree);
```

- **Notes:** The array includes all node data and a `children` key if there are children.

### forest()

Returns all root nodes as an array of nested trees.

```php
$forest = Taxonomy::forest();
foreach ($forest as $tree) {
    // Each $tree is a nested array
}
```
- **Notes:** Useful for generating complete navigation menus or category trees.

### fullSlug($separator = '/')

**Signature:** `fullSlug(string $separator = '/'): string`

Returns the full hierarchical slug for this node, joining ancestor and self slugs.

```php
$slug = $taxonomy->fullSlug('-'); // e.g., "root-child-grandchild"
```
- **Notes:** Skips empty slugs; customizable separator.

### breadcrumbs()

Returns an array of taxonomy nodes from root to this node (breadcrumb trail).

```php
$trail = $taxonomy->breadcrumbs();

foreach ($trail as $node) {
    echo $node->name;
}
```

- **Notes:** Always includes the current node as the last element.

### moveTo()
**Signature:** `moveTo(?int $newParentId = null): void`

Moves this node (and its entire subtree) under a new parent node.

```php
$taxonomy->moveTo(2); // Move under taxonomy ID 2
$taxonomy->moveTo(null); // Move to root
```

- **Notes:** Throws exception if trying to move under itself or any descendant (prevents cycles).

### reorder()

**Signature:** `static reorder(array $idOrderMap): void`

Bulk update the `sort_order` of multiple nodes. Pass an array of `[taxonomy_id => sort_order, ...]`.

```php
Taxonomy::reorder([
    2 => 1, // taxonomy ID 2 becomes first
    3 => 2, // taxonomy ID 3 becomes second
]);
```

- **Notes:** Only affects the order among siblings or roots.

### bulkMove()

**Signature:** `static bulkMove(array $ids, ?int $newParentId = null): void`

Move multiple taxonomy nodes under a new parent in one call.

```php
Taxonomy::bulkMove([3, 4], 5); // Move nodes 3 and 4 under node 5
```

- **Notes:** Throws if any move would create a cycle.

---

## Edge Cases & Behavior

- **Duplicate attaches:** Attaching a taxonomy already present is safe and has no effect.
- **Detaching non-existent taxonomies:** Detaching a taxonomy not present does nothing (no error).
- **Type isolation:** Only taxonomies for the correct model type are returned (see `model_type` in pivot).
- **Syncing:** Removes all taxonomies not in the new list, attaches any new ones.
- **Filtering:** `scopeTaxonomies` matches any of the provided taxonomy IDs (logical OR).
- **Cycle prevention:** `moveTo()` and `bulkMove()` prevent cycles (cannot move under self or descendant).
- **Sort order:** Children and siblings are always ordered by `sort_order` then `id`.
- **Meta fields:** Store custom data (e.g., icon, color) in the `meta` column as an array.
- **Empty state:** All tree and forest helpers handle empty trees gracefully.

---

## Practical End-to-End Examples

Below are practical, real-world scenarios that demonstrate how to use Lightpack Taxonomies in your application. These examples cover taxonomy creation, building a tree, attaching to models, querying, and navigation.

### 1. Creating a Category Tree

```php
// Create root categories
$news = new Taxonomy();
$news->name = 'News';
$news->slug = 'news';
$news->type = 'category';
$news->save();

$opinion = new Taxonomy();
$opinion->name = 'Opinion';
$opinion->slug = 'opinion';
$opinion->type = 'category';
$opinion->save();

// Create children
$local = new Taxonomy();
$local->name = 'Local';
$local->slug = 'local';
$local->type = 'category';
$local->parent_id = $news->id;
$local->save();

$global = new Taxonomy();
$global->name = 'Global';
$global->slug = 'global';
$global->type = 'category';
$global->parent_id = $news->id;
$global->save();
```

### 2. Attaching Taxonomies to a Model

```php
// Assume Post model uses TaxonomyTrait
$post = Post::find(101);

// Attach categories by ID
$post->attachTaxonomies([$news->id, $local->id]);
```

### 3. Querying Posts by Taxonomy

```php
// Get all posts tagged as 'News' OR 'Local'
$posts = Post::filters(['taxonomies' => [$news->id, $local->id]])->all();
```

### 4. Navigating the Taxonomy Tree

```php
// Get all children of 'News'
$children = $news->children()->all(); // ['Local', 'Global']

// Get all ancestors of 'Local'
$ancestors = $local->ancestors(); // ['News']

// Get full slug for 'Local'
$slug = $local->fullSlug(); // 'news/local'

// Get breadcrumbs for 'Local'
$trail = $local->breadcrumbs(); // ['News', 'Local']
```

### 5. Moving and Reordering Taxonomies

```php
// Move 'Local' under 'Opinion'
$local->moveTo($opinion->id);

// Bulk reorder children of 'News'
Taxonomy::reorder([
    $global->id => 1, // 'Global' first
    $local->id => 2,  // 'Local' second
]);
```

### 6. Building a Complete Tree or Forest

```php
// Get the full tree for 'News'
$newsTree = $news->tree();

// Get all taxonomy trees (forest)
$forest = Taxonomy::forest();
```

### 7. Using Meta Fields

```php
$news->meta = [
    'icon' => 'fa-newspaper',
    'color' => '#3366cc',
    'description' => 'All news articles',
];
$news->save();

// Access meta
$icon = $news->meta['icon'];
```

---

These examples show how to:
- Build and manage a taxonomy tree
- Attach taxonomies to your models
- Query and filter by taxonomy
- Navigate and display hierarchical data
- Move and reorder nodes safely
- Store custom data in meta fields

---
