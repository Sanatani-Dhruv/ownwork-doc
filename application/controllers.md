# Controllers

 Controllers contain application-level request handlers.

 In an OwnWork application, controllers are stored under:

```
app/Controller/
```

 A controller is normally connected to the application through a route declared in:

```
bundle/Routes.php
```

 OwnWork provides a worker command for generating controller classes.

 ## Creating a Controller

 Generate a controller with:

```
php worker make controller UserController
```

 The generated class is placed under:

```
app/Controller/UserController.php
```

 The generated controller uses the `App\Controller` namespace and includes an `index()` method that receives Coretex's `Request` and `Response` objects.

 The controller template currently used by OwnWork is:

```php
<?php

declare(strict_types = 1);

namespace App\Controller;

use Dhruv125\Coretex\Viewer\View;
use Dhruv125\Coretex\Support\Request;
use Dhruv125\Coretex\Support\Response;

class UserController
{
    function __construct()
    {
        // Default Controller
    }

    public function index(Request $request, Response $response)
    {
        // ...
    }
}
```

 The generated controller does not extend a framework base controller.

 ## Controller Actions

 A public controller method can be registered as a route handler.

 For example:

```php
<?php

namespace App\Controller;

class UserController
{
    public function index()
    {
        return "Users";
    }
}
```

 Register the method in `bundle/Routes.php`:

```php
<?php

use App\Controller\UserController;

$route->get("/users", [
    UserController::class,
    "index"
]);
```

 A request to:

```
GET /users
```

 is resolved to:

```
UserController::index()
```

 The controller method is invoked by Coretex's `RouteResolver`, which OwnWork creates in the HTTP kernel.

 ## Request and Response Arguments

 Controller methods can receive Coretex's request and response objects:

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
        return "Users";
    }
}
```

 The `Request` object represents the incoming HTTP request.

 The `Response` object represents the HTTP response being constructed.

 Because controller handlers are ultimately passed to Coretex's `RouteResolver`, the exact handler behavior is provided by Coretex rather than by a separate OwnWork controller base class.

 ## Returning a String

 A controller action can return a string:

```php
<?php

public function index()
{
    return "Users";
}
```

 OwnWork's kernel detects a string returned from the middleware and route-handler pipeline and places it into the response body before dispatching the response.

 For simple HTML or text responses, returning a string can therefore be sufficient.

 ## Returning a View

 Controllers can render application views through the global `view()` helper:

```php
<?php

public function index()
{
    return view("users.temp.php");
}
```

 The corresponding application view is located under:

```
resources/views/users.temp.php
```

 For example:

```
<h1>Users</h1>
```

 The view system itself is provided through the Coretex dependency.

 ## Passing Data to a View

 The `view()` helper can receive data as its second argument:

```php
<?php

public function index()
{
    return view("users.temp.php", [
        "title" => "Users"
    ]);
}
```

 The supplied values can then be used by the template:

```
<h1>{{ $title }}</h1>
```

 `.temp.php` templates are processed by OwnWork/Coretex's template system before the resulting PHP representation is executed.

 ## Returning JSON

 A controller can use the response object to produce JSON:

```php
<?php

public function index(
    Request $request,
    Response $response
) {
    return $response->json([
        "users" => []
    ]);
}
```

 This is useful for API-style routes.

 The `Response` implementation is provided by Coretex.

 ## Dynamic Route Parameters

 Controllers can access parameters extracted from dynamic routes.

 Define a route:

```php
<?php

$route->get("/users/{id}", [
    UserController::class,
    "show"
]);
```

 For:

```
GET /users/42
```

 the router extracts:

```
[
    "id" => "42"
]
```

 OwnWork places these values in the request's `dynamicParams` attribute before executing middleware and the route handler.

 A controller can read them with:

```php
<?php

public function show(
    Request $request,
    Response $response
) {
    $params = $request->getAttribute("dynamicParams");

    $id = $params["id"];

    return $response->json([
        "id" => $id
    ]);
}
```

 The OwnWork kernel sets this request attribute from the `params` returned by the matched Coretex route.

 The router does not automatically load a model from the parameter.

 ## Multiple Route Parameters

 A controller can retrieve multiple parameters from the same request.

 Route:

```php
<?php

$route->get(
    "/users/{user}/posts/{post}",
    [
        UserController::class,
        "post"
    ]
);
```

 Controller:

```php
<?php

