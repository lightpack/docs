# Localization

Build multilingual applications with ease using Lightpack's built-in localization system.

- **File-based translations** — PHP array files, no external formats
- **Dot notation** — `messages.hello` maps to `lang/en/messages.php` → `hello`.
- **Placeholders** — `:name`, `:count` replacement.
- **Pluralization** — `choice('messages.items', 5)` with pipe syntax.
- **Fallback** — missing keys fall back to default locale automatically.
- **Validation messages** — override validation error messages with your own translations.

## Quick Start

### Create translation files

Run the following command to create the default language directory:

```bash
php console create:lang
```

It will prompt you to enter the language locale (e.g., `en`, `hi`, `es`) and translation file name to be created.

### Use in views or controllers

```php
// Simple translation
lang('messages.hello');           // 'Hello' (or 'नमस्ते' in hi)

// With placeholders
lang('messages.welcome', ['name' => 'John']);  // 'Welcome, John!'

// Pluralization
lang()->choice('messages.items', 5);  // '5 items'

// Nested arrays
lang('forms.signup.title');   // 'Sign Up' — reads lang/en/forms.php → ['signup']['title']
```

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

## API

| Method | Description |
|--------|-------------|
| `lang('key')` | Get translation string |
| `lang('key', ['name' => 'John'])` | Get with placeholder replacement |
| `lang()->choice('key', 5)` | Pluralized translation |
| `lang()->choice('key', 5, [], 'fr')` | Pluralized with locale override |
| `lang()->has('key')` | Check if translation exists |
| `lang()->setLocale('hi')` | Change locale manually |
| `lang()->getLocale()` | Get current locale |
| `lang()->setLocaleRule('xx', fn($n) => ...)` | Register custom plural rule |

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