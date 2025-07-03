# Image Utilities

Lightpack’s `Image` utility provides a robust, chainable API for image manipulation, supporting all common operations for avatars, thumbnails, and web graphics. It is built on PHP’s GD extension and covers resizing, cropping, filters, format conversion, and more, with strong error handling and test coverage.

---

## Overview
- **Load from file or URL** (JPEG, PNG, WebP)
- **Resize, crop, rotate, flip** with aspect ratio and quality controls
- **Save** to JPEG, PNG, or WebP with custom quality/compression
- **Generate avatars and thumbnails** in standard sizes
- **Apply filters and effects:** grayscale, brightness, contrast, blur, sharpen, colorize, sepia, invert, pixelate, emboss, edge detect, posterize
- **Watermark and text overlay** support
- **Method chaining** for expressive, fluent code

---

## Basic Usage

```php
use Lightpack\Utils\Image;

// Load image
$image = new Image('/path/to/photo.jpg');

// Resize and save
$image->resize(400, 300)->save('/path/to/output.jpg');

// Crop, grayscale, and save as PNG
$image = new Image('/path/to/photo.jpg');
$image->crop(100, 100, 10, 10)
      ->grayscale()
      ->save('/path/to/cropped.png');
```

---

## Loading Images
- Supports JPEG, PNG, WebP (file or URL)
- Throws if file/URL is missing or unsupported

```php
$image = new Image('photo.jpg');
$image = new Image('https://example.com/pic.png');
```

---

## Resizing & Cropping

```php
// Resize (maintains aspect ratio if one dimension is zero)
$image->resize(300, 0); // 300px wide, proportional height

// Crop (x, y, width, height)
$image->crop(100, 100, 10, 10);
```

---

## Saving & Format Conversion

The `save()` method writes the current image to disk, automatically selecting the format (JPEG, PNG, or WebP) based on the file extension you provide. It also lets you control quality and compression for each format.

```php
// Save as JPEG (with quality)
$image->save('output.jpg', 90);    // 90% quality (default is 80)

// Save as PNG (with compression)
$image->save('output.png');        // Compression defaults to 7 (0 = none, 9 = max)
$image->setDefaultPngCompression(4); // Set default for future PNG saves

// Save as WebP (with quality)
$image->save('output.webp', 80);   // 80% quality (default is 80)
```

**Format detection:**
- The format is determined by the file extension (`.jpg`, `.jpeg`, `.png`, `.webp`).
- Throws an exception for unsupported extensions.

**Quality & Compression:**
- JPEG/WebP: Quality is 0–100 (higher = better quality, larger file).
- PNG: Compression is 0–9 (higher = smaller file, slower save, no quality loss).
- You can set global defaults for each format.

**Directory checks:**
- Throws if the target directory does not exist or is not writable.
- Does not create directories automatically.

**Overwriting:**
- If the target file exists, it will be overwritten without warning.

**Image object state after save:**
- After saving, the internal image resource is destroyed to free memory.
- The `Image` object cannot be reused for further operations or additional saves.
- **To perform multiple saves or edits, reload the image or clone before saving:**

```php
$image = new Image('photo.jpg');
$image->resize(200, 200)->save('small.jpg');

// Need to reload or re-instantiate for further use:
$image = new Image('photo.jpg');
$image->resize(400, 400)->save('large.jpg');
```

> **Tip:** If you want to save multiple versions (e.g., different sizes or formats), clone the `Image` object before calling `save()`:
>
> ```php
> $img = new Image('photo.jpg');
> $clone = clone $img;
> $img->resize(200, 200)->save('small.jpg');
> $clone->resize(400, 400)->save('large.jpg');
> ```

**Best Practices:**
- Always check that your target directory exists and is writable.
- Use explicit file extensions to control output format.
- Reload or clone the image for multiple operations after saving.

> **Warning:** If you try to use the same `Image` object after `save()`, you will get an error because the underlying image resource has been destroyed.

---

## Quality & Compression Settings

```php
$image->setDefaultJpegQuality(85);
$image->setDefaultWebpQuality(90);
$image->setDefaultPngCompression(6); // 0 (none) to 9 (max)
```

