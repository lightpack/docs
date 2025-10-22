# Lightpack View Template System

The `Lightpack\View\Template` class is the core of Lightpack's view rendering system. It provides a minimal, robust, and flexible API for rendering PHP templates, including support for data injection, template composition, layout inheritance, and content stacking.

Use the helper function `template()` to utilize the view templating capabilities.

<p class="tip">Lightpack expects all your view templates to be located in the <b>views</b> directory of your project. You can organize your layouts, partials, and page templates here, using subdirectories as needed.</p>

## Available Methods

### include()

`include(string $file, array $data = []): string`

- **Purpose:** Include and render a template file with provided data.
- **Behavior:** Merges provided data with parent data and renders the template.
- **Usage:**
  ```php
  <?= template()->include('profile', ['user' => $user]) ?>
  <?= template()->include('partials/header', ['title' => 'Home']) ?>
  ```

### component()

`component(string $file, array $data = []): string`

- **Purpose:** Render a template as a component, using only the provided data (does not merge with internal data).
- **Difference from `include`:** No data merging with parent; only provided data is available.
- **Usage:**
  ```php
  <?= template()->component('components/button', ['type' => 'submit']) ?>
  ```

---

## Understanding Differences

These methods all render templates, but their behavior around data merging and intended use is different. Here's a practical breakdown to make their differences clear:

- **include()**
  - The primary method for template composition.
  - Used for main views, partials, and nested templates.
  - Merges instance data with provided data.
  - Think of this as "include this template with this data."
- **component()**
  - Used for stateless, reusable UI pieces (buttons, alerts, cards).
  - Think of this as "render a widget with only the data I give it." 
  - It does **NOT** merge with parent data—only what you pass is available.
  - Perfect for isolated, reusable components.

### Example

Include the `views/profile.php` template with passed data.

```php
<?= template()->include('profile', [
    'user' => 'Alice', 
    'role' => 'admin',
    'theme' => 'dark'
]);
```

In `views/profile.php` template:
```php
<h1>Profile</h1>

<!-- views/partials/header.php -->
<?= template()->include('partials/header', [
    'page' => 'Profile'
]) ?>

<!-- views/components/button.php -->
<?= template()->component('components/button', [
    'label' => 'Save'
]) ?>
```

**Data available in each method:**

| Method      | Available Data           |
|-------------|--------------------------------------|
| include()     | user, role, theme, page              |
| component()   | label                                |

- `include` merges parent data + provided arguments.
- `component` only gets what you pass—no parent data.

### Decision Table

| Use case                        | Method      |
|---------------------------------|-------------|
| Include a full page             | include     |
| Insert a partial (header, nav)  | include     |
| Render a stateless widget       | component   |
| Need parent data in template    | include     |
| Want only explicit data         | component   |

> **Tip:** Use `include()` for all template composition (main views, partials, nested templates). Use `component()` for isolated, reusable widgets that shouldn't access parent data.

---

## Practical Usage in Controllers

Typically, templates are rendered as HTML responses directly from controller actions.

```php
class ProfileController
{
    public function show()
    {
        return response()->view('profile', [
            'name' => 'John Doe',
            'email' => 'johndoe@example.com',
        ]);
    }
}
```

This will look for the `views/profile.php` template file and render it with passed data as second argument.

```php
<ul>
    <li>Name: <?= $name ?></li>
    <li>Email: <?= $email ?></li>
</ul>
```
---

## Layout Inheritance

Layout inheritance allows child templates to declare which layout they want to use, and layouts render the child content where specified.

### layout()

`layout(string $file): void`

- **Purpose:** Declare which layout file should wrap the current template.
- **Usage:** Call this at the top of your child template.
- **Pattern:** Child declares parent, parent renders child.

```php
<?= template()->layout('layouts/app') ?>

<h1>Dashboard</h1>
<p>Welcome back!</p>
```

### content()

`content(): string`

- **Purpose:** Render the child template's content within a layout.
- **Usage:** Call this in your layout file where you want the child content to appear.

```php
<!DOCTYPE html>
<html>
    <body>
        <main>
            <?= template()->content() ?>
        </main>
    </body>
</html>
```

### Complete Example

**Controller:**

```php
class DashboardController
{
    public function index()
    {
        return response()->view('dashboard', [
            'title' => 'Dashboard',
            'stats' => ['sales' => 23, 'leads' => 100],
        ]);
    }
}
```

**Directory Structure:**

```
views/
├── layouts/
│   └── app.php          # Main layout
├── dashboard.php        # Dashboard page
└── partials/
    ├── header.php
    └── footer.php
```

**views/dashboard.php** (Child Template):

```php
<?= template()->layout('layouts/app') ?>

<div class="dashboard">
    <h1><?= $title ?></h1>
    <p>Total Sales: <?= $stats['sales'] ?></p>
    <p>Total Leads: <?= $stats['leads'] ?></p>
</div>
```

**views/layouts/app.php** (Layout):

```php
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>My App</title>
</head>
<body>
    <?= template()->include('partials/header') ?>
    
    <main>
        <?= template()->content() ?>
    </main>
    
    <?= template()->include('partials/footer') ?>
</body>
</html>
```

### Nested Layouts

You can nest layouts multiple levels deep. Each template declares its immediate parent.

**views/dashboard.php:**
```php
<?= template()->layout('layouts/admin') ?>
<h1>Dashboard</h1>
```

**views/layouts/admin.php:**
```php
<?= template()->layout('layouts/app') ?>

<div>
    <aside>Admin Sidebar</aside>
    <div>
        <?= template()->content() ?>
    </div>
</div>
```

