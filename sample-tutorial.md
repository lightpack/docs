# Sample Tutorial

In this tutorial, we are going to build a simple **tasks** application. The purpose of this
tutorial is to give you a quick introduction of working with `Lightpack PHP` framework. This tutorial will not include working with advanced features like **containers**, **logging**, **filters**, **events**, **validation**, **commands**, **layouts** etc. 

It will be simple enough to get you a taste of this framework with a simple task management app.

> [Browse tutorial repository link here.](https://github.com/lightpack/taskapp)

<p class="tip">Note: This tutorial has been tested on an <b>Ubuntu</b> dev server running <b>Apache</b>.</p>


So let's get started.

## Create Project

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
composer install --no-dev -v
```            

## Running The App

<p class="tip">You need to configure your server to serve this application from <code>public</code> folder as web root.</p>

But for local development you can run PHP's built-in web server. For that move inside your project root directory and fire this command from terminal.

```bash
php -S localhost:8080 -t public
```

Please change the **IP adress** and **Port** as per your dev machine.

Now open the app in your browser and you should see this screen below.

<img src="_media/tutorial/screen-1.png">

## Listing Tasks

We first need to define a route for `GET /tasks` in `routes/web.php` file. There you will already find a route registered for `GET /` homepage. 

### Adding /tasks route

Add a new route for **tasks** in `routes/web.php` file.

```php
route()->get('/tasks', TaskController::class);
```

We have registered `/tasks` route specifying default method `index` of controller class `TaskController`. So we now need to define our controller.

### Defining tasks controller

Inside your project root, from your terminal fire this command:

```terminal
php lucy create:controller TaskController
```

**Note:** `lucy` is Lightpack's CLI assistant tool. Read more about it [here](https://lightpack.github.io/docs/#/console)

This should have created `TaskController` inside `app/Controllers` folder. Update `TaskController` with 
following code contents. 

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

        return response()->view('tasks/index', $data);
    }
}
```

In the above code, we have defined an array of fake tasks. Each task is an array having **title** and **status** key.

To render a view template, we call the `view()` method of response object returned by `response()` function call. This method takes path to a view template file relative to `app/views` directory and an optional array of data.

So the following code snippet means we want to render `app/views/tasks/index.php` template file with `$data` array.

```php
response()->view('tasks/index', $data);
```

To render our tasks view, create `tasks` folder with `index.php` file in `app/views` directory.

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

**Note:** Read more about `response()` and view rendering [here](https://lightpack.github.io/docs/#/response).

## Adding database

Now that we have successfully rendered our fake tasks list, its time to prepare database.

### Create Table

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

### Insert Data

Execute this query to insert some tasks manually in the table for now. Later we will create task add/update form to do the same.

```sql
INSERT INTO tasks 
    (title, status) 
VALUES 
    ('Buy shoes', 'Done'), 
    ('Eat Snacks', 'Pending'), 
    ('Learn PHP', 'Pending') 
```

### Configure Application

To work with the new database, you will need to configure database credentials. Copy the contents of `env.example.php` file into `env.php` inside your project root.

Look for **MySQL** settings. There you need to configure database credentials with appropriate values.

```php
/**
 * MySQL settings.
 */

'DB_HOST' => 'localhost',
'DB_PORT' => 3306,
'DB_NAME' => '',
'DB_USER' => '',
'DB_PSWD' => '',
```

## List All Tasks

To work with `tasks` table in database, we will create a `Task` class. Fire following command inside terminal
from project root.

```terminal
php lucy create:model Task --table=tasks
```

**Note:** Read more about `models` [here](https://lightpack.github.io/docs/#/models).

This should have created `Task` in `app/Models` folder.

### Update Controller

We will update our `TaskController` to use this model. Update your controller's `index()` method with following code.

```php
public function index()
{
    return response()->view('tasks/index', [
        'tasks' => Task::query()->all(),
    ]);
}
```

### Update Tasks View

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

Open `app/views/tasks/index.php` template and update the markup to support links editing a task and creating new tasks.

```php
<ul>
<?php foreach($tasks as $task): ?>
    <li>
        <?= $task->title ?> :
        <?= $task->status ?>
        <a href="<?= url()->to('tasks/edit', $task->id) ?>">
            Edit
        </a>
    </li>
<?php endforeach; ?>
</ul>

<hr>

<a href="<?= url()->to('tasks/add') ?>">
    + New Task
</a>
```

Now refresh your browser to view the updated screen.

<img src="_media/tutorial/screen-4.png" style="max-width: 520px">

<p class="tip">Note that we have used a utility function <code>url()</code> to generate our urls. The `to()` method of url generator object takes any number of string arguments, concats them, and produces an URL relative to our application's base url.
</p>

### Add New Task

We will first start with adding new task feature. Clicking on **new task** link will take you to `/tasks/add` URL path which will throw `RouteNotFoundException` exception because we have not registered our `GET /tasks/add` route.

<img src="_media/tutorial/screen-5.png">

#### Show task form

Open `routes/web.php` file and add a new route for `/tasks/add`

```php
route()->get('/tasks/add', TaskController::class, 'showAddForm');
```

Now create a method named `showAddForm()` in `TaskController.php` file.

```php
public function showAddForm()
{
    return response()->view('tasks/form');
}
```

We will need to create our task form markup in `app/views/tasks/form.php` file.

```php
<form method="post">
    <input name="title" placeholder="Title" required>
    <button>Submit</button>
</form>

<a href="<?= url()->to('tasks') ?>">Cancel</a>
```

Refresh your browser to see the task form.

<img src="_media/tutorial/screen-6.png" style="max-width:420px">

#### Submit task form

Try to add a new task and submit the form. You should see `RouetNotFoundException` because
when the form is posted, the browser requests `POST /tasks/add` for which we have not registered any route in our routes definition file.

Add a route for `POST /tasks/add` in `routes/web.php` file.

```php
route()->post('/tasks/add', TaskController::class, 'postAddForm');
```

We now need to update `TaskController::postAddForm()` method to support inserting new data in database.

Update the `TaskController` with following method:

```php
public function postAddForm()
{
    $task = new Task();
    $task->title = request()->input('title');
    $task->save();

    return redirect()->to('tasks');
}
```

<p class="tip">Note that <code>request()</code> gives an instance of current HTTP request.</p>

Try to add a new task. You should now see the new task listed. 

To access the `POST` request form data, you can simply use the global `$_POST` array, but we used `request()->input()` method. It takes the name of form field and returns the data. We then finally insert the posted data in the database using `save()` method on the model.

Note: The **Task** extends Lightpack's ORM model to ease working with **tasks** table. Read more about [ORM models](https://lightpack.github.io/docs/#/orm-introduction).

### Edit Task

Click on **edit** task link to edit a task. You should see `RouteNotFoundException` because we need to register a route for editing our task. Let us solve that now.

#### Show task form

Update `routes/web.php` file to support task edting.

```php
route()->get('/tasks/edit/:id', TaskController::class, 'showEditForm');
```

In the `TaskController.php` file, add a new method named `showEditForm()`.

```php
public function showEditForm($id)
{
    $task = new Task($id);

    return response()->view('tasks/form', [
        'task' => $task,
    ]);
}
```

This method will search for the task with matching task ID and return the result.

We are going to use the same form to edit a task. Update `app/views/tasks/form.php` as shown below.

```php
<form method="post">
    <input name="title" placeholder="Title" value="<?= $task->title ?? '' ?>" required>
    <button>Submit</button>
</form>

<a href="<?= url()->to('tasks') ?>">Cancel</a>
```

Now if you try to edit a task, you should see the form populated with task title.

<img src="_media/tutorial/screen-7.png" style="max-width: 420px">

Let us also populate **tasks** status field. Update `app/views/tasks/form.php` as shown below.

```php
<form method="post">

    <!-- Title -->
    <input name="title" placeholder="Title" value="<?= $task->title ?? '' ?>" required>
    
    <!-- Status -->
    <?php if($task->status ?? null): ?>
        <select name="status">
            <?php foreach(['done', 'pending'] as $status): ?>
                <option <?= $task->status == $status ? 'selected' : '' ?>>
                    <?= $status ?>
                </option>
            <?php endforeach; ?>
        </select>
    <?php endif; ?>
    
    <!-- Submit -->
    <button>Submit</button>

</form>

<a href="<?= url()->to('tasks') ?>">Cancel</a>
```

Refresh your browser to see the task form populated with **status** field.

<img src="_media/tutorial/screen-8.png" style="max-width: 420px">


#### Post task form

Let us add a route for handling `POST` requests to edit our tasks. Update `routes/web.php` file a new route definition for posting task edit form.

```php
route()->post('/tasks/edit/:id', TaskController::class, 'postEditForm');
```

Now add `postEditForm()` method in `TaskController`.

```php
public function postEditForm($id)
{
    $task = new Task($id);
    $task->title = request()->input('title');
    $task->status = request()->input('status');
    $task->save();

    redirect('tasks');
}
```

Now try to edit a task. It should successfully update the database to reflect changes.

> [Browse tutorial repository link here.](https://github.com/lightpack/taskapp)

## Final Notes

If you reached so far with this tutorial, you definitely have got acquainted with `Lightpack` framework. We could have extended this tutorial to introduce **filters**,
**models**, **events**, **layouts**, and a couple of tips for much cleaner code. But to 
keep things simple, we restricted it to only have an introduction about working with `Lightpack`.

Performance benchmark shows `Lightpack` outshines some of the well known frameworks in PHP community. At the same time it has very small learning curve.

We encourage you to evaluate `Ligtpack` with your own benchmarks and let us know about your experience. `Lightpack` is in its **alpha** version for now and is bound to change in its architecture. 

**This is the best time to support this framework and become a core contributor.**

<p class="tip">Open <b>pull requests</b> and <b>issues</b> if you find it good enough for your attention. 😀</p>