# URL Utility

Lightpack's **Url** utility class provides a comprehensive set of methods for URL generation, manipulation, and validation. It supports route-based URLs, asset URLs, signed URLs, and various URL manipulation operations.

You can instantiate to **Url** utility:

```php
$url = new Lightpack\Utils\Url();
```

Or you can call the utility function `url()` which returns an instance or **Url** class.

## Basic URL Generation

### Simple URLs

```php
// Generate basic URL
url()->to('users'); // Returns: /users

// URL with multiple segments
url()->to('blog', 'posts', '123'); // Returns: /blog/posts/123

// URL with query parameters
// Returns: /users?sort=asc&status=active
url()->to('users', ['sort' => 'asc', 'status' => 'active']); 
```

If you set the environment variable `APP_URL`, then it returns fully qualified URL:

```php
// If APP_URL=https://example.com
// Returns: https://example.com/api/v1
url()->to('api', 'v1'); 
```

## Asset URLs

Generate URLs for static assets in the public directory:

```php
// Basic asset URL
url()->asset('css/styles.css'); // Returns: /css/styles.css
```

With `ASSET_URL` environment variable:

```php
// If ASSET_URL=https://cdn.example.com
// Returns: https://cdn.example.com/js/app.js
url()->asset('js/app.js'); 
```

Fallback to `APP_URL` if `ASSET_URL` not set:

```php
// If APP_URL=https://example.com
// Returns: https://example.com/images/logo.png
url()->asset('images/logo.png'); 
```

## Route URLs

Generate URLs for named routes:

```php

Basic route URL

```php
// Returns: /users/123/profile
url()->route('user.profile', ['id' => 123]); 
```
Route with optional parameters

```php
// Returns: /blog/tech/php-tips
url()->route('blog.post', [
    'category' => 'tech',
    'slug' => 'php-tips'
]); 
```

Route with query parameters:

```php
// Returns: /search?q=php&page=1
url()->route('search', [
    'q' => 'php',
    'page' => 1
]); 
```

## URL Manipulation

### Query Parameters

Add/update query parameters:

```php
// Returns: https://example.com/search?q=php
url()->withQuery('https://example.com/search', ['q' => 'php']);
```

Array parameters:

```php
// Returns: https://example.com/posts?tags[0]=php&tags[1]=mysql
url()->withQuery('https://example.com/posts', [
    'tags' => ['php', 'mysql']
]);
```

Remove query parameters:

```php
// Returns: example.com?page=1
url()->withoutQuery('example.com?page=1&sort=desc', 'sort');
```

Remove multiple parameters:

```php
// Returns: example.com?page=1
url()->withoutQuery('example.com?utm_source=fb&page=1', ['utm_source', 'utm_medium']);
```

Remove all query parameters:

```php
// Returns: example.com
url()->withoutQuery('example.com?page=1&sort=desc');
```

### URL Fragments

Add/update fragment:

```php
// Returns: example.com#section1
url()->withFragment('example.com', 'section1');
```

Update existing fragment:

```php
// Returns: example.com#new
url()->withFragment('example.com#old', 'new');
```

Remove fragment:

```php
// Returns: example.com
url()->withoutFragment('example.com#section');
```

### URL Normalization

Clean up URLs:

```php
// Returns: https://example.com/api/users
url()->normalize('https://example.com//blog/../api/./users//');
```

Join URL segments:

```php
// Returns: https://api.com/v1/users?sort=desc
url()->join('https://api.com', 'v1/', '/users', '?sort=desc');
```

## URL Signing

Generate a signed URL for a given route for secure access:

```php
// Generate signed URL (expires in 1 hour default)
$signedUrl = url()->sign('download.file', ['id' => 123]);

// Generate signed URL with custom expiration
$signedUrl = url()->sign('download.file', ['id' => 123], 7200); // 2 hours
```

Verify signed URL:

```php
if (url()->verify($signedUrl)) {
    // URL is valid and not expired
}
```

Verify with ignored parameters:

```php
url()->verify($signedUrl, ['utm_source', 'utm_medium']);
```

## Comprehensive Example Scenarios

The following document shows you some valid practical use case scenarios.

### 1. URL Generation Service

```php
class UrlGenerator
{
    public function getProfileUrl(int $userId, array $params = []): string
    {
        return url()->route('user.profile', array_merge(
            ['id' => $userId],
            $params
        ));
    }
    
