# Lightpack View Template System

The `Lightpack\View\Template` class is the core of Lightpack’s view rendering system. It provides a minimal, robust, and flexible API for rendering PHP templates, including support for data injection, template composition, conditional inclusion, and error handling. This documentation provides an exhaustive breakdown of its design, usage, and internals.

Use the helper function `template()` to utilize the view templating capabilities.

<p class="tip">Lightpack expects all your view templates to be located in the <b>views</b> directory of your project. You can organize your layouts, partials, and page templates here, using subdirectories as needed.</p>

## Available Methods

### include()

`include(string $file, array $data = []): string`

- **Purpose:** Include and render a template file with provided data.
- **Behavior:** Merges provided data with instance data and renders the template.
- **Usage:**
  ```php
  <?= template()->include('profile', ['user' => $user]) ?>
  <?= template()->include('partials/header', ['title' => 'Home']) ?>
  ```

### includeIf()

`includeIf(bool $flag, string $file, array $data = []): string`

- **Purpose:** Conditionally include a sub-template based on a boolean flag.
- **Pattern:**
  - If `$flag` is `true`, includes the template (via `include`).
  - If `false`, returns an empty string.
- **Usage:**
  ```php
  <?= template()->includeIf($user->isAdmin(), 'partials/admin', ['user' => $user]) ?>
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

### Variable Scope Isolation

**All templates have isolated variable scope**, regardless of which method you use. Variables defined or modified in one template cannot affect another template.

```php
// In parent.php:
<?php $title = 'Parent Title'; ?>
<?= template()->include('child') ?>
<?= $title ?>  <!-- Still "Parent Title" -->

// In child.php:
<?php $title = 'Child Title'; ?>  <!-- Does NOT affect parent -->
``` 

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

## Building Layouts

Layouts enable you to define a common structure for your pages (such as headers, footers, navigation, and meta tags) while allowing individual templates to inject their own content.

There is a special method for rendering embedded child content inside a layout:

```php
template()->embed()
```

### 1. Controller

Specify the `__embed` key in view data with the name of the template you expect to be embedded in **layout.php** view.

```php
public function dashboard()
{
    return response()->view('layout', [
        '__embed' => 'dashboard', // dashboard template
        'title'   => 'Dashboard',
        'copyright' => '@Lightpack',
        'stats'    => ['sales' => 23, 'leads' => 100],
    ]);
}
```

> All the data is automatically passed to **layout.php** except `__embed` which is unset by the framework when rendering the view template.

### 2. Define Templates

An example directory structure for defining view templates might look like:

```
app/
└── views/
    ├── layout.php           # Main layout template
    ├── dashboard.php        # Dashboard content template
    └── partials/
        └── header.php
        └── footer.php
```

- **layout.php**: Your main layout file (includes header, footer, and an embed slot for child content).
- **header.php, footer.php**: Common partials included in layouts or other templates.
- **dashboard.php**: referenced as embedded content inside layout.

`views/partials/header.php`

```php
<header>
    <?= $title ?>
</header>
```

`views/partials/footer.php`

```php
<footer>
    <?= $copyright ?>
</footer>
```

`views/dashboard.php`

```php
Total Sales: <?= $stats['sales'] ?>
Total Leads: <?= $stats['leads'] ?>
```

`views/layout.php`

```php
<!DOCTYPE html>
<html>
    <body>
        <!-- include header template -->
        <?= template()->include('partials/header') ?>

        <main>
            <!-- embed child content -->
            <?= template()->embed() ?>
        </main>

        <!-- include footer template -->
        <?= template()->include('partials/footer') ?>
    </body>
</html>
```

**Observe that we've embed the `views/dashboard.php` template using:**

`<?= template()->embed() ?>`

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