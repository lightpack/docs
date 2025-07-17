# FormRequest: Elegant HTTP Form Validation for Lightpack

The `FormRequest` class in Lightpack provides a powerful, expressive, and reusable way to handle HTTP request validation and authorization in your applications. It encapsulates validation logic, error handling, and request data preparation, making your controllers clean and focused.

---

## Key Features

- **Declarative Validation:** Define your rules in a dedicated method, not in the controller.
- **Automatic Validation:** Requests are validated before reaching your controller logic.
- **AJAX & JSON Support:** Automatically returns JSON error responses for AJAX/JSON requests.
- **Custom Hooks:** Easily customize data preparation and error handling with overridable methods.
- **Seamless Integration:** Works with Lightpack’s DI container, session, and redirect systems.

---

## How It Works

1. **Extend FormRequest:** Create your own request classes by extending `Lightpack\Http\FormRequest`.
2. **Define Rules:** Implement the `rules()` method to return your validation rules.
   - **Rule Resolution:** Your `rules()` method is called via the container.
   - You can typehint dependencies you would like to get injected by the framework.
3. **Automatic Bootstrapping:** Lightpack boots your FormRequest, runs validation, and handles errors or passes control to your controller.
4. On successful validation, controller method executes further.
5. **On Failure:**
  - **AJAX/JSON:** Responds with HTTP 422 and a JSON error structure.
  - **Standard Request:** Redirects back with errors in the session.
  - Custom hooks (`beforeSend`, `beforeRedirect`) are available for advanced control.

## Example Usage

### 1. Create a FormRequest

```php
php console create:request RegisterUserRequest
```

Then implement the `rules()` method. For example:

```php
namespace App\Requests;

use Lightpack\Http\FormRequest;

class RegisterUserRequest extends FormRequest
{
    protected function rules()
    {
         $this->validator
            ->field('name')
            ->required()
            ->max(255);

         $this->validator
            ->field('email')
            ->required()
            ->email()
            ->custom(new UniqueEmailRule, UniqueEmailRule::MESSAGE);

         $this->validator
            ->field('password')
            ->required()
            ->min(6)
            ->max(25);

         $this->validator
            ->field('confirm_password')
            ->required()
            ->same('password');
    }
}
```

### 2. Use in Controller

Typehint the request class as dependency in your controller's method:

```php
public function register(RegisterUserRequest $request)
{
   // If validation passes, you reach here!

    $data = $request->all();

   // Proceed with user registration...
}
```

## Overridable Hooks

You can customize the request lifecycle by overriding these methods:

- `protected function data()`: Prepare or mutate request data before validation.
- `protected function beforeSend()`: Run logic before sending a JSON error response.
- `protected function beforeRedirect()`: Run logic before redirecting on validation failure.

---

## Error Handling

- **ValidationException:** Thrown automatically on validation failure for standard requests.
- **Session Flash:** Errors and input are flashed to the session for easy display in views.
- **Error Structure (AJAX):**
  ```json
  {
    "success": false,
    "message": "Request validation failed",
    "errors": {
      "email": ["The email field is required."],
      "password": ["The password must be at least 8 characters."]
    }
  }
  ```

---

## Best Practices

- **Thin Controllers:** Move all validation logic to FormRequest classes.
- **Reusable Rules:** Centralize and reuse request validation across controllers.
- **Custom Data Preparation:** Use the `data()` method to preprocess or sanitize input.
- **AJAX-Friendly:** No extra work needed for API endpoints—JSON errors are automatic.

---

## Advanced: Customizing Validation

Override any of the following in your FormRequest:

```php
protected function data()
{
   // Manipulate request input data before validation
}

protected function beforeSend()
{
   // Add custom headers or logging before JSON error response
}

protected function beforeRedirect()
{
   // Custom logic before redirecting on failure
}
```

---