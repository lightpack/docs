# Crypto

Lightpack’s `Crypto` utility provides secure, easy-to-use methods for encryption, decryption, token generation, and hashing. It is designed for safe handling of sensitive data using modern cryptography standards.

---

## Overview
- **Encryption/Decryption:** Securely encrypt and decrypt strings using AES-256-CBC and your app key.
- **Token Generation:** Generate unique, cryptographically secure tokens.
- **Hashing:** Generate HMAC-SHA256 hashes for data integrity or signatures.
- **Provider/Container:** Available via dependency injection and the global `crypto()` helper.

---

## Configuration & Usage

### App Key Requirement
Your `.env` file **must** have an `APP_KEY` set. The `CryptoProvider` will throw if missing.

```
APP_KEY=your-very-secret-key
```

### Accessing the Utility
```php
use Lightpack\Utils\Crypto;

$crypto = new Crypto($key); // Manual instantiation
$crypto = crypto();         // Preferred: via container helper
```

---

## Encrypting & Decrypting Data

```php
$encrypted = crypto()->encrypt('Sensitive data');
$decrypted = crypto()->decrypt($encrypted); // 'Sensitive data'
```
- Uses AES-256-CBC with a random IV per encryption.
- Output is base64-encoded; safe for storage and transport.
- Decryption returns the original string or `false` on failure.
- Each encryption produces a unique output (random IV).

---

## Generating Tokens

```php
$token = crypto()->token(); // 64-char secure token
```
- Useful for CSRF, password resets, API keys, etc.
- Each token is unique and cryptographically secure.

---

## Hashing Data

```php
$hash = crypto()->hash('my data');
```
- Returns a SHA-256 HMAC hash using your app key.
- Use for signatures, integrity checks, or non-reversible data storage.

---

## Helper Function

You can use the global `crypto()` helper anywhere in your app:
```php
crypto()->encrypt('secret');
crypto()->decrypt($encrypted);
crypto()->token();
crypto()->hash('data');
```

---

## Error Handling & Security
- If `APP_KEY` is not set, an exception is thrown at service registration.
- All cryptographic operations use secure random bytes and modern algorithms.
- Never share your app key or use weak keys.
- Ensure you backup your secret key.

---