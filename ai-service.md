# Lightpack AI System

A unified, explicit, and extensible interface for text generation, summarization, and structured AI tasks in your Lightpack apps. Supports multiple providers, robust schema validation, and a fluent builder for advanced use cases.

- **Purpose:** Seamlessly add AI/ML-powered text, content, and structured data generation to any Lightpack project.
- **Where to Use:** Blog/content generation, summarization, Q&A, code generation, structured data extraction, creative writing, and more.

**Lightpack AI** exposes three methods:

```php
ai()->ask();      // Simple question-answer
ai()->task();     // Structured data extraction
ai()->generate(); // Raw API access
```

## Supported Providers

| Driver      | Class                        | Use Case / Notes         |
|-------------|------------------------------|--------------------------|
| `openai`    | `Providers\OpenAI`           | OpenAI GPT-3.5, GPT-4    |
| `anthropic` | `Providers\Anthropic`        | Claude models            |
| `mistral`   | `Providers\Mistral`          | Mistral models           |
| `groq`      | `Providers\Groq`             | Groq API                 |
| `gemini`    | `Providers\Gemini`           | Gemini API               |

**Add your own:** Implement `ProviderInterface` and register in config.

## Configuration

Please run following command to create `config/ai.php` configuration file.

```cli
php console create:config --support=ai
```

Below is a brief explanation of **config/ai.php** file.

| Key                          | Description                              | Example / Default         |
|------------------------------|------------------------------------------|---------------------------|
| `driver`                     | Which provider to use                    | `'openai'`                |
| `key`                        | API key for the provider                 | (secret, per provider)    |
| `temperature`                | Controls creativity (0.0–2.0)            | `0.7`                     |
| `max_tokens`                 | Max output length                        | `256`                     |
| `http_timeout`               | Request timeout (seconds)                | `10`                      |
| `model`                      | Model name (per provider)                | `'gpt-3.5-turbo'`, etc.   |
| `cache_ttl`                  | Cache TTL when enabled (seconds)         | `3600`                    |
| `endpoint`                   | Custom API endpoint (optional)           | Per-provider default      |
| `providers.<driver>.endpoint`| API endpoint (per provider)              |                           |
| `providers.<driver>.model`   | Default model for driver                 |                           |
| `providers.<driver>.key`     | API key for driver                       |                           |

## Usage

Lightpack AI provides three methods for different use cases:

| Method | Use When | Returns |
|--------|----------|---------|
| `ask()` | Simple questions, plain text answers | String |
| `task()` | Structured data extraction with validation | Array with `success`, `data`, `errors` |
| `generate()` | Custom endpoints, non-JSON responses, raw API access | Array with `text`, `usage`, `raw` |

**Quick Decision Guide:**
- Need a quick answer? → `ask()`
- Need JSON with specific fields? → `task()`
- Need embeddings/images or custom API? → `generate()`

---

### ask()

**Use for:** Quick questions that need plain text answers.

For simple, one-off questions, use the `ask()` method:

```php
$answer = ai()->ask('What is the capital of France?');
echo $answer; // "Paris"
```
- Internally uses the TaskBuilder for prompt handling.
- Returns the raw answer as a plain string.

---

### task()

**Use for:** Extracting structured data with type validation and required field checks.

```php
$result = ai()->task()
    ->prompt('Who created Monalisa and at what age?')
    ->expect(['name' => 'string', 'age' => 'int'])
    ->required('name', 'age')
    ->run();

if ($result['success']) {
    echo $result['data']['name']; // "Leonardo da Vinci"
    echo $result['data']['age'];  // 51
} else {
    print_r($result['errors']);   // ["Missing required field: name"]
}
```

**Key methods:**
- `prompt(string)` - Set the question
- `expect(array)` - Define JSON schema with types
- `required(...fields)` - Mark fields as required
- `expectArray(key)` - Expect array of objects
- `system(string)` - Set system prompt
- `model(string)` - Override model
- `cache(bool)` - Enable caching
- `run()` - Execute and return `['success', 'data', 'errors', 'raw']`

---

#### Example Recipes

**1. Validate Array of Objects**

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

**2. Use Conversation History**

```php
$result = ai()->task()
    ->message('system', 'You are a helpful assistant.')
    ->message('user', 'How do I reset my password?')
    ->run();
```

### generate()

**Use for:** Raw API access, custom endpoints (embeddings, images), or non-JSON responses.

```php
// Standard chat completion
$result = ai()->generate([
    'prompt' => 'Explain quantum computing',
    'model' => 'gpt-4',
    'temperature' => 0.7,
    'max_tokens' => 500,
]);

echo $result['text'];      // Generated text
echo $result['usage'];     // Token usage stats (if available)
```

**Supported parameters:**
- `prompt` or `messages`: Input text or conversation history
- `system`: System prompt/persona
- `model`: Model name (overrides config)
- `temperature`: Creativity level (0.0-2.0)
- `max_tokens`: Max output length
- `cache`: Enable caching (default: false)
- `cache_ttl`: Cache TTL in seconds
- `endpoint`: Custom API endpoint
- `timeout`: HTTP timeout in seconds

---

#### Example Recipes

**1. Custom Model/Temperature**

```php
$result = ai()->generate([
    'prompt' => 'Write a creative poem.',
    'model' => 'gpt-4',
    'temperature' => 1.2,
]);
```

**2. Custom Endpoint (Multiple APIs)**

```php
// Use embeddings API instead of chat
$result = ai()->generate([
    'endpoint' => 'https://api.openai.com/v1/embeddings',
    'input' => 'Text to embed',
    'model' => 'text-embedding-ada-002'
]);

// Use image generation API
$result = ai()->generate([
    'endpoint' => 'https://api.openai.com/v1/images/generations',
    'prompt' => 'A sunset over mountains',
    'model' => 'dall-e-3'
]);
```

## Caching

- **Caching is opt-in** (disabled by default) to preserve AI response variability.
- Enable caching for deterministic tasks (data extraction, classification) or cost optimization:
  ```php
ai()->task()
    ->prompt('Extract email from: john@example.com')
    ->cache(true)     // enables cache for this run
    ->cacheTtl(3600)  // cache for 1 hour (optional, uses config default)
    ->run();
  ```
- **When to cache:** Deterministic tasks with `temperature: 0`, expensive structured data extraction, repeated queries.
- **When NOT to cache:** Creative writing, real-time data, personalized responses.

## Error Handling

- Transport/API call related errors throw exceptions.
- Validation errors (missing required fields, schema mismatch) are in `$result['errors']`.
- Always check `$result['success']` when using `TaskBuilder`.

## Security & Best Practices

- **Never commit API keys to version control.**
- **Limit max_tokens and temperature for cost and predictability.**
- **Log and monitor AI errors for production apps.**
- **Use schema/required fields for structured data extraction.**

---