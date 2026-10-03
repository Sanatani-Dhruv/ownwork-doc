# Controllers

Controllers contain application-level request handlers.

In an OwnWork application, controllers are stored under:

```text
app/Controller/
````

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

 A controller belongs to the `App\Controller` namespace.

 A typical controller has the following structure:

```
<?php

namespace App\Controller;

class UserController
{
    //
}
```

 The controller itself does not need to extend a framework base controller.

 ## Controller Actions

 A public method on a controller can be used as a route handler.

 For example:

```
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

```
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

 ## Request and Response Arguments

 Controller methods can receive Coretex's request and response objects.

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

 The `Request` object represents the incoming HTTP request.

 The `Response` object can be used to construct an HTTP response.

 See:

- Request
- Response

 ## Returning a String

 A controller action can return a string:

```
public function index()
{
    return "Users";
}
```

 The returned value becomes part of the HTTP response processing.

 For simple responses, this can be sufficient.

 ## Returning a View

 Controllers can render application views through the global `view()` helper:

```
public function index()
{
    return view("users.temp.php");
}
```

 The corresponding template is located under:

```
resources/views/users.temp.php
```

 For example:

```
<h1>Users</h1>
```

 ## Passing Data to a View

 The `view()` helper accepts data as its second argument:

```
public function index()
{
    return view("users.temp.php", [
        "title" => "Users"
    ]);
}
```

 The value can then be used by the template:

```
<h1>{{ $title }}</h1>
```

 The view system makes the supplied variables available when the template is rendered.

 ## Returning JSON

 A controller can use the response object to produce JSON:

```
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

 The response implementation is provided by Coretex.

 ## Dynamic Route Parameters

 Controllers can access parameters extracted from dynamic routes.

 Define a route:

```
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

 The parameters are available from the request's `dynamicParams` attribute:

```
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

 The router does not automatically load a model from the parameter.

 ## Multiple Route Parameters

 A controller can retrieve multiple parameters from the same request.

 Route:

```
$route->get(
    "/users/{user}/posts/{post}",
    [
        UserController::class,
        "post"
    ]
);
```

 Controller:

```
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

 ## Reading Request Data

 The request object should be used when controller logic needs information from the HTTP request.

 For example:

```
public function store(
    Request $request,
    Response $response
) {
    $name = $request->get("name");

    // Process the request.

    return $response->json([
        "name" => $name
    ]);
}
```

 The exact request API is documented separately in Request.

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

 The controller can create or use the service:

```
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

 The exact model implementation depends on the application.

 OwnWork does not provide an ORM through its controller layer.

 ## Controller and Middleware

 Middleware executes around route handling.

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

```
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

 Middleware should handle cross-cutting request concerns while the controller focuses on the application's operation.

 See Middleware.

 ## Controller Naming

 The worker accepts the controller name supplied by the developer:

```
php worker make controller UserController
```

 A conventional OwnWork controller therefore looks like:

```
UserController.php
```

 and is stored as:

```
app/Controller/UserController.php
```

 The generated namespace is:

```
namespace App\Controller;
```

 ## Controller Generation

 The worker's controller generator uses the project's controller template.

 The source template is located under:

```
resources/template/
```

 The worker generates the resulting PHP class in:

```
app/Controller/
```

 This means generated controllers can be used as starting points and then customized for the application's needs.

 ## Example Controller

 A small controller can combine routing parameters, request data, views, and responses:

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

```
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

 The resulting structure is:

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

 A controller is a convenient boundary between HTTP routing and application operations.

 Typical responsibilities include:

- receiving the request
- reading request and route data
- calling application services
- coordinating model operations
- selecting a view
- creating or returning a response

 Controllers should not be confused with the framework's HTTP kernel.

 The kernel coordinates the request lifecycle; controllers implement application-specific request handlers.

 ## What OwnWork Does Not Require

 OwnWork does not require controllers to:

- extend a base controller class
- implement a controller interface
- use a dependency-injection container
- use a particular ORM
- use a particular database
- return only one response type

 A controller is ultimately a class whose callable method can be registered as a route handler.

 ## Controller Flow

 A typical controller request can be represented as:

```
HTTP Request
     ↓
Route
     ↓
Middleware
     ↓
Controller Action
     ↓
Service / Model
     ↓
View or Response
     ↓
HTTP Response
```

 This keeps the framework's request infrastructure separate from application-specific behavior.

> next: `application/models.md`
