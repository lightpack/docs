# Lightpack AI System

A unified, explicit, and extensible interface for text generation, summarization, structured AI tasks, and semantic search in your Lightpack apps. Supports multiple providers, robust schema validation, and a fluent builder for advanced use cases.

- **Purpose:** Seamlessly add AI/ML-powered text generation, embeddings, and semantic search to any Lightpack project.
- **Where to Use:** Blog/content generation, summarization, Q&A, code generation, structured data extraction, semantic search, RAG applications, content recommendations, and more.

**Lightpack AI** exposes four methods:

```php
ai()->ask();      // Simple question-answer
ai()->task();     // Structured data extraction
ai()->embed();    // Text to vector embeddings
ai()->similar();  // Semantic similarity search
```

## Supported Providers

| Driver      | Class                        | Text Generation | Embeddings |
|-------------|------------------------------|-----------------|------------|
| `openai`    | `Providers\OpenAI`           | ✅ GPT-3.5, GPT-4 | ✅ text-embedding-3-small |
| `gemini`    | `Providers\Gemini`           | ✅ Gemini models  | ✅ text-embedding-004 (FREE) |
| `mistral`   | `Providers\Mistral`          | ✅ Mistral models | ✅ mistral-embed |
| `anthropic` | `Providers\Anthropic`        | ✅ Claude models  | ❌ Not supported |
| `groq`      | `Providers\Groq`             | ✅ Llama, Mixtral | ❌ Not supported |

**Add your own:** Implement `ProviderInterface` and register in config.

## Configuration

Please run following command to create `config/ai.php` configuration file.

```cli
php console create:config --support=ai
```


## Usage

Lightpack AI provides four methods for different use cases:

| Method | Use When | Returns |
|--------|----------|---------|
| `ask()` | Simple questions, plain text answers | String |
| `task()` | Structured data extraction with validation | Array with `success`, `data`, `errors` |
| `embed()` | Convert text to vector embeddings | Array of floats (single) or array of arrays (batch) |
| `similar()` | Find semantically similar items | Array of matches with similarity scores |

**Quick Decision Guide:**
- Need a quick answer? → `ask()`
- Need JSON with specific fields? → `task()`
- Need semantic search? → `embed()` + `similar()`

---

### ask()

**Use for:** Quick questions that need plain text answers.

For simple, one-off questions, use the `ask()` method:

```php
$answer = ai()->ask('What is the capital of France?');
echo $answer; // "Paris"
```

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

---

### embed()

**Use for:** Converting text into vector embeddings for semantic search, similarity matching, or RAG applications.

```php
// Single text
$embedding = ai()->embed('wireless headphones');
// Returns: [0.123, -0.456, 0.789, ...] (768-1536 floats)

// Batch processing (efficient - single API call)
$texts = [
    'Product A description',
    'Product B description',
    'Product C description'
];
$embeddings = ai()->embed($texts);

// Use batch results - CRITICAL: maintains same order as input!
foreach ($texts as $i => $text) {
    echo "Text: {$text}\n";
    echo "Embedding: " . json_encode($embeddings[$i]) . "\n";
}
```

**Method signature:**
```php
ai()->embed(
    string|array $input,  // Required: Single text or array of texts
    array $options = []   // Optional: Provider-specific options
): array
```

**Options parameter:**
```php
$options = [
    'model' => 'text-embedding-3-small',  // Override default model
    // Provider-specific options vary
];
```

**Returns:**
- **Single input:** `array` of floats (e.g., 768 or 1536 dimensions)
- **Batch input:** `array` of arrays (one embedding per input text, **same order as input**)

**Key points:**
- Returns array of floats (vector representation of text)
- Batch processing uses single API call (cost-efficient)
- Batch results maintain same order as input array
- Store embeddings in database for reuse
- Dimensions vary by provider (768 for Gemini, 1536 for OpenAI)
- Not all providers support embeddings (see provider table above)
- Options parameter allows model override and provider-specific settings

---

### similar()

**Use for:** Finding semantically similar items.

```php
$queryEmbedding = ai()->embed('laptop for programming');

$results = ai()->similar($queryEmbedding, $products);

foreach ($results as $result) {
    echo $result['id'];         // Product ID
    echo $result['similarity']; // 0.0-1.0 score
    echo $result['item'];       // Original item data
}
```

**Method signature:**
```php
ai()->similar(
    array $queryEmbedding,  // Required: Query vector
    mixed $target,          // Required: Array of items (in-memory) or collection name (vector DB)
    int $limit = 5,         // Optional: Max results (default: 5)
    float $threshold = 0.0  // Optional: Min similarity score (default: 0.0)
): array
```

**Returns:**
```php
[
    [
        'id' => mixed,           // Item identifier
        'similarity' => float,   // Score 0.0-1.0
        'item' => array          // Original item data
    ],
    // ... more results
]
```

**Key points:**
- Returns exact matches (100% recall, not approximate)
- Works in-memory by default (fast for < 5K items)
- Scores range from 0.0 (different) to 1.0 (identical)
- Results sorted by similarity (highest first)
- **Threshold defaults to 0.0** (returns all results) - set to 0.6-0.8 for quality filtering
- Extensible via `setVectorSearch()` for custom implementations

**Threshold recommendations:**
- `0.0` (default) - Return all results, let user decide
- `0.6-0.7` - Moderate similarity (related items)
- `0.8-0.9` - High similarity (very similar items)
- `0.95+` - Near-identical items

---

#### Example Recipes 1: Storing Embeddings

**1. Store Product Embeddings**

