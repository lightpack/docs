# Sample Tutorial

In this tutorial, we are going to build a simple **tasks** application. The purpose of this
tutorial is to give you a quick introduction of working with `Lightpack PHP` framework.

> **Note:** This tutorial has been tested on an **Ubuntu** dev server running **apache**.

So let's get started.

## Install Framework

Installing `Lightpack` is dead simple. Just clone the framework's GitHub repository in your
server root.

```bash
git clone https://github.com/lightpack/lightpack.git
```

Move into your repository folder.

```bash
cd lightpack
```

Now run `composer` to install the project's dependencies.

```bash
composer install --no-dev -vvv
```            

## Running The App

You can simply visit `http://localhost/lightpack` (or whatever IP address you have configured to run your PHP applications locally) in your browser to test your application. 

Other option is to run PHP's built in web server to test your application.

```bash
php -S 127.0.0.1:8080
```

Now you should see this screen below.

<img src="_media/tutorial/screen-1.png">

## Listing Tasks

We first need to define a route for `GET /tasks` in `config/routes.php` file. There you will already find a route registered for `GET /` homepage. 

### Adding /tasks route

Add a new route for **tasks** in `config/routes.php` file.

```php
$route->get('/tasks', 'TaskController@index');
```

We have registered `/tasks` route specifying `index` method of `TaskController`. So we now need to define our controller.

### Defining tasks controller

Create a new file in `app/Controller` folder named `TaskController.php` with following code contents. 

```php
<?php

namespace App\Controllers;

class TaskController
{
    public function index()
    {
        echo 'Tasks...';
    }
}
```

Now if you visit `/tasks` in your browser, you should see the following screen.

<img src="_media/tutorial/screen-2.png" style="width: 420px">

### Rendering tasks view 

Rather than simply echoing tasks, we would like to render a view template with actual
tasks list. For now let us fake tasks data in our controller. Later in this tutorial, we will
fetch tasks from database itself.

Edit `app/Controllers/TaskController.php` file to add fake tasks and render tasks view template.

```php
<?php

namespace App\Controllers;

class TaskController
{
    public function index()
    {
        $data['tasks'] = [
            ['title' => 'Buy shoes', 'status' => 1], 
            ['title' => 'Eat Snacks', 'status' => 0], 
            ['title' => 'Learn PHP', 'status' => 0],
        ];

        app('response')->render('tasks/home', $data);
    }
}
```

In the above code, we have defined an array of fake tasks. Each task is an array having **title** and **status** key. By **status**, we mean whether the task is done **(1)** or not **(0)**.

To render a view template, we call the `render()` method of response object returned by `app('response')` function call. This method takes path to a view template file relative to `app/views` directory and an optional array of data.

So the following code snippet means we want to render `app/views/tasks/home.php` template file with `$data` array.

```php
app('response')->render('tasks/home', $data);
```

To render our tasks view, create `tasks` folder with `home.php` file in `app/views` directory.

```php
<ul>
<?php foreach($tasks as $task): ?>
    <li>
        <?= $task['title'] ?> :
        <?= $task['status'] == 0 ? 'Pending' : 'Done' ?>
    </li>
<?php endforeach; ?>
</ul>
```

Now if you refresh the browser, you should see tasks rendered.

<img src="_media/tutorial/screen-3.png" style="width: 420px">