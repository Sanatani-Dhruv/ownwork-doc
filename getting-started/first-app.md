 # Your First OwnWork Application

 This guide walks through the basic structure of an OwnWork application and shows how to create a route, view, and controller.

 ## Application Entry Point

 Every OwnWork request starts at:

```bash
public/index.php
```

 The entry point loads the application bundler:

```php
<?php

declare(strict_types = 1);

ob_start();

use Bundle\Bundler;

require __DIR__ . "/../bundle/Bundler.php";

$app = new Bundler();
$app->bundle();

ob_end_flush();
```

 `Bundler` initializes the application environment and error handler, then creates the HTTP `Kernel`.

 You normally do not need to modify `public/index.php`.

 ## Define a Route

 Application routes are defined in:

```bash
bundle/Routes.php
```

 The `$route` object is provided when `Routes.php` is loaded by the HTTP kernel.

 A route can use an HTTP method, URL, and handler.

 For example:

```php
<?php

$route->get("/", "home.temp.php");
```

 A string route handler is treated as a view name by the Coretex route resolver.

 Create the corresponding view:

```bash
resources/views/home.temp.php
```

 with:

```html
<h1>Welcome to OwnWork</h1>

<p>Your application is running.</p>
```

 Start the development server:

```bash
php worker serve
```

 Then open:

```bash
http://localhost:8000
```

 The `/` route will resolve the string handler and render the corresponding view.

 > Views using the `.temp.php` extension are processed by OwnWork's view transpilation system. The `php worker serve` command itself starts PHP's built-in server; view transpilation is handled separately by the `transpile` worker command.

 ## Returning a String

 A route can also use a PHP callable instead of a view name:

```php
$route->get("/hello", function () {
    return "Hello from OwnWork";
});
```

 Opening:

```bash
http://localhost:8000/hello
```

 returns the string from the route handler.

 The route resolver invokes callable handlers directly.

 ## Create a Controller

 For application logic, use a controller.

 Generate one with:

```bash
php worker make controller UserController
```

 The generated controller is placed in:

```bash
app/Controller/UserController.php
```

 The worker creates the controller from:

```bash
resources/template/Controller.php
```

 The generated controller uses the `App\Controller` namespace and includes an `index()` method that receives Coretex `Request` and `Response` objects.

 A basic controller can be written as:

```php
<?php

declare(strict_types = 1);

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

```php
<?php

use App\Controller\UserController;
```

 Then register the controller action:

```php
$route->get("/users", [
    UserController::class,
    "index"
]);
```

 The complete route file can look like:

```php
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

 The `/users` request is resolved to:

```php
UserController::index()
```

 For a two-element controller handler, the route resolver creates an instance of the controller and invokes the specified method with the request and response objects.

 ## Route Parameters

 OwnWork supports dynamic route parameters using `{parameter}` segments.

 For example:

```php
$route->get("/users/{id}", [
    UserController::class,
    "show"
]);
```

 A request such as:

```bash
/users/42
```

 matches the route and produces:

```php
id = 42
```

 The dynamic parameters are stored on the request as the `dynamicParams` attribute. They are also passed to controller handlers by the route resolver.

 A controller action can therefore receive the request and response objects followed by the route parameters:

```php
public function show(
    Request $request,
    Response $response,
    array $params
) {
    return "User: " . $params["id"];
}
```

 ## Render a View from a Controller

 Views can be rendered with OwnWork's global `view()` helper.

 For example:

```php
public function index(
    Request $request,
    Response $response
) {
    return view("users.temp.php");
}
```

 The `view()` helper delegates to Coretex's view implementation.

 Create the corresponding view:

```bash
resources/views/users.temp.php
```

 For example:

```html
<h1>Users</h1>

<p>Welcome to the users page.</p>
```

 The route now follows this flow:

```bash
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

```php
return view("users.temp.php", [
    "title" => "Users",
]);
```

 The supplied values are passed to the view.

 For example:

```html
<h1>{{ $title }}</h1>
```

 The `{{ ... }}` syntax is processed by Coretex's template transpiler and converted into an escaped PHP output expression.

 The `.temp.php` extension identifies a template that OwnWork's transpilation system scans and compiles into `storage/views/`.

 ## Create Components with the Worker

 OwnWork includes a worker command for generating application components.

 For example:

```bash
php worker make controller UserController
```

```bash
php worker make middleware AuthMiddleware
```

```bash
php worker make model UserModel
```

```bash
php worker make service UserService
```

```bash
php worker make view users
```

 The worker supports these component types:

 - `controller`
- `middleware`
- `model`
- `service`
- `view`

 The generated files are placed in:

```bash
app/Controller/
app/Middleware/
app/Model/
app/Service/
resources/views/
```

 The worker uses the corresponding templates under:

```bash
resources/template/
```

 If a generated component already exists, the worker reports that the component already exists instead of replacing it.

 ## A Small Application

 A minimal application using a controller and views can contain:

```bash
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

```php
<?php

use App\Controller\UserController;

$route->get("/", "home.temp.php");

$route->get("/users", [
    UserController::class,
    "index"
]);
```

 The controller:

```php
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

```html
<h1>Welcome to OwnWork</h1>
```

 The users view:

```html
<h1>Users</h1>
```

 ## How a Request Is Handled

 At a high level, an OwnWork request follows this process:

```bash
public/index.php
      ↓
Bundler
      ↓
Kernel
      ↓
bundle/Routes.php
      ↓
Route matching
      ↓
Middleware
      ↓
RouteResolver
      ↓
View / Controller / Callable
      ↓
HTTP response
```

 The `Kernel` creates the request, response, route, and resolver objects, loads `bundle/Routes.php`, matches the request, executes middleware, and passes the selected handler to Coretex's `RouteResolver`.

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
