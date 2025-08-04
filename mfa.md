# Lightpack MFA: Complete Developer Guide

> **Lightpack MFA** enables robust, flexible, and secure multi-factor authentication for modern PHP applications. It supports TOTP (authenticator app), SMS, Email, and backup codes out of the box, and is designed for extensibility, security, and ease of use.

## Supported Factors

- **TOTP (Authenticator App):**  
  - Time-based one-time passwords (Google Authenticator, Authy, etc.)
- **SMS:**  
  - One-time code sent to user’s phone (uses Lightpack SMS subsystem)
- **Email:**  
  - One-time code sent to user’s email
- **Backup Codes:**  
  - Single-use codes for account recovery
- **Null:**  
  - No MFA (for testing or fallback)

## Installation

### 1. Migration

Your **users** table already contains required fields to support **MFA** features. You do not need a seperate migration to be generated and run.

### 2. Install Dependencies

- For TOTP:  
  `composer require robthree/twofactorauth`
- For SMS:  
  See [Lightpack SMS](sms.md) documentation (Twilio, etc.)

---

### 3. Configure MFA

View `config/mfa.php` for available configuration options.

## User Model Integration

Add the trait to your User model:

```php
use Lightpack\Mfa\MfaTrait;

class User extends Model {
    use MfaTrait;
    // ...
}
```

This extends your User model capabilities with methods:

- `getMfaFactor()`
- `sendMfa()`
- `validateMfa($input)`

---

## Usage Patterns

### Enrollment

1. **TOTP:**  
   - Generate secret: `TotpSetupHelper::generateSecret()`
   - Show QR: `TotpSetupHelper::getQrUri($secret, $user->email)`
   - Save secret to user: `$user->mfa_totp_secret = $secret; $user->save();`
   - User scans QR with authenticator app.

2. **SMS/Email:**  
   - Ensure user’s phone/email is set.
   - Set `$user->mfa_method = 'sms'` or `'email'`, `$user->save();`

3. **Backup Codes:**  

Generate and show codes to user (store only hashes!)

```php
$codes = BackupCodeHelper::generateCodes(); // array of codes
$hashes = BackupCodeHelper::hashCodes($codes);
$user->mfa_backup_codes = json_encode($hashes);
$user->save();
```

### Verification

- To send challenge:  
  `$user->sendMfa(); // sends code via chosen factor`
- To validate:  
  `$user->validateMfa($input); // returns true/false`

### Backup Codes

- On form input, use:  

```php
$hashes = json_decode($user->mfa_backup_codes, true);

[$ok, $remaining] = BackupCodeHelper::verifyAndRemoveCode($hashes, $input);

if ($ok) {
    $user->mfa_backup_codes = json_encode($remaining);
    $user->save();
}
```

### TOTP (Authenticator App)

- Use `TotpSetupHelper` for QR code and secret generation.
- Validation is handled by TOTP factor automatically.

### SMS & Email

- SMS uses the configured SMS provider (see Lightpack SMS docs).
- Email uses the built-in mail system.

---

## Example using TOTP


### 1. Enrolling TOTP

```php
$secret = TotpSetupHelper::generateSecret();
$user->mfa_totp_secret = $secret;
$user->mfa_method = 'totp';
$user->save();

$qr = TotpSetupHelper::getQrUri($secret, $user->email);
// Show $qr in frontend for scanning
```

### 2. Sending and Validating MFA

```php
$user->sendMfa(); // Sends challenge via chosen factor

if ($user->validateMfa($inputCode)) {
    // MFA successful
} else {
    // MFA failed
}
```

### 3. Generating and Using Backup Codes

```php
$codes = BackupCodeHelper::generateCodes();
$hashes = BackupCodeHelper::hashCodes($codes);
$user->mfa_backup_codes = json_encode($hashes);
$user->save();
// Show $codes to user (never show hashes)
```

## Extending with Custom Factors

Create a new factor:

```php
namespace App\Mfa\Factor;
use Lightpack\Mfa\MfaInterface;
use Lightpack\Auth\Models\AuthUser;

class MyPushMfa implements MfaInterface {
    public function send(AuthUser $user): void {
        // Send push notification
    }
    public function validate(AuthUser $user, ?string $input): bool {
        // Validate push approval
        return true;
    }
    public function getName(): string {
        return 'push';
    }
}
```

Register in config:

```php
'factors' => [
    'push' => App\Mfa\Factor\MyPushMfa::class,
    // ...other factors
]
```

---

## Error Handling & Security

- **All secrets/codes are hashed or encrypted.**
- **Backup codes are one-time use.**
- **TOTP uses secure, random secrets.**
- **SMS/email codes are time-limited (implement expiry in your app logic).**
- **Always check return value of `validateMfa()`.**
- **Never log or expose secrets/codes.**

---

## Best Practices & Tips

- Always offer backup codes for account recovery.
- Encourage TOTP as the most secure default.
- Use SMS/email as fallback, not primary, if possible.
- Rotate secrets/codes after use or on user request.
- Log failed attempts for monitoring.

---

## FAQ & Troubleshooting

**Q: How do I reset a user's MFA?**  
A: Clear all MFA fields on the user model.

**Q: Can I require MFA for only some users?**  
A: Use the `mfa_enabled` boolean field.

**Q: How do I support multiple factors per user?**  
A: Extend your user model and UI view to allow selection/switching.

**Q: How do I test MFA?**  
A: Use the 'null' factor or set predictable secrets/codes in dev.

---