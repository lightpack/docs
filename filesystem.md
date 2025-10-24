# Filesystem

**PHP** already has a rich support for working with local filesystem in your application. However some file operations can be abstracted to make performing operations like copying/moving/deleting files and/or folders much easier. `Lightpack` comes with a handy filesystem library to help you working with local filesystem.

To get started, simply create an instance of `Lightpack\File\File` class.

```php
<?php

$file = new Lightpack\File\File();
```

Now you can access following listed methods as documented.

```php
$file->exists();
$file->read();
$file->write();
$file->delete();
$file->append();
$file->copy();
$file->rename();
$file->move();
$file->extension();
$file->size();
$file->modified();
$file->info();
$file->isDir();
$file->makeDir();
$file->copyDir();
$file->emptyDir();
$file->removeDir();
$file->recent();
$file->traverse();
```

## exists()

To check whether a file or folder exists, call `exists()` method. It returns a boolean `true` if it exists otherwise `false`.

```php
$file->exists('/path/to/file');
```

## read()

To read the entire contents of a file into a string, call `read()` method. It returns `null` if the file doesn't exist.

```php
$file->read('/path/to/file');
```

## write()

To write a string data to a file, call `write()` method. It will try to create the file if the file doesn't exist otherwise it will overwrite the existing file contents with new data.

```php
$file->write('/path/to/file', 'Hello World');
```

## delete()

To delete a file, call `delete` method. It returns `true` on success otherwise `false`.

```php
$file->delete('/path/to/file');
```

## append()

To append data to the end of the file, call `append()` method.

```php
$file->append('/path/to/file', 'Thanks.');
```

## copy()

To make a copy of a file, call `copy()` method. Pass it the source file path and destination path. If the destination path already exists, its content will be overwritten with source file contents.

This method returns `true` on success otherwise `false`.

```php
$file->copy('/path/to/source', '/path/to/destination');
```

## rename()

To rename a file, call `rename()` method. This is same as `copy()` except that it will delete the source file.

```php
$file->rename('/path/to/source', '/path/to/destination');
```

## move()

This is an alias of the `rename()` method above. It moves a source file to a given destination path and deletes the source file.

```php
$file->move('/path/to/source', '/path/to/destination');
```

## extension()

To get the extension of a file (e.g. txt, jpg, png, etc.), call `extension()` method.

```php
$file->extension('path/to/file');
```

## size()

To get the size of a file in bytes, call `size()` method.

```php
$file->size('/path/to/file'); // 1024
```

You can also get the formatted file size as `B, KB, MB, GB, TB`. Pass `true` as second parameter to this method.

```php
$file->size('/path/to/file', true); // 1.00KB
```

## modified()

To get the time the file was last modified, call `modified()` method. 

```php
$file->modified('/path/to/file');
```

It returns a UNIX timestamp by default. But you can also format the result by passing `true` as second parameter.

This will return the last modified date formatted as `M d, Y`.

```php
$file->modified('/path/to/file', true); // Feb 7, 2020
```

You can pass the date format as third parameter to this method.

```php
$file->modified('/path/to/file', true, 'd-M, Y'); // 7-Feb, 2020
```

## info()

Calling `info()` method returns an instance of `SplFileInfo` class from PHP SPL library. This class provides an object-oriented access to the properties of a file or directory.

- **Returns `null` if the file does not exist.**

It is worth your time to have a look at this class details.

