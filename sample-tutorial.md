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

### Defining Task Controller

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