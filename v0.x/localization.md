# Localization

Build multilingual applications with ease using Lightpack's built-in localization system.

- **File-based translations** — Plain PHP array files.
- **Dot notation** — `messages.hello` maps to `lang/en/messages.php` → `hello`.
- **Placeholders** — `:name`, `:count` replacement.
- **Pluralization** — `choice('messages.items', 5)` with pipe syntax.
- **Fallback** — missing keys fall back to the app fallback locale.
- **Validation messages** — override validation error messages with your own translations.

## Quick Start

### Create translation files

Run the following command to create a new language translation file:

```bash
php console create:lang
```

It will prompt you to enter the language locale (e.g., `en`, `hi`, `es`) and translation file name to be created.

Below is an example of the translation file structure `lang/en/messages.php`:

```php
return [
    'hello' => 'Hello',
    'welcome' => 'Welcome, :name!',
    'items' => ':count item|:count items',
];
```

### Use in views or controllers

```php
// Simple translation
lang('messages.hello');           // 'Hello' (or 'नमस्ते' in hi)

// With placeholders
lang('messages.welcome', ['name' => 'John']);  // 'Welcome, John!'

// Pluralization
lang()->choice('messages.items', 5);  // '5 items'

```

> **No-dot keys:** `lang('hello')` (without a dot) defaults to `messages.php`: `lang/en/messages.php → 'hello'`.

> **Missing keys:** If a key is missing in both the current locale and the fallback locale, the key string itself is returned (e.g., `'messages.hello'`).

## Configuration

Localization settings live in the `lang` block inside `config/app.php`:

```php
// config/app.php
return [
    'app' => [
        // ...
        'lang' => [
            'default'   => get_env('APP_LOCALE', 'en'),
            'fallback'  => get_env('APP_FALLBACK_LOCALE', 'en'),
        ],
    ],
];
```

## Locale Management

You can check or change the active locale at runtime:

```php
// Get the current locale
$lang = lang()->getLocale(); // 'en'

// Set a new locale for the current request
lang()->setLocale('hi');
```

## Validation Messages

You can create form validation translation files for each locale:

```bash
php console create:lang --support=validation
```

## Pluralization Syntax

### Simple (English-style singular/plural)

```php
'items' => ':count item|:count items',
```

- `choice('items', 1)` → `1 item`
- `choice('items', 5)` → `5 items`

### Indexed (Arabic, Russian, Polish, etc.)

For languages with more than two plural forms, prefix each form with `{index}`:

```php
// Russian — 3 forms
'articles' => '{0} :count статей|{1} :count статья|{2} :count статьи',
```

- `choice('articles', 1)` → `1 статья` (form 1)
- `choice('articles', 2)` → `2 статьи` (form 2)
- `choice('articles', 5)` → `5 статей` (form 0)

---