public function post(
    Request $request,
    Response $response
) {
    $params = $request->getAttribute("dynamicParams");

    $userId = $params["user"];
    $postId = $params["post"];

    return $response->json([
        "user" => $userId,
        "post" => $postId
    ]);
}
```

 The parameter names come directly from the route definition.

 ## Reading Request Data

 The request object should be used when controller logic needs information from the HTTP request.

 For example:

```php
<?php

public function store(
    Request $request,
    Response $response
) {
    $name = $request->get("name");

    return $response->json([
        "name" => $name
    ]);
}
```

 The exact request API belongs to Coretex and should be consulted when working with request-specific functionality.

 ## Using Services

 Controllers can delegate reusable application logic to services.

 For example:

```
app/
├── Controller/
│   └── UserController.php
└── Service/
    └── UserService.php
```

 A controller can create or use a service:

```php
<?php

public function store(
    Request $request,
    Response $response
) {
    $service = new UserService();

    $user = $service->create(
        $request->get("name")
    );

    return $response->json($user);
}
```

 OwnWork does not impose a dependency-injection container or a required service interface.

 The service architecture is therefore application-defined.

 ## Using Models

 Models belong under:

```
app/Model/
```

 A controller can delegate data operations to a model or service.

 For example:

```
Controller
    ↓
Service
    ↓
Model
```

 OwnWork provides the application directory and worker generation support for models, but it does not impose an ORM or database implementation.

 The actual persistence layer is application-specific.

 ## Controller and Middleware

 Middleware executes before the final route handler and can surround its execution.

 For example:

```
Request
   ↓
AuthMiddleware
   ↓
UserController
   ↓
Response
```

 Register middleware for a controller route:

```php
<?php

$route->get("/users", [
    UserController::class,
    "index"
]);

$route->middleware(
    "GET",
    "/users",
    [AuthMiddleware::class, "handle"]
);
```

 The controller remains responsible for the application operation, while middleware handles concerns that should apply around request processing.

 ## Controller Naming

 The worker accepts the controller name supplied by the developer:

```
php worker make controller UserController
```

 A conventional controller therefore looks like:

```
UserController.php
```

 and is stored under:

```
app/Controller/UserController.php
```

 The generated namespace is:

```php
<?php

namespace App\Controller;
```

 ## Controller Generation

 The worker's controller generator uses the controller template under:

```
resources/template/
```

 The current controller template provides:

 - `declare(strict_types = 1);`
- the `App\Controller` namespace
- Coretex `Request` and `Response` imports
- an empty constructor
- an `index(Request $request, Response $response)` method

 The generated file is written under:

```
app/Controller/
```

 Developers can then modify the generated class for their application.

 ## Example Controller

 A controller can combine request data, route parameters, views, and responses:

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
        return view("users.temp.php", [
            "title" => "Users"
        ]);
    }

    public function show(
        Request $request,
        Response $response
    ) {
        $params = $request->getAttribute(
            "dynamicParams"
        );

        $id = $params["id"];

        return $response->json([
            "id" => $id
        ]);
    }
}
```

 Routes:

```php
<?php

use App\Controller\UserController;

$route->get("/users", [
    UserController::class,
    "index"
]);

$route->get("/users/{id}", [
    UserController::class,
    "show"
]);
```

 The request flow is:

```
Request
   │
   ▼
Route
   │
   ├── /users
   │      ↓
   │   UserController::index()
   │      ↓
   │   users.temp.php
   │
   └── /users/{id}
          ↓
      UserController::show()
          ↓
        JSON
```

 ## What Controllers Are Responsible For

 A controller is a boundary between HTTP routing and application operations.

 Typical responsibilities include:

 - receiving the request
- reading request and route data
- calling application services
- coordinating model operations
- selecting a view
- creating or returning a response

 Controllers should not be confused with the framework's HTTP kernel.

 The kernel coordinates request processing, while controllers implement application-specific request handlers.

 ## What OwnWork Does Not Require

 OwnWork does not require controllers to:

 - extend a base controller class
- implement a controller interface
- use a dependency-injection container
- use a particular ORM
- use a particular database
- return only one response type

 A controller is an application class whose registered callable method is resolved by Coretex's route resolver.

 ## Controller Flow

 A typical controller request can be represented as:

```
HTTP Request
     ↓
Route
     ↓
Middleware
     ↓
RouteResolver
     ↓
Controller Action
     ↓
Service / Model
     ↓
View or Response
     ↓
HTTP Response
```

 OwnWork's kernel creates the Coretex `RouteResolver` and invokes it after the middleware chain completes.

> `application/models.md`