```php
// In your model
class Product extends Model
{
    protected $casts = [
        'embedding' => 'array'
    ];
}

// store the embeddings for the product
$product->embedding = ai()->embed($product->description);
$product->save();
```

**2. Semantic Product Search**

```php
// fetch products with their embeddings
$items = Product::query()
    ->select('id', 'embedding')
    ->whereNotNull('embedding')
    ->all()
    ->map(fn($p) => [
        'id' => $p->id,
        'embedding' => $p->embedding
    ]);

// semantic search for similar products
$query = 'best laptop for programming';
$queryEmbedding = ai()->embed($query);
$results = ai()->similar($queryEmbedding, $items, limit: 10);

// process results
foreach ($results as $result) {
    // Product ID: $result['id'];
}
```

---

#### Example Recipes 2: Filtering and RAG

**1. Filter by Similarity Threshold**

```php
// Only return matches with 70%+ similarity
$results = ai()->similar($queryEmbedding, $items, limit: 10, threshold: 0.7);
```

**2. RAG (Retrieval Augmented Generation)**

```php
// Find relevant docs
$userQuestion = 'How do I reset my password?';
$queryEmbedding = ai()->embed($userQuestion);
$relevant = ai()->similar($queryEmbedding, $docs, limit: 3);

// Build context
$context = implode("\n\n", array_column($relevant, 'item'));

// Ask AI with context
$answer = ai()->task()
    ->system("Answer based on this documentation:\n\n{$context}")
    ->prompt($userQuestion)
    ->run();
```

**3. Content Recommendations**

```php
$articles = Article::query()
    ->select('id', 'embedding')
    ->whereNotNull('embedding')
    ->all()
    ->map(fn($a) => [
        'id' => $a->id,
        'embedding' => $a->embedding
    ]);

// "Users who read this also read..."
$similar = ai()->similar($article->embedding, $articles, limit: 5);
```

---

### Embedding Provider Notes

**Important:** Embeddings are NOT cross-compatible. Always use the same provider for embedding and searching.

---

## Vector Search: Architecture & Extensibility

When you call `ai()->similar()`, Lightpack uses a `VectorSearchInterface` implementation to find matches. By default, this is `InMemoryVectorSearch`, but you can swap it for Qdrant, Meilisearch, or any custom implementation. Lightpack's vector search is designed with a simple principle: **start simple, scale when needed**. The default in-memory implementation is good for most of real-world applications, but you can seamlessly upgrade to vector databases when you outgrow it.

```php
// Default behavior - uses InMemoryVectorSearch automatically
$results = ai()->similar($queryEmbedding, $items);

// Custom implementation - swap to vector database
ai()->setVectorSearch(new QdrantVectorSearch());
$results = ai()->similar($queryEmbedding, 'products_collection');
```

---

### Default: InMemoryVectorSearch

**What it is:** A brute-force cosine similarity search that compares your query against every item in memory.

| Aspect | Details |
|--------|---------|
| **Algorithm** | Brute-force cosine similarity (O(n)) |
| **Accuracy** | 100% recall (exact, not approximate) |
| **Performance** | ~20-250ms for < 5K items |
| **Memory** | ~3 KB per item (embeddings only) |
| **Scale** | < 5K documents, < 50 concurrent searches/sec |

**Note:** Above is not a hard benchmark but a good approximation.

**Example:**
```php
// Load only embeddings (not full models!)
$items = Product::query()
    ->select('id', 'embedding')
    ->whereNotNull('embedding')
    ->all()
    ->map(fn($p) => [
        'id' => $p->id,
        'embedding' => $p->embedding
    ]);

// Search for similar items
$results = ai()->similar($queryEmbedding, $items, limit: 10);
```

---

### Extending with Vector Databases

**Extend with vector databases for larger scale:**

```php
use Lightpack\AI\VectorSearch\VectorSearchInterface;

class QdrantVectorSearch implements VectorSearchInterface
{
    public function __construct(private $client) {}

    public function search(array $queryEmbedding, mixed $target, int $limit = 5, array $options = []): array
    {
        // $target is collection name for vector DBs
        $response = $this->client->search($target, [
            'vector' => $queryEmbedding,
            'limit' => $limit,
            'score_threshold' => $options['threshold'] ?? 0.0
        ]);
        
        return $this->formatResults($response);
    }
    
    private function formatResults($response): array
    {
        // Must return same format as InMemoryVectorSearch
        return array_map(fn($hit) => [
            'id' => $hit['id'],
            'similarity' => $hit['score'],
            'item' => $hit['payload']
        ], $response['result']);
    }
}

// Use custom implementation
$vectorSearch = app(QdrantVectorSearch::class);
ai()->setVectorSearch($vectorSearch);

// Now similar() uses Qdrant
$results = ai()->similar($queryEmbedding, 'products_collection', limit: 10);
```

**Interface contract:**
```php
interface VectorSearchInterface
{
    /**
     * @param array $queryEmbedding The query vector
     * @param mixed $target For in-memory: array of items. For vector DBs: collection name
     * @param int $limit Max results to return
     * @param array $options Implementation-specific options (threshold, filters, etc.)
     * @return array Array of results with 'id', 'similarity', and 'item' keys
     */
    public function search(array $queryEmbedding, mixed $target, int $limit = 5, array $options = []): array;
}
```

**Return format (must match):**
```php
[
    [
        'id' => mixed,           // Required: Item identifier
        'similarity' => float,   // Required: Score 0.0-1.0
        'item' => array          // Required: Item data
    ],
    // ...
]
```

---

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