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

## Available Commands

Lightpack ships with a couple of console commands for code file generation to assist in 
rapid application development.

```terminal
create:event
create:model
create:filter
create:command
create:controller
```

### create:env

This command creates an environment file named `env.php` by copying the contents of `env.example.php` in project root directory.

```terminal
php lucy create:env
```

### create:event

This command creates an event class in `app/Events` folder. Simply pass it the name
of the event class.

```terminal
php lucy create:event SubscribeEvent
```

### create:model

This command creates a model class in `app/Models` folder. Simply pass it the name
of the model class and provide the `--table` flag with database table name.

```terminal
php lucy create:model ProductModel --table=products
```

### create:filter

This command creates a filter class in `app/Filters` folder. Simply pass it the name
of the filter class.

```terminal
php lucy create:filter InputFilter
```

### create:command

This command creates a command class in `app/Commands` folder. Simply pass it the name
of the command class.

```terminal
php lucy create:command HelloCommand
```

### create:controller

This command creates a controller class in `app/Controllers` folder. Simply pass it the name of the controller class.

```terminal
php lucy create:controller ProductController
```

You can also create a namespaced controller as shown:

```terminal 
php lucy create:controller 'Product\IndexController'
```

## Build Your Own Command

You can easily build your own custom command classes and register them to run from **PHP CLI** environment. Follow these steps as documented below.

### Create Command Handler

As a first step, you should create a command class inside 'app/Commands' folder. You can easliy do so with the help of Lightpack's `lucy` console assistant.

Inside the terminal from your project root, run following command to **auto-generate**
the command class.

```terminal
php lucy create:command TestCommand
```

This command should have created the `TestCommand` class inside `app/Commands` folder. Now the next step is to register a **console command** for this class.

### Register New Command

To register a new command, open `config/console.php` file. There in the array,
put your **command name** as key and **command handler** as a new key-value pair.

For example:

```php
<?php

return [
    'test:command' => App\Console\TestCommand::class,
];
```

Now you are ready to run your `test:command` from **PHP CLI** itself.

### Test Run Command

Open your terminal and navigate to your projects root folder. There you can fire
following command to test it.

```terminal
php lucy test:command
```

You should see an output same as below:

```terminal
Built my first test command
```

### Passing Command Arguments

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