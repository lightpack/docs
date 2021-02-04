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

## size()

The get the size of a file in bytes, call `size()` method.

```php
$file->size('/path/to/file'); // 1024
```

You can also get the formatted file size as `B, KB, MB, GB, TB`. Pass `true` as second parameter to this method.

```php
$file->size('/path/to/file'); // 1KB
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

It is worth your time to have a look at this class details.

[SplFileInfo](https://www.php.net/manual/en/class.splfileinfo.php)

```php
$fielinfo = $file->info('/path/to/file');
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

By default it tries to create the directory with a permission value `0777`. You can pass the permission assecond parameter to this method.

```php
$file->makeDir('/path/to/file', 0775);
```

## copyDir()

To copy the contents of a source folder into another folder, call `copyDir()` method. It will recursively copy all the files and sub-folders from the source to the destination folder.

This returns `true` on success otherwise `false`. 

```php
$file->copyDir('/path/to/source', '/path/to/destination');
```

If you want to delete the surce folder after copy operation, pass
`true` as third argument.

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

It returns `true` on success otherwise `false`.

```php
$file->removeDir('/path/to/file');
```