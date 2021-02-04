# Filesystem

**PHP** already has a rich support for working with local filesystem in your application. However some file operations can be abstracted to make performing operations like copying/moving/deleting files and/or folders much easier. `Lightpack` comes with a handy filesystem library to help you working with local filesystem.

To get started, simply create an instance of `Lightpack\File\File` class.

```php
<?php

$file = new Lightpackk\File\File();
```

Now you can access following listed methods to work with files in your local system.

```php
$file->get();
$file->put();
$file->exists();
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

## get()

To read the entire contents of a file into a string, call `get()` method. It returns `null` if the file doesn't exist.

```php
$file->get('/path/to/file');
```

## put()

To write a string data to a file, call `put()` method. It will try to create the file if the file doesn't exist otherwise it will overwrite the existing file contents with new data.

```php
$file->put('/path/to/file', 'Hello World');
```

## exists()

To check whether a file or folder exists, call `exists()` method. It returns a boolean `true` if it exists otherwise `false`.

```php
$file->exists('/path/to/file');
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