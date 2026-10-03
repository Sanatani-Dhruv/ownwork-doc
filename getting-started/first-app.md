# Your First OwnWork Application

This guide walks through the basic structure of an OwnWork application and shows how to create a route, view, and controller.

## Application Entry Point

Every OwnWork request starts at:

```text
public/index.php
````

 The entry point loads the application bundler:

```
<?php

require __DIR__ . "/../bundle/Bundler.php";

$app = new Bundler();
$app->bundle();
```

 The bundler initializes the application and starts the HTTP kernel.

 You normally do not need to modify `public/index.php`.

 ## Define a Route

 Application routes are defined in:

```
bundle/Routes.php
```

 A route receives an HTTP method, URL, and handler.

 For example:

```
<?php

$route->get("/", "home.temp.php");
```

 This maps:

```
GET /
```

 to:

```
resources/views/home.temp.php
```

 ## Create the Home View

 Create:

```
resources/views/home.temp.php
```

 with:

```
<h1>Welcome to OwnWork</h1>

<p>Your application is running.</p>
```

 Start the development server:

```
php worker serve
```

 Then open:

```
http://localhost:8000
```

 The `/` route will render the view.

 ## Returning a String

 A route does not have to render a view.

 A callable can return a string:

```
$route->get("/hello", function () {
    return "Hello from OwnWork";
});
```

 Opening:

```
http://localhost:8000/hello
```

 returns the string from the route handler.

 ## Create a Controller

 For application logic, use a controller.

 Generate one with:

```
php worker make controller UserController
```

 The generated controller is placed in:

```
app/Controller/UserController.php
```

 A controller uses the `App\Controller` namespace.

 A basic controller action can be written as:

```
<?php

namespace App\Controller;

use Dhruv125\Coretex\Support\Request;
use Dhruv125\Coretex\Support\Response;

class UserController
{
    public function index(
        Request $request,
        Response $response
    ) {
        return "Users";
    }
}
```

 ## Connect the Controller to a Route

 Import the controller in `bundle/Routes.php`:

```
use App\Controller\UserController;
```

 Then register the controller action:

```
$route->get("/users", [
    UserController::class,
    "index"
]);
```

 The complete route file can look like:

```
<?php

use App\Controller\UserController;

$route->get("/", "home.temp.php");

$route->get("/hello", function () {
    return "Hello from OwnWork";
});

$route->get("/users", [
    UserController::class,
    "index"
]);
```

 The `/users` request is now resolved to:

```
UserController::index()
```

 ## Render a View from a Controller

 Views can be rendered with the global `view()` helper.

 For example:

```
public function index(
    Request $request,
    Response $response
) {
    return view("users.temp.php");
}
```

 Create the corresponding view:

```
resources/views/users.temp.php
```

 For example:

```
<h1>Users</h1>

<p>Welcome to the users page.</p>
```

 The route now follows this flow:

```
GET /users
    ↓
Route
    ↓
UserController::index()
    ↓
users.temp.php
    ↓
HTTP response
```

 ## Pass Data to a View

 The `view()` helper accepts an array of data:

```
return view("users.temp.php", [
    "title" => "Users",
]);
```

 The supplied values are made available inside the view.

 For example:

```
<h1>{{ $title }}</h1>
```

 The `.temp.php` extension indicates that the template should be processed by OwnWork's template system.

 ## Create Components with the Worker

 OwnWork includes a worker command for generating application components.

 For example:

```
php worker make controller UserController
```

```
php worker make middleware AuthMiddleware
```

```
php worker make model UserModel
```

```
php worker make service UserService
```

```
php worker make view users
```

 The worker uses the framework's templates under:

```
resources/template/
```

 and places generated application files into their corresponding directories.

 ## A Small Application

 A minimal application can therefore contain:

```
my-app/
├── app/
│   └── Controller/
│       └── UserController.php
│
├── bundle/
│   └── Routes.php
│
├── public/
│   └── index.php
│
└── resources/
    └── views/
        ├── home.temp.php
        └── users.temp.php
```

 The route configuration:

```
<?php

use App\Controller\UserController;

$route->get("/", "home.temp.php");

$route->get("/users", [
    UserController::class,
    "index"
]);
```

 The controller:

```
<?php

namespace App\Controller;

use Dhruv125\Coretex\Support\Request;
use Dhruv125\Coretex\Support\Response;

class UserController
{
    public function index(
        Request $request,
        Response $response
    ) {
        return view("users.temp.php");
    }
}
```

 The home view:

```
<h1>Welcome to OwnWork</h1>
```

 The users view:

```
<h1>Users</h1>
```

 ## Next Steps

 After creating a first application, the main concepts to learn are:

 - routing and route parameters
- controllers
- middleware
- requests and responses
- views and templates
- components
- services
- the worker CLI

> next: `getting-started/development.md`
