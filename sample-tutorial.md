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

Create a new file in `app/Controllers` folder named `TaskController.php` with following code contents. 

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
            ['title' => 'Buy shoes', 'status' => 'Done'], 
            ['title' => 'Eat Snacks', 'status' => 'Pending'], 
            ['title' => 'Learn PHP', 'status' => 'Pending'],
        ];

        app('response')->render('tasks/home', $data);
    }
}
```

In the above code, we have defined an array of fake tasks. Each task is an array having **title** and **status** key.

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
        <?= $task['status'] ?>
    </li>
<?php endforeach; ?>
</ul>
```

Now if you refresh the browser, you should see tasks rendered.

<img src="_media/tutorial/screen-3.png" style="width: 420px">

### Adding database

Now that we have successfully rendered our fake tasks list, its time to prepare database.

> We are going to use `MySQL` as our database system for this tutorial.

In your `MySQL` database, create a new database named `taskapp`. When done, execute
this query to create a table named `tasks`.

```sql
CREATE TABLE tasks ( 
    id INT NOT NULL AUTO_INCREMENT , 
    title VARCHAR(120) NOT NULL , 
    status ENUM('Done','Pending') NOT NULL DEFAULT 'Pending' , 
    PRIMARY KEY (id)
) ENGINE = InnoDB; 
```

Execute this query to insert some tasks manually in the table for now. Later we will create task add/update form to do the same.

```sql
INSERT INTO tasks 
    (title, status) 
VALUES 
    ('Buy shoes', 'Done'), 
    ('Eat Snacks', 'Pending'), 
    ('Learn PHP', 'Pending') 
```

To work with the new database, you will need to configure database credentials. Open `config/default.php` file and look for 'connection' key. There you need to configure 'mysql` key with appropriate database credentials.

```php
<?php

return [
    // ...
    'connection' => [
        // ...
        'mysql' => [
            'host' => 'localhost',
            'port' => 3306,
            'username' => 'root',
            'password' => 'root',
            'database' => 'taskapp',
            'options' => null,
        ]
    ],
];
```

Now its time to update our `TaskController.php` file to fetch tasks from database. Replace `app/ControllersTaskController.php` with following code.

```php
<?php

namespace App\Controllers;

class TaskController
{
    public function index()
    {
        $data['tasks'] = app('db')->table('tasks')->fetchAll(true);

        app('response')->render('tasks/home', $data);
    }
}
```

If you refresh your browser, you should see the tasks view as same as previous one before working with database.

<img src="_media/tutorial/screen-3.png" style="max-width: 420px">

Calling `app('db')` returns the database connection instance that defines a `table()` method. This methods takes the name of database table we are interested in querying. The `table()` method returns an instance of query builder. The `fetchAll()` method returns the result set as an array of objects by default. Passing it `true` returns the result set as an array.

#### Using objects instead of arrays

Before we go further with our **tasks** application, let us update `TaskController.php` to fetch tasks from database as an "array of objects". You will possibly find working with objects much cleaner than arrays in your view templates. Edit the contents of `app/Controllers/TaskController.php` as shown below.

```php
<?php

namespace App\Controllers;

class TaskController
{
    public function index()
    {
        // $data['tasks'] = app('db')->table('tasks')->fetchAll(true);
        $data['tasks'] = app('db')->table('tasks')->fetchAll();

        app('response')->render('tasks/home', $data);
    }
}
```

In the view template file, you will need to update array keys with object keys. Open `app/views/tasks/home.php` to update the template.

```php
<ul>
<?php foreach($tasks as $task): ?>
    <li>
        <?= $task->title ?> :
        <?= $task->status ?>
    </li>
<?php endforeach; ?>
</ul>
```

If you refresh your browser, you should see tasks rendered with no errors.

<img src="_media/tutorial/screen-3.png" style="width: 420px">

Now that you have successfully listed tasks from database, its time to build functionality to **add/edit** tasks in the database.

## Task Management

We have successfully rendered tasks from database. Now its time to enable task management feature by providing
**add/edit** tasks form.

Open `app/views/tasks/home.php` template and update the markup to support links editing a task and creating new tasks.

```php
<ul>
<?php foreach($tasks as $task): ?>
    <li>
        <?= $task->title ?> :
        <?= $task->status ?>
        <a href="<?= url('tasks/edit', $task->id) ?>">
            Edit
        </a>
    </li>
<?php endforeach; ?>
</ul>

<hr>

<a href="<?= url('tasks/add') ?>">
    + New Task
</a>
```

Now refresh your browser to view the updated screen.

<img src="_media/tutorial/screen-4.png" style="max-width: 520px">

<p class="tip">Note that we have used a utility function <code>url()</code> to generate our urls. This function takes any number of string arguments, concats them, and produces an URL relative to our application's base url.
</p>

### Add New Task

We will first start with adding new task feature. Clicking on **new task** link will take you to `/tasks/add` URL path which will throw `RouteNotFoundException` exception because we have not registered our `GET /tasks/add` route.

<img src="_media/tutorial/screen-5.png">

#### Show Task Form

Open `config/routes.php` file and add a new route for `/tasks/add`

```php
<?php

$route->group(['namespace' => 'App\Controllers'], function($route) {
    // ...
    $route->get('/tasks/add', 'TaskController@add');
});
```

Now create a method named `add()` in `TaskController.php` file.

```php
<?php

namespace App\Controllers;

class TaskController
{
    // ...

    public function add()
    {
        app('response')->render('tasks/form');
    }
}
```

We will need to create our task form template in `app/views/tasks/form.php` file.

```php
<form method="post">
    <input name="title" placeholder="Title" required>
    <button>Submit</button>
</form>

<a href="<?= url("tasks") ?>">Cancel</a>
```

Refresh your browser to see the task form.

<img src="_media/tutorial/screen-6.png" style="max-width:420px">

#### Post Task Form

Try to add a new task and submit the form. You should see `RouetNotFoundException` because
when the form is posted, the browser requests `POST /tasks/add` for which we have not registered any route in our routes definition file.

Add a route for `POST /tasks/add` in `config/routes.php` file.

```php
<?php

$route->group(['namespace' => 'App\Controllers'], function($route) {
    // ...
    $route->post('/tasks/add', 'TaskController@add');
});
```

Now try to re-submit the form. This time you should see the task form rendered with no exception. We now need to update `TaskController::add()` method to support inserting new data in database.

Update the `add()` method in `app/Controllers/TaskController.php` file as shown.

```php
<?php

namespace App\Controllers;

class TaskController
{
    // ...

    public function add()
    {
        $request = app('request');

        if($request->isPost()) {
            app('db')->table('tasks')->insert([
                'title' => $request->post('title')
            ]);

            redirect('tasks');
        }

        app('response')->render('tasks/form');
    }
}
```

<p class="tip">Note that <code>app('request')</code> gives an instance of current HTTP request.</p>

Try to add a new task. You should now see the new task listed. 

We first check if the current request method is `POST`. For that we call `app('request')->isPost()` method. To access the `POST` request form data, you can simply use the global `$_POST` array, but we used `app('request)->post()` method. It takes the name of form field and returns the data. We then finally insert the posted data in the database using `insert()` method.

> Note: We can also define a **TaskModel** that extends Lightpack's ORM model to ease working with **tasks** table. In this tutorial though, we will simply use the query builder to work with database.