---

## Avatars & Thumbnails

Lightpack’s `Image` utility provides dedicated methods for generating standardized avatar and thumbnail images—ideal for user profiles, listings, galleries, blog posts, and more. These helpers automate resizing and format selection, ensuring consistency and optimal display across your application.

### What Are Avatars and Thumbnails?
- **Avatars:** Square images, typically used for user profile pictures, comments, or account listings. Sizes are chosen for clarity at different UI scales.
- **Thumbnails:** Rectangular or square images, optimized for previews, cards, or gallery grids. Useful for content teasers and fast-loading image lists.

### Standard Sizes
- **Avatars:**
  - `small`: 48x48 px (comments, lists)
  - `medium`: 96x96 px (profile preview)
  - `large`: 192x192 px (profile page)
- **Thumbnails:**
  - `small`: 300px wide (height auto)
  - `medium`: 600px wide (height auto)
  - `large`: 1200px wide (height auto)

> **Note:** Thumbnail height is automatically calculated to preserve the original aspect ratio.

### Generating Avatars
```php
$image = new Image('userpic.jpg');
$paths = $image->avatar('avatars/user123');
// $paths['small']  => 'avatars/user123_avatar_small.webp'
// $paths['medium'] => 'avatars/user123_avatar_medium.webp'
// $paths['large']  => 'avatars/user123_avatar_large.webp'
```
- Avatars are always saved as WebP for modern browser support and efficient storage.
- You can specify which sizes to generate:

```php
$paths = $image->avatar('avatars/user123', ['small', 'large']);
```

### Generating Thumbnails
```php
$image = new Image('photo.jpg');
$thumbs = $image->thumbnail('thumbs/photo123');
// $thumbs['small']  => 'thumbs/photo123_thumb_small.jpg'
// $thumbs['medium'] => 'thumbs/photo123_thumb_medium.jpg'
// $thumbs['large']  => 'thumbs/photo123_thumb_large.jpg'
```
- Thumbnails are always saved as JPEG for maximum compatibility and fast loading.
- You can specify which sizes to generate:

```php
$thumbs = $image->thumbnail('thumbs/photo123', ['medium']);
```

### File Naming Conventions
- Avatars: `{base}_avatar_{size}.webp`
- Thumbnails: `{base}_thumb_{size}.jpg`

### Custom Sizes & Error Handling
- Both methods accept a custom array of sizes (must match supported keys).
- Throws an `InvalidArgumentException` if an invalid size is given.
- The original image is not modified; each output is generated from a clone.

### Method Chaining & Reuse
- Both `avatar()` and `thumbnail()` clone the image internally, so you can reuse the original `Image` object for other operations.
- The returned array contains the full paths to each generated image.

### Best Practices
- Store avatars and thumbnails in dedicated directories for easy management.
- Always check that your output directories exist and are writable.
- Use the returned paths for referencing images in your application.

> **Tip:** You can combine avatars, thumbnails, and other manipulations by chaining methods or by working with clones for batch processing.


---

## Filters & Effects

```php
$image->grayscale()->contrast(-20)->brightness(30)->blur(2)->sharpen();
$image->colorize(255, 0, 0)->sepia()->invert()->pixelate(8);
$image->emboss()->edgedetect()->posterize();
```
- All filters are chainable

---

## Watermark & Text Overlay

```php
// Overlay PNG watermark at (x, y), opacity 0-100
$image->watermark('logo.png', 10, 10, 60);

// Overlay text (x, y, size, color, font)
$image->text('Sample', 20, 40, 16, '#ff0000', '/path/to/font.ttf');
```

---

## Rotating & Flipping

```php
$image->rotate(90); // Degrees, counter-clockwise
$image->flip('vertical'); // or 'horizontal'
```

---

## Error Handling & Edge Cases
- Throws exceptions for missing files, unsupported formats, invalid sizes, or failed operations
- All operations require GD extension
- Output image is destroyed after save (reload to reuse)
- Method chaining is supported for all mutators

---