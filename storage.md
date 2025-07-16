# Lightpack Storage: Complete Developer Guide

> **Lightpack Storage** provides a simple, extensible, and robust abstraction for file storage in PHP applications. It supports both local filesystem and AWS S3 out of the box, with a unified API for file operations.

## Supported Drivers

- **LocalStorage:**  
  - Stores files on the server’s local filesystem.
  - Designed for development, testing, or single-server production.
- **S3Storage:**  
  - Stores files in AWS S3 buckets.
  - Supports signed URLs for private files, directory listing, and robust error handling.

- Note: For S3 support:  
  `composer require aws/aws-sdk-php`

View available configurations options for storage in `config/storage.php`.

## Storage Interface

All drivers implement:

```php
interface Storage {
    public function read(string $path): ?string;
    public function write(string $path, string $contents): bool;
    public function delete(string $path): bool;
    public function exists(string $path): bool;
    public function copy(string $source, string $destination): bool;
    public function move(string $source, string $destination): bool;
    public function store(string $source, string $destination): void; // For uploads
    public function url(string $path, int $expiration = 3600): string;
    public function files(string $directory, bool $recursive = true): array;
    public function removeDir(string $directory, bool $delete = true): void;
}
```

---

## Usage Patterns

You can resolve an instance of `Lightpack\Storage\Storage` class from the dependency container or simply call the `storage()` function.

### Reading & Writing Files

```php
// Write a file
storage()->write('docs/readme.txt', 'Hello, Lightpack!');

// Read a file
$content = storage()->read('docs/readme.txt');
```

### Uploading Files

```php
// Store an uploaded file (e.g., from $_FILES)
storage()->store($_FILES['avatar']['tmp_name'], 'uploads/public/avatars/user1.png');
```

### Listing & Deleting Files

```php
// List all files in a directory (recursively)
$files = storage()->files('uploads/public');

// Delete a file
storage()->delete('uploads/public/avatars/user1.png');
```

### Generating URLs

```php
// For public files (local): returns web-accessible URL (e.g., /uploads/avatars/user1.png)
$url = storage()->url('uploads/public/avatars/user1.png');

// For S3: returns direct or signed URL (expiration applies for private files)
$url = storage()->url('private/reports/secret.pdf', 600); // 10 min signed URL
```

### Directory Operations

```php
// Remove an entire directory and its contents
storage()->removeDir('uploads/temp');
```

---

## Error Handling & Security

- **FileUploadException** is thrown on upload failure.
- All file operations return `bool` for success/failure (except `read`, which returns `null` on failure).
- S3 driver handles AWS exceptions gracefully.
- Local driver uses secure `move_uploaded_file()` for uploads.
- Only expose public files via URLs/routes; private files should use signed URLs or controller logic.
- Always validate user input and file paths.

---