**views/layouts/app.php:**
```php
<!DOCTYPE html>
<html>
<body>
    <?= template()->content() ?>
</body>
</html>
```

**Result:** `app.php` wraps `admin.php`, which wraps `dashboard.php`.

---

## Stacks (Assets Management)

Stacks allow child templates to push content (like CSS/JS links) into specific locations in parent layouts. This is perfect for managing page-specific assets.

### push()

`push(string $stack): void`

- **Purpose:** Start pushing content to a named stack.
- **Usage:** Call this before the content you want to stack, then call `endPush()` after.

### endPush()

`endPush(): void`

- **Purpose:** Stop pushing and save the content to the stack.
- **Usage:** Always pair with `push()`.

### stack()

`stack(string $stack): string`

- **Purpose:** Render all content that has been pushed to a stack.
- **Usage:** Call this in your layout where you want the stacked content to appear.
- **Behavior:** Returns all pushed content joined with newlines.

### Example: Page-Specific Assets

**views/layouts/app.php:**

```php
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>My App</title>
    
    <!-- Common CSS -->
    <link rel="stylesheet" href="/css/app.css">
    
    <!-- Page-specific styles stack -->
    <?= template()->stack('styles') ?>
</head>
<body>
    <?= template()->content() ?>
    
    <!-- Common JS -->
    <script src="/js/app.js"></script>
    
    <!-- Page-specific scripts stack -->
    <?= template()->stack('scripts') ?>
</body>
</html>
```

**views/dashboard.php:**

```php
<?= template()->layout('layouts/app') ?>

<?php template()->push('styles') ?>
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
<link rel="stylesheet" href="/css/dashboard.css">
<?php template()->endPush() ?>

<?php template()->push('scripts') ?>
<script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
<script src="/js/dashboard.js"></script>
<?php template()->endPush() ?>

<h1>Dashboard</h1>
<p>Your dashboard content here...</p>
```

**Rendered Output:**

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>My App</title>
    <link rel="stylesheet" href="/css/app.css">
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="/css/dashboard.css">
</head>
<body>
    <h1>Dashboard</h1>
    <p>Your dashboard content here...</p>
    
    <script src="/js/app.js"></script>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <script src="/js/dashboard.js"></script>
</body>
</html>
```

### Multiple Pushes to Same Stack

You can push to the same stack multiple times—all content accumulates:

```php
<?php template()->push('scripts') ?>
<script src="/js/charts.js"></script>
<?php template()->endPush() ?>

<!-- Later in the same template -->

<?php template()->push('scripts') ?>
<script src="/js/analytics.js"></script>
<?php template()->endPush() ?>
```

Both scripts will be rendered when `stack('scripts')` is called.

### Common Stack Names

| Stack Name | Purpose | Location in Layout |
|------------|---------|--------------------|
| `styles` | CSS files and `<style>` tags | In `<head>` |
| `scripts` | JavaScript files and `<script>` tags | Before `</body>` |
| `meta` | Meta tags | In `<head>` |
| `head` | Any other head content | In `<head>` |

---

## Escaping Output

Lightpack provides a convenient global function, `_e()`, to help you safely output dynamic data in your templates:

- It converts special characters to their HTML entity equivalents.
- Prevents Cross-Site Scripting (XSS) by ensuring user-supplied or dynamic data cannot break out of HTML context.
- Use it whenever you output any data that may contain special HTML characters or come from user input.

### ⚠️ Security: Always Escape User Input

**DANGEROUS (XSS Vulnerability):**
```php
<!-- ❌ Never do this with user input! -->
<div><?= $userComment ?></div>
<h1><?= $userName ?></h1>
<a href="<?= $userUrl ?>">Link</a>
```

**SAFE (Properly Escaped):**
```php
<!-- ✅ Always escape user-provided data -->
<div><?= _e($userComment) ?></div>
<h1><?= _e($userName) ?></h1>
<a href="<?= _e($userUrl) ?>">Link</a>
```

### When to Escape

**Always escape:**
- ✅ User-submitted data (comments, posts, names, bios)
- ✅ Database content that users can edit
- ✅ URL parameters and query strings
- ✅ Form input values
- ✅ Any data from external sources (APIs, files, etc.)

**Do NOT escape:**
- ❌ HTML you control and trust (like template includes)
- ❌ Data that's already been escaped
- ❌ Output from `template()->include()` or `template()->component()`

### Example

```php
<?php foreach($comments as $comment): ?>
    <div class="comment">
        <strong><?= _e($comment->author) ?></strong>
        <p><?= _e($comment->text) ?></p>
    </div>
<?php endforeach ?>
```

## Few Suggestions

Using PHP as a template engine doesn’t mean you have to write messy or “spaghetti” code. Follow these principles for clean, maintainable views:

**Keep business logic out of templates.**
- All data preparation, decisions, and calculations should happen in your controllers or services.
- Templates should focus only on presentation: displaying variables, looping over data, and calling includes/components.
- This is the single biggest reason why templates become messy.

**Use Partials and Components**
- Break up large templates into smaller, reusable pieces (headers, footers, navbars, widgets).
- Use `template()->include()` and `template()->component()` to compose your UI from simple building blocks.

**Leverage Layouts**
- Define a base layout for your site and inject page content into it.
- This keeps your structure consistent and your page templates focused.

**Minimize Inline PHP**
- Use short echo tags (`<?= ... ?>`) for output.
- Avoid complex `if`/`else` or function calls in templates—prepare everything in advance.

**Name Variables Clearly**
- Use descriptive variable names so your templates are self-explanatory.

**Escape Output Where Needed**
- Always escape user-provided data with `_e()`, unless you are certain it’s safe.

---