# Lightpack View Template System

The `Lightpack\View\Template` class is the core of Lightpack’s view rendering system. It provides a minimal, robust, and flexible API for rendering PHP templates, including support for data injection, template composition, conditional inclusion, and error handling. This documentation provides an exhaustive breakdown of its design, usage, and internals.

Use the helper function `template()` to utilize the view templating capabilities.

## Available Methods

### render()

`render(string $file, array $data = []): string`

- **Purpose:** Render a template file with provided data.
- **Usage:**
  ```php
  <?= template()->render('profile', ['user' => $user]) ?>
  ```

### include()

`include(string $file, array $data = []): string`

- **Purpose:** Render a sub-template (partial) with provided data, 
- Also has access to parent template but overrides data with same matching keys.
- However, the data set in the included template does not affect the parent template’s data.
- **Usage:**
  ```php
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

These methods all render templates, but their behavior around data merging, isolation, and intended use is different. Here’s a practical breakdown to make their differences clear:

- **render()**
  - Think of this as "show me a complete page." 
  - Used for top-level views.
- **include()**
  - Like "inserting a partial inside another template." 
  - It creates a new, isolated template instance, merges the parent’s data with any additional data, and renders the partial. 
  - Used for reusable bits (headers, nav, footers).
- **component()**
  - Used for stateless, reusable UI pieces (buttons, alerts).
  - Think of this as "render a widget with only the data I give it." 
  - It does **NOT** merge with parent data—only what you pass is available. 

### Example

Render the `app/views/profile.php` template with passed data.

```php
<?= template()->render('profile', [
    'user' => 'Alice', 
    'role' => 'admin',
    'theme' => 'dark'
]);
```

In `app/view/profile.php` template:
```php
<h1>Profile</h1>

<!-- app/views/partials/header.php -->
<?= template()->include('partials/header', [
    'page' => 'Profile'
]) ?>

<!-- app/views/components/header.php -->
<?= template()->component('components/button', [
    'label' => 'Save'
]) ?>
```

**Data available in each method:**

| Method      | Available Data           |
|-------------|--------------------------------------|
| render()      | user, role, theme                    |
| include()     | user, role, theme, page              |
| component()   | label                                |

- `render` acts as parent template with access to all data passed.
- `include` merges parent data + provided arguments (for the partial).
- `component` only gets what you pass—no parent data.

### Decision Table

| Use case                        | Method      |
|---------------------------------|-------------|
| Render a full page              | render      |
| Insert a partial (header, nav)  | include     |
| Render a stateless widget       | component   |
| Need parent data in partial     | include     |
| Want only explicit data         | component   |

> **Tip:** If you want maximum isolation and reusability, use `component`. For partials that need context, use `include`. For the main view, use `render`.

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

This will look for the `app/views/profile.php` template file and render it with passed data as second argument.

```php
<ul>
    <li>Name: <?= $name ?></li>
    <li>Email: <?= $email ?></li>
</ul>
```
---

## Building Layouts

Layouts enable you to define a common structure for your pages (such as headers, footers, navigation, and meta tags) while allowing individual templates to inject their own content.

### 1. Controller
```php
public function dashboard()
{
    $data = [
        'title'   => 'Dashboard',
        'stats'    => ['sales' => 23, 'leads' => 100],
        'copyright' => '@Lightpack',
        'template' => 'dashboard',
    ];

    return response()->view('layout', $data);
}
```

### 2. Define Templates

`app/views/header.php`

```php
<header>
    <?= $title ?>
</header>
```

`app/views/footer.php`

```php
<footer>
    <?= $copyright ?>
</footer>
```

`app/views/dashboard.php`

```php
Total Sales: <?= $stats['sales'] ?>
Total Leads: <?= $stats['leads'] ?>
```

`app/views/layout.php`

```php
<!DOCTYPE html>
<html>
    <body>
        <!-- include header template -->
        <?= template()->include('header') ?>

        <main>
            <!-- include template to render -->
            <?= template()->include($template) ?>
        </main>

        <!-- include footer template -->
        <?= template()->include('footer') ?>
    </body>
</html>
```

**Observe that we include the `app/view/dashboard.php` template using:**

`<?= template()->include($template) ?>`

---

## Escaping Output

Lightpack provides a convenient global function, `_e()`, to help you safely output dynamic data in your templates:

- It converts special characters to their HTML entity equivalents.
- Prevents Cross-Site Scripting (XSS) by ensuring user-supplied or dynamic data cannot break out of HTML context.
- Use it whenever you output any data that may contain special HTML characters or come from user input.

```php
<?php foreach($comments as $comment): ?>
    <?= _e($comment) ?>
<?php endforeach  ?>
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