    public function getDownloadUrl(string $fileId, int $expiresIn = 3600): string
    {
        return url()->sign('file.download', [
            'id' => $fileId
        ], $expiresIn);
    }
    
    public function getAssetUrl(string $path, bool $versioned = false): string
    {
        $url = url()->asset($path);
        
        if ($versioned) {
            return url()->withQuery($url, [
                'v' => filemtime(public_path($path))
            ]);
        }
        
        return $url;
    }
}
```

### 2. URL Validation Service

```php
class UrlValidator
{
    private array $defaultOptions = [
        'schemes' => ['https'],
        'require_scheme' => true,
        'max_length' => 2048
    ];
    
    public function isValidExternalUrl(string $url): bool
    {
        return url()->validate($url, array_merge(
            $this->defaultOptions,
            ['allowed_hosts' => $this->getAllowedHosts()]
        ));
    }
    
    public function isValidInternalUrl(string $url): bool
    {
        return url()->validate($url, array_merge(
            $this->defaultOptions,
            ['allowed_hosts' => [parse_url(get_env('APP_URL'), PHP_URL_HOST)]]
        ));
    }
    
    public function isValidAssetUrl(string $url): bool
    {
        $assetHost = get_env('ASSET_URL') 
            ? parse_url(get_env('ASSET_URL'), PHP_URL_HOST)
            : parse_url(get_env('APP_URL'), PHP_URL_HOST);
            
        return url()->validate($url, array_merge(
            $this->defaultOptions,
            ['allowed_hosts' => [$assetHost]]
        ));
    }
    
    private function getAllowedHosts(): array
    {
        // Load from configuration or environment
        return explode(',', get_env('ALLOWED_EXTERNAL_HOSTS', ''));
    }
}
```

### 3. URL Cleaner Service

```php
class UrlCleaner
{
    public function clean(string $url): string
    {
        // Remove tracking parameters
        $url = url()->withoutQuery($url, [
            'utm_source',
            'utm_medium',
            'utm_campaign',
            'utm_term',
            'utm_content'
        ]);
        
        // Normalize the URL
        $url = url()->normalize($url);
        
        // Remove fragments if present
        return url()->withoutFragment($url);
    }
    
    public function cleanBatch(array $urls): array
    {
        return array_map([$this, 'clean'], $urls);
    }
}
```

### 4. URL Security Service

```php
class UrlSecurity
{
    private array $sensitiveParams = [
        'token',
        'key',
        'password',
        'secret'
    ];
    
    public function sanitize(string $url): string
    {
        // Remove sensitive query parameters
        return url()->withoutQuery($url, $this->sensitiveParams);
    }
    
    public function createSecureDownloadUrl(string $path, array $params = []): string
    {
        // Generate a signed URL that expires in 5 minutes
        return url()->sign('secure.download', array_merge(
            ['path' => $path],
            $params
        ), 300);
    }
    
    public function verifySecureUrl(string $url): bool
    {
        // Verify the URL ignoring non-security related parameters
        return url()->verify($url, [
            'ref',
            'source',
            'campaign'
        ]);
    }
}
```

### 5. URL Analytics Service

```php
class UrlAnalytics
{
    public function addTracking(string $url, array $params): string
    {
        return url()->withQuery($url, array_merge(
            [
                'utm_source' => $params['source'] ?? 'website',
                'utm_medium' => $params['medium'] ?? 'link',
                'utm_campaign' => $params['campaign'] ?? 'default'
            ],
            array_filter([
                'utm_term' => $params['term'] ?? null,
                'utm_content' => $params['content'] ?? null
            ])
        ));
    }
    
    public function extractTrackingParams(string $url): array
    {
        $parts = url()->parse($url);
        $tracking = [];
        
        foreach ($parts['query'] as $key => $value) {
            if (str_starts_with($key, 'utm_')) {
                $tracking[substr($key, 4)] = $value;
            }
        }
        
        return $tracking;
    }
}
```
