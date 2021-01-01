# Controllers

Controllers are classes that define action methods that respond to a request. Define 
your controllers within <code>app/controllers</code> folder. 

```php
<?php

namespace App\Controllers;

class HomeController
{
    public function index()
    {
        echo 'welcome';
    }
}
```

As you can see, defining controllers in Lightpack is quite easy.

## Route Params

For dynamic routes, you can access route parameters by defining them in the
controller's action method signature. For example, for the following route definition:

```php
$route->get('/page/:num/status/:str', 'PageController@index');
```

Define your controller as:

```php
<?php

namespace App\Controllers;

class PageController
{
    public function index($page, $status)
    {
        // ...
    }
}
```
## Rendering Views

Define your view templates in <code>app/views</code> folder. To render a view template, 
call the <code>render()</code> method of <code>app('response')</code>. This method
takes a view template name and an optional array of view data as arguments.

```php
<?php

namespace App\Controllers;

class PageController
{
    app('response')->render('page');
}
```

You can pass view data as second argument to <code>render()</code> method.

```php
class PageController
{
    public function index()
    {
        app('response')->render('users', [
            'title' => 'Lightpack PHP',
        ]);
    }
}
```

Now you can access the view data array by key names as variables within your view template files.

```php
// app/views/page.php
Title: <?= $title ?>
```

You can also organize your view templates in folder within <code>app/views</code> folder. For
example, to render the template <code>app/views/page/home.php</code>,

```php
app('response')->render('page/home');
```

## JSON Response

For APIs, you may be interested in sending JSON response instead of view templates. For that,
simply call the <code>json()</code> method inherited from parent class.

```php
app('response')->json(['framework' => 'Lighpack']);
```

## XML Response

For sending XML response, simply call the <code>xml()</code> method inherited from parent class
passing it the XML formatted string data.

```php
app('response')->xml('xml_data_string');
```

<p class="tip">
Calling methods <code>render()</code>, or <code>json()</code>, or <code>xml()</code> automatically takes care of setting appropriate response content type and status code <code>200</code>,
thereby saving you some typing.
</p>
