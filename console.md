# Console: Lucy CLI

**Lightpack** comes with a console CLI assistant named **`Lucy CLI`**. It is purposefully kept simple and is dead easy to start building CLI features for your application.

Suppose you want to build a CLI command that generates fresh new **controller** class
code file for you. For that you may fire following command from your terminal inside
your project root folder.

```terminal
php lucy create:controller Product
```

Now that you have got an overview of what **`Lucy CLI`** is, read the docs below
for detailed guidelines.

## Register New Command

To register a new command, open `config/console.php` file. There in the array,
put your **command name** as key and **command handler** as a new key-value pair.

For example:

```php
<?php

return [
    'test:command' => App\Console\TestCommand::class,
];
```

Now the next step is to create the **`TestCommand`** class.

## Create Command Handler

In the `app/Console` folder, create a new class file named `TestCommand.php` with following code example:

```php
<?php

namespace App\Console;

use Lightpack\Console\ICommand;

class TestCommand implements ICommand
{
    public function run(array $arguments = [])
    {
        fputs(STDOUT, "Built my first test command\n");
    }
}
```

## Test Run Command

Open your terminal and navigate to your projects root folder. There you can fire
following command to test it.

```terminal
php lucy test:command
```

You should see an output same as below:

```terminal
Built my first test command
```

## Passing Command Arguments

You can pass any number of argument/options to the command you tested.

```terminal
php lucy test:command arg1 arg2 -n=23 --flag='hello world'
```

Now in the **`run()`** method of your TestCommand class, you can access all the
arguments passed in the **`$arguments`** array.

```php
<?php
...
class TestCommand
{
    public function run(array $arguments = [])
    {
        print_r($arguments);
    }
}
...
```

You should see an ouput as shown below:

```terminal
Array
(
    [0] => arg1
    [1] => arg2
    [2] => -n=23
    [3] => --flag=hello world
)
```