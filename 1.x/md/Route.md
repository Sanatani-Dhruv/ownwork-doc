# Router
---

- Routing defines how to handle incoming HTTP Requests like, what to do when your app hit with a `GET /welcome` request.
- Your Application Routes are defined in `bundle/Routes.php`.
- There Are Some Default Route Set, they look like:

```php
<?php

namespace Bundle;

use App\Router\Route;
use App\Controller\UserController;

$route = new Route();

$route->get("/", "main.php");

$route->post("/game/{name}/game", [
   UserController::class, "showDetail", [
      "name" => "Hi",
      "id" => 11
   ]
]);

$route->end();
```

## Let's Understand This:

> The class `App\Router\Route` is Your Main Router Class, placed at `app/Router/Route.php`.

1. `Route::get(string $request_uri, $viewName_methodCall, array $arguments)`

- It defines HTTP GET Requests for your App.
- It's Arguments:
   - `$request_uri` : Incoming Url request from client ex. `/welcome`
   - `$viewName_methodCall` : It can accept two type of argument:
      - string: It will be treated as View name. We will Learn about Views later.
      - array: It will be treated as Method call:
         - 0th element would be Full class name (type: string)
         - 1st element would be method name(type: string)
         - 2nd element would be array of arguments going to be passed into method (type: depends on your defined method).
   - `$arguments` : It accepts associative array of elements which will be used as variable's in form key and value pair in the view.

- Example:

```php
<?php
$route->get('/welcome', 'welcome.php', [
   'id' => 15,
   'name' => 'Shyam'
]);

// Or

$route->get('/hello', [
    UserController::class, 'showDetail', [
        'foo' => 'bar',
        'baz' => 'bas'
    ]
]);
```

2. `Route::post(string $request_uri, $viewName_methodCall)`

- It defines HTTP POST Requests for your App.
- It's Working is Similar to `Router::get()`.

#Incomplete Doc
