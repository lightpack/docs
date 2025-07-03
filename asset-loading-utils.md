# Asset Utilities

Lightpack’s `Asset` utility provides a feature rich toolkit for managing, versioning, and rendering static assets (CSS, JS, images, fonts) in your PHP applications. It supports cache-busting, CDN integration, HTML tag generation, and Google Font management—all with a clear, expressive API and robust test coverage.

---

## Overview
- **Asset URL generation** with optional versioning for cache busting
- **Manifest-based versioning** for tracked asset directories
- **CDN/base URL support** via environment variables
- **HTML helpers** for CSS, JS, images, and ES modules
- **Import map generation** for modern JavaScript workflows
- **Google Fonts downloader** for local font hosting
- **Strong test coverage** for all major features

---

## Common Challenges

Lightpack's asset loading facility tries to solve some of the most common challenges for which developers usually prefer frontend build tools usually in **Node.js**.

### 1. Basic Asset Loading
**Problem:** Managing and loading different types of assets
- CSS files
- JavaScript files
- Images
- Fonts

**Pain Points:**
- Generating correct URLs
- Maintaining proper paths
- Handling different asset types
- Serving with correct headers

### 2. Cache Busting
**Problem:** Browser caching of outdated assets
- Files change but browsers serve cached versions
- Need to force reload when files update
- Cache invalidation strategies

**Pain Points:**
- When to invalidate cache
- How to version assets
- Maintaining cache headers
- Balancing cache vs fresh content

### 3. Development vs Production
**Problem:** Different needs for different environments
- Development needs quick feedback
- Production needs optimization
- Different serving strategies

**Pain Points:**
- Fast development cycle
- Production optimization
- Environment-specific configs
- Switching between environments

### 4. Asset Organization
**Problem:** Managing multiple assets efficiently
- Multiple CSS files
- Multiple JS files
- Page-specific assets
- Shared assets

**Pain Points:**
- Organizing assets logically
- Managing dependencies
- Loading order
- Code splitting

### 5. Module Dependencies
**Problem:** Managing JavaScript module dependencies
- Files depending on other files
- Loading order matters
- Mix of local and CDN files

**Pain Points:**
- Dependency management
- Module resolution
- Avoiding Node.js
- Browser compatibility

### 6. Performance
**Problem:** Optimizing asset delivery
- Multiple HTTP requests
- Large file sizes
- Slow loading times
- Browser limitations

**Pain Points:**
- Reducing requests
- File size optimization
- Loading strategies
- HTTP/2 optimization

Read further to gain an understanding of how **Asset** utility solves above challenges and reduces the need to depend on external build tools.

## Asset URL Generation & Versioning

```php
use Lightpack\Utils\Asset;

$asset = new Asset(); // Defaults to assets in public directory

// Asset url for public/css/app.css file
$url = $asset->url('css/app.css'); 
```

- By default, URLs include a version query string (`?v=...`) for cache busting.
- Version is read from a manifest (`assets.json`) or falls back to file modification time.

**Disable versioning if needed:**

```php
$url = $asset->url('css/app.css', false); // /css/app.css
```

**CDN Support:**
- Set `ASSET_URL` or `APP_URL` in your environment to prefix all asset URLs with a CDN or base URL.

```php
$asset->url('css/app.css'); // https://cdn.example.com/css/app.css?v=...
```

---

## HTML Helpers

### CSS & JS
```php
// Render CSS and JS tags
$html = $asset->load(['css/app.css', 'js/app.js']);
// <link rel='stylesheet' ...><script src='...' defer></script>
```

JS files use `defer` attribute by default; pass `async` or null for other modes. Choose one based on the script's purpose:
- **async**: For independent scripts (analytics, ads)
- **defer**: For scripts that need DOM or depend on other scripts

If neither **defer** nor *async* is used, the script is treated as "render-blocking" which means:

- Blocking Behavior
- HTML parsing stops
- Script is downloaded
- Script executes immediately
- Then HTML parsing continues
  
This is useful when you have scripts that:

- Must execute before anything else
- Need to be available immediately
- Are critical for initial page functionality

### Images
```php
// Render an <img> tag with attributes
$html = $asset->img('img/logo.png', ['alt' => 'Logo', 'width' => 200]);
// <img src='/img/logo.png?v=...' alt='Logo' width='200'>
```

### ES Modules
```php
// Render <script type="module"> for JS modules
$html = $asset->module('js/app.js');
$html = $asset->module(['js/app.js' => null, 'js/utils.js' => 'async']);
```

### Import Maps
```php
// Generate an import map for ES modules
$html = $asset->importMap([
    'uikit' => 'js/uikit.js',
    'app' => 'js/app.js',
]);
// <script type='importmap'>...</script>
```

---

## Google Fonts Downloader

```php
// Download and locally host a Google Font
$asset->googleFont('Roboto', [400, 700]);
// Downloads font files, generates local CSS, and updates manifest
```

- Fonts are saved to `/fonts`, CSS to `/css/fonts.css`.
- Only the latin subset is downloaded for efficiency.
- Throws on download or file errors.

---


## Version Manifest Management

- The utility tracks asset versions for files in `css/`, `js/`, `fonts/`, and `img/` directories by default.
- Use `generateVersions()` to scan and update the `assets.json` manifest:
- Only files in tracked directories are included in the manifest.

```php
$asset->generateVersions();
```

## Asset Management Best Practices
- Use versioned URLs for all assets to ensure users always get the latest files after deploys.
- Use a CDN for production by setting `ASSET_URL`.
- Regenerate the manifest (`generateVersions()`) after deploying new or updated assets.
- Organize assets in tracked directories for automatic versioning.
- Use the HTML helpers to keep your templates clean and DRY.

---