[SplFileInfo](https://www.php.net/manual/en/class.splfileinfo.php)

```php
$fileinfo = $file->info('/path/to/file');
```

Now you can call all the methods defined in `SplFileInfo` class. For example:

```php
$fileinfo->getFilename();
```

## isDir()

To check if a filepath is a folder, call `isDir()` method. It returns `true` on success otherwise `false`.

```php
$file->isDir('/path/to/file');
```

## makeDir()

To create a new folder, call `makeDir()` method. This method returns `true` on success otherwise `false`.

```php
$file->makeDir('/path/to/file');
```

By default it tries to create the directory with a permission value `0777`. You can pass the permission as second parameter to this method.

```php
$file->makeDir('/path/to/file', 0775);
```

## copyDir()

To copy the contents of a source folder into another folder, call `copyDir()` method. It will recursively copy all the files and sub-folders from the source to the destination folder.

- **The third argument (`delete`) deletes the source folder after copy (useful for moves).**

This returns `true` on success otherwise `false`. 

```php
$file->copyDir('/path/to/source', '/path/to/destination');
```

If you want to delete the source folder after copy operation, pass `true` as third argument.

```php
$file->copyDir('/path/to/source', '/path/to/destination', true);
```

## emptyDir()

To empty a folder, call `emptyDir()` method. This method will delete all the files and folders in the source directory.

It returns `true` on success otherwise `false` on failure.

```php
$file->emptyDir('/path/to/file');
```

**Note** that this method will not delete the source folder itself.

## removeDir()

To delete a folder and all its content, call `removeDir()` method.

- **The second argument (`delete`) controls whether the directory itself is deleted (default: `true`).**

It returns `true` on success otherwise `false`.

```php
$file->removeDir('/path/to/file');
```

## recent()

To get the most recently modified file in a folder, call `recent()` method. 

- **Returns `null` if there are no files in the directory.**

This method returns an instance of [`SplFileInfo`](https://www.php.net/manual/en/class.splfileinfo.php) class which provides a number of methods to get the information about a file.

```php
$recent = $file->recent('/path/to/file');

// Now you can access all methods in SplFileInfo class.
$recent->getFilename();
```

## traverse()

To list all the files and folders in a given directory, call `traverse()` method. 

- **Returns `null` if the path is not a directory.**

This method will return an array of the contents of a folder with filepath as `key` and an instance of `SplFileInfo` as value.

```php
$files = $file->traverse('path/to/file');

foreach($files as $file) {
    echo $file->getFilename();
}
```

---

## moveDir()

To move a directory and all its contents to a new location, call `moveDir()` method. This is a wrapper around `copyDir()` that deletes the source after copy.

```php
$file->moveDir('/path/to/source', '/path/to/destination');
```

## hash()

To get a cryptographic hash (checksum) of a file's contents, call `hash()` method. This is useful for verifying file integrity, cache-busting, or detecting changes.

- **Returns `null` if the file does not exist.**

By default it uses `sha256` algorithm, but you can specify any hash algorithm supported by PHP.

```php
$file->hash('/path/to/file'); // sha256 by default
$file->hash('/path/to/file', 'md5'); // using MD5
$file->hash('/path/to/file', 'sha1'); // using SHA1
```

Common use cases:

```php
// Verify file integrity
$hash1 = $file->hash('original.zip');
$hash2 = $file->hash('downloaded.zip');
if ($hash1 === $hash2) {
    echo 'Files are identical';
}

// Cache busting
$version = $file->hash('app.js'); // Use hash as version
echo "<script src='app.js?v={$version}'"></script>";
```

## atomic()

To write contents to a file atomically, call `atomic()` method. This prevents partial or corrupted writes by writing to a temporary file first and then renaming it.

This is especially useful for:

* Configuration files that must never be corrupted
* Cache files that need consistency
* Data files that are read by other processes

```php
$file->atomic('/path/to/config.json', $jsonData);
```

**How it works:**

1. Writes content to a temporary file (e.g., `config.json.tmp.abc123`)
2. If write succeeds, renames temp file to target file
3. If anything fails, removes temp file and returns `false`

This ensures the target file is either completely written or not modified at all.

## getIterator()

To get a non-recursive iterator for a directory, call `getIterator()` method. This returns a `FilesystemIterator` that lists only the immediate contents of a directory.

- **Returns `null` if the path is not a directory.**

```php
$iterator = $file->getIterator('/path/to/dir');

foreach ($iterator as $file) {
    echo $file->getFilename();
}
```

<p class="tip">Use this when you only need files in the immediate directory, not subdirectories.</p>

## getRecursiveIterator()

To get a recursive iterator that traverses through all subdirectories, call `getRecursiveIterator()` method. This returns a `RecursiveIteratorIterator`.

- **Returns `null` if the path is not a directory.**

```php
$iterator = $file->getRecursiveIterator('/path/to/dir');

foreach ($iterator as $file) {
    if ($file->isFile()) {
        echo $file->getRealPath();
    }
}
```

You can also control the iteration mode:

```php
// Default: SELF_FIRST (parent before children)
$iterator = $file->getRecursiveIterator('/path/to/dir');

// CHILD_FIRST (children before parent, useful for deletion)
$iterator = $file->getRecursiveIterator('/path/to/dir', RecursiveIteratorIterator::CHILD_FIRST);
```

## sanitizePath()

To sanitize a file path for safe usage, call `sanitizePath()` method. This helps prevent directory traversal attacks and path manipulation vulnerabilities.

```php
$userInput = $_GET['file'];
$safePath = $file->sanitizePath($userInput);
```

**What it does:**

* Normalizes directory separators to the system standard
* Removes parent directory traversal sequences (`..`)
* Prevents path manipulation attacks

**Example:**

```php
// Dangerous user input
$input = '../../../etc/passwd';

// Sanitized output
$safe = $file->sanitizePath($input); // 'etcpasswd'
```

<p class="tip">Always sanitize user-provided file paths before using them in file operations.</p>