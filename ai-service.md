# Lightpack AI System

A unified, explicit, and extensible interface for text generation, summarization, and structured AI tasks in your Lightpack apps. Supports multiple providers, robust schema validation, and a fluent builder for advanced use cases.

---

## What is Lightpack AI?

- **Purpose:** Seamlessly add AI/ML-powered text, content, and structured data generation to any Lightpack project.
- **Where to Use:** Blog/content generation, summarization, Q&A, code generation, structured data extraction, creative writing, and more.

---

## Supported Providers

| Driver      | Class                        | Use Case / Notes         |
|-------------|------------------------------|--------------------------|
| `openai`    | `Providers\OpenAI`           | OpenAI GPT-3.5, GPT-4    |
| `anthropic` | `Providers\Anthropic`        | Claude models            |
| `mistral`   | `Providers\Mistral`          | Mistral models           |
| `groq`      | `Providers\Groq`             | Groq API                 |

**Add your own:** Implement `ProviderInterface` and register in config.

---


## Configuration

All settings live in `config/ai.php`:

| Key                          | Description                              | Example / Default         |
|------------------------------|------------------------------------------|---------------------------|
| `driver`                     | Which provider to use                    | `'openai'`                |
| `key`                        | API key for the provider                 | (secret, per provider)    |
| `temperature`                | Controls creativity (0.0–2.0)            | `0.7`                     |
| `max_tokens`                 | Max output length                        | `256`                     |
| `http_timeout`               | Request timeout (seconds)                | `10`                      |
| `model`                      | Model name (per provider)                | `'gpt-3.5-turbo'`, etc.   |
| `cache_ttl`                  | Cache lifetime (seconds)                 | `3600`                    |
| `providers.<driver>.endpoint`| API endpoint (per provider)              |                           |
| `providers.<driver>.model`   | Default model for driver                 |                           |
| `providers.<driver>.key`     | API key for driver                       |                           |

---

## Service Registration

- The `AiProvider` registers the `ai` service with the container.
- The active driver is set via `config/ai.php` in your config.
- Use `ai()` helper as convinience to access the current provider.

---


## ask()

For simple, one-off questions, use the `ask()` method:

```php
$answer = ai()->ask('What is the capital of France?');
echo $answer; // "Paris"
```
- Internally uses the TaskBuilder for prompt handling.
- Returns the raw answer as a plain string.

---

## generate()

```php
// Generate text using the configured provider
$result = ai()->generate([
    'prompt' => 'Suggest 5 catchy blog titles for ecommerce.',
]);

echo $result['text'];
```

- Returns an array: `['text' => ..., 'finish_reason' => ..., 'usage' => ..., 'raw' => ...]`
- All parameters can be overridden per call.

---

## task()

The `TaskBuilder` enables advanced, schema-aware structured AI response:

```php
// Get a structured JSON object with validation
$result = ai()->task()
    ->prompt('What is your name and age?')
    ->expect(['name' => 'string', 'age' => 'int'])
    ->required('name', 'age')
    ->run();

if ($result['success']) {
    echo $result['data']['name']; // "Alice"
    echo $result['data']['age'];  // 30
} else {
    print_r($result['errors']);
}
```

**It exposes following capabilities:**

- **expect(array $schema):** Specify expected keys/types for JSON output.
- **required(...$fields):** Mark fields as required (fail if missing/null).
- **expectArray($key):** Expect an array of objects (e.g., list of movies).
- **example(array $example):** Provide an example for the model.
- **messages/system:** Compose multi-turn, role-based conversations.
- **Robust JSON extraction:** Handles messy LLM output.

**The returned response contains:**

```php
[
    'text' => 'The generated content.',
    'finish_reason' => 'stop', // or other reason, if available
    'usage' => [...],          // token stats, if available
    'raw' => [...],            // full provider response
]
```

---

### Example Recipes

#### 1. Validate Array of Objects

```php
$result = ai()->task()
    ->prompt('List 2 movies with title, rating, and summary.')
    ->expect(['title' => 'string', 'rating' => 'int', 'summary' => 'string'])
    ->required('title', 'rating', 'summary')
    ->expectArray('movie')
    ->run();

if (!$result['success']) {
    // $result['errors'] contains missing fields per item
}
```

#### 2. Use Conversation History

```php
$result = ai()->task()
    ->message('system', 'You are a helpful assistant.')
    ->message('user', 'How do I reset my password?')
    ->run();
```

#### 3. Custom Model/Temperature

```php
$result = ai()->generate([
    'prompt' => 'Write a creative poem.',
    'model' => 'gpt-4',
    'temperature' => 1.2,
]);
```

#### 4. Caching

- All providers cache results by default (configurable via `cache` and `cache_ttl` keys).
- To bypass cache for direct calls: `ai()->generate(['cache' => false ])`
- **Note:** When using the TaskBuilder (`ai()->task()->...->run()`), caching uses the provider’s default settings and cannot be overridden per request via the builder.

---

## Error Handling

- Transport/API call related errors throw exceptions.
- Validation errors (missing required fields, schema mismatch) are in `$result['errors']`.
- Always check `$result['success']` when using `TaskBuilder`.

---

## Security & Best Practices

- **Never commit API keys to version control.**
- **Limit max_tokens and temperature for cost and predictability.**
- **Log and monitor AI errors for production apps.**
- **Use schema/required fields for structured data extraction.**

---