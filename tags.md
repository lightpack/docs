# Tags System

Lightpack’s Tags system provides framework-native way to add tagging support to any Lucid model. It is designed for many-to-many polymorphic tagging (e.g., posts, products, users can all be tagged).

---

## Migration

The Tags system uses two tables:
- `tags`: Stores tag definitions (`id`, `name`, `slug`, timestamps).
- `tag_models`: Pivot table connecting tags to models (`tag_id`, `model_id`, `model_type`).

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

### tags

Returns a pivot query for the model’s tags.

```php
// Get all Tag objects for this post
$post->tags()->all(); 
```

### attachTags

Attach one or more tags to the model.

```php
// Attach tags by ID
$post->attachTags([1, 2, 3]); 
```

### detachTags

Detach one or more tags from the model.

```php
// Remove tag with ID 2
$post->detachTags([2]); 
```

### syncTags

Replace all tags on the model with the given IDs.

```php
$post->syncTags([3]); // Only tag 3 remains
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
- **Type isolation:** Only tags for the correct model type are returned (see `model_type` in pivot).
- **Syncing:** Removes all tags not in the new list, attaches any new ones.
- **Filtering:** `scopeTags` matches any of the provided tag IDs (logical OR).

---