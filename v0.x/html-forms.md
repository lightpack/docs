# HTML Forms

<p class="tip">Lightpack provides a <code>Form</code> class for generating HTML form elements programmatically. Every method returns a raw HTML string — no wrappers, no divs, no extra markup — giving you complete control over layout and styling. It facilitates creating:</p>

- Sticky forms based on previous request input.
- CSRF token field automatically included.
- Pre-rendered error messages for validation.
- Type-specific inputs (email, password, number, etc.).
- Custom attributes for any input type.
- Handling multiple radio buttons and checkboxes.
- Flexible form structure for custom layouts.

---

## Quick Start

```php
$form = new \Lightpack\Html\Form;
```

A quick form example for reference:

```php
<?= $form->open('/profile') ?>

    <?= $form->label('Full Name') ?>
    <?= $form->input('name') ?>
    <?= $form->error('name') ?>

    <?= $form->label('Email') ?>
    <?= $form->email('email') ?>
    <?= $form->error('email') ?>

    <?= $form->label('Role') ?>
    <?= $form->select('role', ['admin' => 'Admin', 'editor' => 'Editor']) ?>

    <?= $form->submit('Save Changes') ?>

<?= $form->close() ?>
```

---

## Opening and Closing Forms

### open()

```php
$form->open(string $action = '', string $method = 'POST', array $attrs = [], bool $csrf = true): string
```

- Generates the opening `<form>` tag. 
- Default method is `POST`.
- Includes a CSRF token by default and handles method spoofing for `PUT`, `PATCH`, and `DELETE`.

```php
<?= $form->open('/users') ?>
<?= $form->open('/users/1', 'PUT') ?>
<?= $form->open('/search', 'GET', ['class' => 'search-form']) ?>
```

### openMultipart()

Same as `open()`, but sets `enctype="multipart/form-data"` — required for file uploads.

```php
<?= $form->openMultipart('/profile') ?>
```

### close()

```php
<?= $form->close() ?>
```

Outputs `</form>`.

---

## Text Input

```php
$form->input(string $name, array $attrs = []): string
```

Generates a text input. Pass any HTML attribute via `$attrs`.

```php
<?= $form->input('username') ?>
<?= $form->input('username', ['placeholder' => 'Enter username', 'class' => 'form-control']) ?>
```

To set a default value when no session data exists:

```php
<?= $form->input('username', ['value' => 'john']) ?>
```

<p class="tip">If validation fails and the form is resubmitted, the previously entered value is automatically restored. This makes it easy to repopulate the form with the user's input making sticky forms a breeze.</p>

---

## Type-Specific Inputs

These all behave identically to `input()` but emit the correct `type` attribute according to the method name:

```php
$form->email(string $name, array $attrs = []): string
$form->number(string $name, array $attrs = []): string
$form->tel(string $name, array $attrs = []): string
$form->url(string $name, array $attrs = []): string
$form->date(string $name, array $attrs = []): string
$form->search(string $name, array $attrs = []): string
$form->color(string $name, array $attrs = []): string
```

Examples:

```php
<?= $form->email('email') ?>
<?= $form->number('qty', ['min' => 1, 'max' => 99]) ?>
<?= $form->date('birthdate') ?>
<?= $form->tel('phone', ['placeholder' => '+1 555 0100']) ?>
```

---

## Password

```php
$form->password(string $name, array $attrs = []): string
```

Generates a password input. **The value is never repopulated from previous input**, regardless of session data. This is intentional — password fields should always be blank on reload.

```php
<?= $form->password('password') ?>
<?= $form->password('password', ['autocomplete' => 'new-password']) ?>
```

---

## Textarea

```php
$form->textarea(string $name, array $attrs = []): string
```

```php
<?= $form->textarea('bio') ?>
<?= $form->textarea('bio', ['rows' => 6, 'placeholder' => 'Tell us about yourself']) ?>
```

---

## Select Dropdown

```php
$form->select(string $name, array $options, array $attrs = []): string
```

Pass a flat key-value array for options:

