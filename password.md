# Password Utilities

Lightpack provides a robust utility for password hashing, verification, random password generation, and strength checking. This helps you securely store user passwords and enforce good password practices in your application.

## Overview

- **Hashing:** Securely hash user passwords for storage (never store plain text passwords).
- **Verification:** Check if a user-supplied password matches a stored hash.
- **Random Generation:** Generate strong random passwords for users or system use.
- **Strength Checking:** Assess password strength according to best practices.

You can get an instance of the utility via:

```php
use Lightpack\Utils\Password;

$password = new Password();
```

Or use the `password()` utility function.

```php
$hash = password()->hash('secret');
```

---

## Hashing Passwords

Hash a plain-text password for storage:

```php
$hash = $password->hash('secret');
// Store $hash in your database
```

- Uses PHP's `password_hash()` with the best available algorithm.
- Output is a one-way, cryptographically secure hash.

> **Tip:** Never store or log plain-text passwords.

---

## Verifying Passwords

Check if a user-supplied password matches a stored hash:

```php
if ($password->verify('user-input', $hashFromDb)) {
    // Password is correct
} else {
    // Incorrect password
}
```

- Uses `password_verify()` under the hood.
- Safe against timing attacks.

---

## Generating Random Passwords

Generate a random password string of a given length (default: 8, minimum: 6):

```php
$random = $password->generate();      // 8 characters
$random16 = $password->generate(16);  // 16 characters
```

- Always includes at least one uppercase, one lowercase, one number, and one special character.
- Throws an exception if length is less than 6.

> **Tip:** Use this for "reset password" flows or to suggest strong passwords to users.

---

## Checking Password Strength

Assess the strength of a password:

```php
$strength = $password->strength('A123#abc'); // 'strong', 'medium', or 'weak'
```

**Rules:**
1. At least 8 characters
2. At least one uppercase letter
3. At least one lowercase letter
4. At least one number
5. At least one special character

**Returns:**
- `'weak'` (meets 1-2 rules)
- `'medium'` (meets 3-4 rules)
- `'strong'` (meets all 5 rules)

> **Tip:** Use this to enforce password policies or provide feedback to users.

---

## Edge Cases & Notes

- `generate()` will always return a password meeting all strength rules (if length >= 8).
- If you pass a length < 6 to `generate()`, an exception is thrown.
- Hashing and verification use PHP's built-in password functions for maximum security.
- All features are covered by comprehensive tests.

---