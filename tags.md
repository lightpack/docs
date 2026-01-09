# Tags System

Lightpack’s Tags system provides framework-native way to add tagging support to any Lucid model. It is designed for many-to-many polymorphic tagging (e.g., posts, products, users can all be tagged).

---

## Migration

The Tags system uses two tables:
- `tags`: Stores tag definitions (`id`, `name`, `slug`, timestamps).
- `tag_morphs`: Pivot table connecting tags to models (`tag_id`, `morph_id`, `morph_type`).

Create schema migration file:

```cli
php console create:migration --support=tags
```

Run migration:

```cli
php console migrate:up
```

---

## Adding Tagging to Your Model

Add `TagsTrait` to any model to make it taggable:

```php
use Lightpack\Tags\TagsTrait;

class Post extends Model {
    use TagsTrait;
}
```

---

## TagsTrait API

### tags()

Returns the model's tags. Can be accessed as a property or method.

```php
$tags = $post->tags;
```

### Pivot Operations

Use the pivot methods to manage tag associations:

```php
// Attach tags by ID
$post->tags()->attach([1, 2, 3]);

// Detach tags by ID
$post->tags()->detach([2]);

// Sync tags (replace all with given IDs)
$post->tags()->sync([1, 3]);
```

### filters()

Query scope to filter models by tag IDs.

```php
// All posts with tag 1 or 2
$posts = Post::filters(['tags' => [1, 2]])->all(); 
```

---

## Edge Cases & Behavior
- **Duplicate attaches:** Attaching a tag already present is safe and has no effect.
- **Detaching non-existent tags:** Detaching a tag not present does nothing (no error).
- **Type isolation:** Only tags for the correct model type are returned (see `morph_type` in pivot).
- **Syncing:** Removes all tags not in the new list, attaches any new ones.
- **Filtering:** `scopeTags` matches any of the provided tag IDs (logical OR).

---