```php
<?= $form->select('country', ['us' => 'United States', 'uk' => 'United Kingdom', 'de' => 'Germany']) ?>
```

To pre-select an option:

```php
<?= $form->select('country', ['us' => 'United States', 'uk' => 'United Kingdom'], ['selected' => 'us']) ?>
```

### Option Groups

Wrap options in a nested array keyed by the group label:

```php
<?= $form->select('city', [
    'Europe' => ['ldn' => 'London', 'ber' => 'Berlin', 'par' => 'Paris'],
    'Asia'   => ['tok' => 'Tokyo', 'mum' => 'Mumbai'],
]) ?>
```

You can also mix flat options and groups:

```php
<?= $form->select('area', [
    'all' => 'All Areas',
    'Europe' => ['uk' => 'UK', 'de' => 'Germany'],
]) ?>
```

### Multiple Select

```php
$form->selectMultiple(string $name, array $options, array $attrs = []): string
```

```php
<?= $form->selectMultiple('interests', ['coding' => 'Coding', 'music' => 'Music', 'art' => 'Art']) ?>
```

To pre-select multiple options:

```php
<?= $form->select('interests', ['coding' => 'Coding', 'music' => 'Music', 'art' => 'Art'], [
    'multiple' => true,
    'selected' => ['coding', 'art'],
]) ?>
```

---

## Checkbox

```php
$form->checkbox(string $name, mixed $value = 1, array $attrs = []): string
```

For a single checkbox, use `checkbox()`. It emits a hidden input alongside the checkbox so the field is always present in the POST data — even when unchecked. This is intentional to ensure the field is always present in the POST data because by default, unchecked checkboxes are not sent in the POST data. That results in the field not being present in the POST data when the checkbox is unchecked.

```php
<?= $form->checkbox('agree', 1) ?>
```

To mark it checked by default (when no previous input exists):

```php
<?= $form->checkbox('agree', 1, ['checked' => true]) ?>
```

### Checkbox Group

```php
$form->checkboxes(string $name, array $options, array $attrs = []): string
```

For a group of checkboxes sharing the same array name, use `checkboxes()`. Unlike `checkbox()`, it does not emit hidden inputs. This means if the user checks nothing, the field is absent from `$_POST` entirely — PHP does not send unchecked checkboxes. Always handle this in your controller:

```php
$interests = request()->input('interests', []);
```

```php
<?= $form->checkboxes('interests[]', [
    'coding' => 'Coding',
    'music'  => 'Music',
    'art'    => 'Art',
]) ?>
```

Each checkbox and its label text are wrapped in a `<label>` element — clicking the text checks the checkbox.

---

## Radio Button

```php
$form->radio(string $name, mixed $value, array $attrs = []): string
```

```php
<?= $form->radio('gender', 'male') ?> Male
<?= $form->radio('gender', 'female') ?> Female
```

To mark one checked by default (when no previous input exists):

```php
<?= $form->radio('gender', 'male', ['checked' => true]) ?> Male
<?= $form->radio('gender', 'female') ?> Female
```

### Radio Group

```php
$form->radios(string $name, array $options, array $attrs = []): string
```

Each radio and its label text are wrapped in a `<label>` element — clicking the text checks the radio. No radio is pre-selected unless `old()` data exists from a previous submission.

```php
<?= $form->radios('gender', ['male' => 'Male', 'female' => 'Female']) ?>
```

For a **default selection** or **custom markup**, use individual `radio()` calls:

```php
<?php foreach (['male' => 'Male', 'female' => 'Female'] as $val => $label): ?>
    <label class="radio-option">
        <?= $form->radio('gender', $val, ['checked' => $val === 'male']) ?>
        <?= $label ?>
    </label>
<?php endforeach ?>
```

---

## File Upload

```php
$form->file(string $name, array $attrs = []): string
```

The value is **never repopulated** from previous input — browsers do not allow pre-filling file inputs.

```php
<?= $form->file('avatar') ?>
<?= $form->file('avatar', ['accept' => 'image/*']) ?>
```

