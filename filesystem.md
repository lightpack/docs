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

To read the entire contents of a file, call `get()` method. It returns `null` if the file doesn't exist.

```php
$file->get('/path/to/file');
```