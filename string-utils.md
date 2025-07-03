# String Utilities

Lightpack’s `Str` utility class offers a comprehensive set of methods for string manipulation, formatting, validation, and generation. It is designed to cover common and advanced string operations with a clear, expressive API and robust error handling.

---

## Inflection & Naming

- **singularize() / pluralize()**: Convert between singular and plural forms.
  ```php
  $str->singularize('quizzes'); // quiz
  $str->pluralize('quiz');      // quizzes
  ```
- **pluralizeIf()**: Pluralizes only if a number > 1.
  ```php
  $str->pluralizeIf(2, 'role'); // roles
  ```
- **camelize() / variable()**: Convert to PascalCase or camelCase.
  ```php
  $str->camelize('my_class');   // MyClass
  $str->variable('MyClass');    // myClass
  ```
- **underscore() / dasherize()**: snake_case or kebab-case.
  ```php
  $str->underscore('MyClass');  // my_class
  $str->dasherize('MyClass');   // my-class
  ```
- **humanize() / headline()**: Human-readable or title-cased.
  ```php
  $str->humanize('lazy_brown_fox'); // Lazy brown fox
  $str->headline('lazyBrownFox');   // Lazy Brown Fox
  ```
- **tableize() / classify() / foreignKey() / ordinalize()**: Naming for DB tables, classes, keys, ordinals.
  ```php
  $str->tableize('UserGroup');   // user_groups
  $str->classify('user_groups'); // UserGroup
  $str->foreignKey('User');      // user_id
  $str->ordinalize(21);          // 21st
  ```

---

## Slugs & URL Helpers

- **slug()**: URL-friendly slugs from any string (UTF-8 safe, customizable separator).
  ```php
  $str->slug('Über grünen!'); // uber-grunen
  $str->slug('Hello World', '_'); // hello_world
  ```
- **slugify()**: (Deprecated) Use `slug()` instead.

---

## String Checks

- **startsWith() / endsWith() / contains()**: Substring checks.
  ```php
  $str->startsWith('admin/products', 'admin'); // true
  $str->endsWith('file.txt', '.txt');          // true
  $str->contains('hello world', 'world');      // true
  ```

---

## Random, Masking, and Truncation

- **random()**: Cryptographically secure random hex string.
  ```php
  $str->random(16); // e.g. '9f2d4b1a...'
  ```
- **mask()**: Obfuscate part of a string (e.g., credit cards).
  ```php
  $str->mask('KEYSECRET', '*', 3); // 'KEY***'
  ```
- **truncate() / limit() / excerpt()**: Truncate by length, words, or create readable excerpts.
  ```php
  $str->truncate('Hello World', 5); // Hello...
  $str->limit('one two three four', 2); // one two...
  $str->excerpt('This is a very long text', 10); // This is a...
  ```

---

## Padding & Casing

- **pad()**: Pad string to a given length (left, right, both).
- **title() / upper() / lower()**: Title, upper, or lower case (UTF-8 safe).
  ```php
  $str->title('über grünen'); // Über Grünen
  $str->upper('hello');       // HELLO
  $str->lower('HELLO');       // hello
  ```

---

## HTML & Content Helpers

- **escape()**: HTML-escape a string.
  ```php
  $str->escape('<h1>Hello</h1>'); // &lt;h1&gt;Hello&lt;/h1&gt;
  ```
- **strip()**: Remove all HTML tags (including script/style content).
  ```php
  $str->strip('<p>Hello</p>'); // Hello
  ```

---

## Validation Helpers

- **isEmail() / isUrl() / isIp() / isHex() / isUuid() / isDomain() / isBase64() / isMimeType() / isPath() / isJson()**
  ```php
  $str->isEmail('test@example.com'); // true
  $str->isHex('#ff0033');            // true
  $str->isJson('{"a":1}');         // true
  ```

---

## File & Path Helpers

- **filename() / stem() / ext() / dir()**: Extract file name, stem, extension, or directory.
  ```php
  $str->filename('/path/to/file.txt'); // file.txt
  $str->stem('/path/to/file.txt');     // file
  $str->ext('/path/to/file.txt');      // txt
  $str->dir('/path/to/file.txt');      // /path/to
  ```

---

## Character Filters

- **alphanumeric() / alpha() / number()**: Keep only specific character types.
  ```php
  $str->alphanumeric('Hello, World! 123'); // HelloWorld123
  $str->alpha('Hello123');                 // Hello
  $str->number('Price: $123.45');          // 12345
  ```
- **collapse()**: Replace multiple whitespace with a single space.
  ```php
  $str->collapse('Hello   World'); // Hello World
  ```
- **initials()**: Generate initials from a name.
  ```php
  $str->initials('John Doe'); // JD
  ```

---

## One-Time Password (OTP)

- **otp()**: Generate a numeric OTP of specified length (default: 6).
  ```php
  $str->otp();    // e.g. '123456'
  $str->otp(8);   // e.g. '12345678'
  ```
  - Throws `InvalidArgumentException` if length < 1.

> For more complex OTP generation, Lightpack provides a robust OTP utility
> [OTP Utility](/otp-utils.md)

---