Use `openMultipart()` when your form has file fields.

---

## Hidden Input

```php
$form->hidden(string $name, ?string $value = null, array $attrs = []): string
```

Hidden inputs are never repopulated from session data. The value you pass is always used.

```php
<?= $form->hidden('user_id', (string) $user->id) ?>
<?= $form->hidden('_source', 'dashboard') ?>
```

To emit a hidden input with no value:

```php
<?= $form->hidden('referral') ?>
```

---

## Label

```php
$form->label(string $text, string $for = '', array $attrs = []): string
```

```php
<?= $form->label('Email Address', 'email') ?>
<?= $form->label('Bio', 'bio', ['class' => 'required']) ?>
<?= $form->label('Terms') ?>
```

---

## Submit Button

```php
$form->submit(string $text, array $attrs = []): string
```

```php
<?= $form->submit('Save Changes') ?>
<?= $form->submit('Delete', ['class' => 'btn-danger']) ?>
```

---

## Button Element

```php
$form->button(string $text, array $attrs = []): string
```

Generates a `<button>` element. Useful for JavaScript-driven actions or custom styling.

```php
<?= $form->button('Cancel', ['type' => 'button', 'onclick' => 'history.back()']) ?>
```

---

## Datalist (Autocomplete)

```php
$form->datalist(string $id, array $options): string
```

Generate a `<datalist>` element for providing autocomplete suggestions to an input field.

```php
<?= $form->input('city', ['list' => 'cities']) ?>
<?= $form->datalist('cities', ['New York', 'London', 'Tokyo', 'Berlin']) ?>
```

---

## Validation Errors

```php
$form->error(string $name): string
```

Returns the validation error message for a field, or an empty string if there is none.

```php
<?= $form->input('email') ?>
<?= $form->error('email') ?>
```

Wrap in your own markup:

```php
<?php $err = $form->error('email'); ?>
<?php if ($err): ?>
    <p class="error"><?= $err ?></p>
<?php endif ?>
```

Works with array field names too:

```php
<?= $form->error('user[email]') ?>
```

---

## Sticky Forms

When a form is submitted and validation fails, **Lightpack** stores the submitted input in the session. On the next render, **all form methods automatically restore the previously submitted values** — you do not need to call `old()` manually on each field.

The only exceptions are:
- **`password()`** — never repopulates, for security.
- **`file()`** — browsers do not allow pre-filling file inputs.
- **`hidden()`** — always uses the value you explicitly pass.

---

## Array Field Names

Field names using bracket notation (e.g., `user[name]`, `user[address][city]`) are fully supported.

```php
<?= $form->input('user[name]') ?>
<?= $form->error('user[name]') ?>

<?= $form->input('user[address][city]') ?>
<?= $form->error('user[address][city]') ?>

<?= $form->checkboxes('interests[]', ['php' => 'PHP', 'js' => 'JavaScript']) ?>
```

---

## Passing HTML Attributes

Every method accepts an `$attrs` array as its last parameter. Pass any valid HTML attribute as a key-value pair:

```php
<?= $form->input('email', [
    'class'       => 'form-control',
    'placeholder' => 'you@example.com',
    'required'    => true,
    'autofocus'   => true,
]) ?>
```

**Boolean attributes** (`required`, `autofocus`, `disabled`, `readonly`, `multiple`) use `true` and `false`:

| Value | Output |
|-------|--------|
| `true` | Attribute emitted with no value: `disabled` |
| `false` | Attribute omitted entirely |
| `null` | Attribute omitted entirely |

```php
<?= $form->input('qty', ['required' => true, 'disabled' => false]) ?>
// → <input ... required>
```

This makes conditional attributes natural — just pass a boolean expression:

```php
<?= $form->input('ref', ['disabled' => $isLocked, 'required' => $isRequired]) ?>
```

```php
<?= $form->input('code', ['oninput' => 'validate(this.value)']) ?>
<?= $form->button('Reset', ['type' => 'button', 'onclick' => 'resetForm()']) ?>
```

---