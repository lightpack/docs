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
        return 'welcome';
    }
}
```

As you can see, defining controllers in Lightpack is quite easy.

## Route Params

For dynamic routes, you can access route parameters by defining them in the
controller's action method signature. For example, for the following route definition:

```php
route()->get('/users/:user/posts/:post', PostController::class);
```

Define your controller as:

```php
<?php

namespace App\Controllers;

class PostController
{
    public function index($user, $post)
    {
        // ...
    }
}
```
## Rendering Views

Define your view templates in <code>app/views</code> folder. To render a view template, 
call the <code>view()</code> method of <code>response()</code>. This method
takes a view template name and an optional array of view data as arguments.

```php
<?php

namespace App\Controllers;

class PageController
{
    return response()->view('page');
}
```

You can pass view data as second argument to <code>render()</code> method.

```php
class PageController
{
    public function index()
    {
        return response()->view('page', [
            'title' => 'Lightpack PHP',
        ]);
    }
}
```

Now you can access the view data array by key names as variables within your view template files. For example, if you want to access the <code>title</code> variable, you can do it as follows:

```php
Title: <?= $title ?>
```

You can also organize your view templates in folder within <code>app/views</code> folder. For
example, to render the template <code>app/views/page/home.php</code>,

```php
response()->view('page/home');
```

## JSON Response

For APIs, you may be interested in sending JSON response instead of view templates. For that,
simply call the <code>json()</code> method inherited from parent class.

```php
return response()->json(['framework' => 'Lighpack']);
```

## XML Response

For sending XML response, simply call the <code>xml()</code> method inherited from parent class
passing it the XML formatted string data.

```php
return response()->xml('xml_data_string');
```

<p class="tip">
Calling methods <code>view()</code>, or <code>json()</code>, or <code>xml()</code> automatically takes care of setting appropriate response content type and status code <code>200</code>,
thereby saving you some typing. However, you can set them manually using the methods available in <code>response()</code> object.
</p>

Read more about using [response()](/response) in **Lightpack PHP**.
