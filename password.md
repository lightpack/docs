# Password Hashing

When providing user registration options, we need to store a user's password in the database. As a good practice, we should never store the password as a plain text as supplied by the user.

Instead, we should make a `one-way` hash of the password and then store the hashed password in the database.

**Lightpack** provides an easy to use password hashing and hash verification library.

```php
class Lightpack\Crypto\Password {}
```

You can access an instance of this class anywhere in your application using `password()` function.

## hash()

Use this method to get the hash of the user supplied password text. It takes the plain text password string as argument and return the hash.

```php
$hash = password()->hash('secret');
```

## verify()

Use this method to verify if the user supplied password matches the existing hashed password.

Suppose `$hash` is the hashed password stored in database. To match if the stored password hash matches the user supplied password:

```php
if(password()->verify('secret', $hash)) {
    // correct password
}
```