# Request Lifecycle

OwnWork processes an HTTP request through a small sequence of bootstrap, routing, middleware, handler resolution, and response steps.

Understanding this lifecycle helps when working with routes, controllers, middleware, views, and error handling.

## Overview

The request lifecycle can be represented as:

```text
HTTP Request
     │
     ▼
public/index.php
     │
     ▼
Bundler
     │
     ├── Load Composer
     ├── Load environment
     └── Configure error handling
     │
     ▼
Kernel
     │
     ├── Create Request
     ├── Create Response
     ├── Create Router
     └── Register Routes
     │
     ▼
Route Matching
     │
     ▼
Route Resolution
     │
     ▼
Middleware Pipeline
     │
     ▼
Controller / Closure / View
     │
     ▼
Response
     │
     ▼
HTTP Output
````

 ## 1\. HTTP Request

 The web server receives a request from the client.

 For example:

```
GET /users/42
```

 PHP executes the application's front controller:

```
public/index.php
```

 The `public` directory is the application's web document root.

 ## 2\. Front Controller

 `public/index.php` loads the OwnWork bundler:

```php
<?php

require __DIR__ . "/../bundle/Bundler.php";

$app = new Bundler();
$app->bundle();
```

 The front controller does not define routes or process individual requests itself.

 Its responsibility is to start the application.

 ## 3\. Application Bootstrap

 `bundle/Bundler.php` performs the application bootstrap.

 The bootstrap includes:

 - checking that the application has been set up
- loading Composer's autoloader
- loading environment configuration
- configuring the error handler
- creating the application kernel
- starting request handling

 Conceptually:

```
Bundler
   │
   ├── Environment
   ├── Error handling
   └── Kernel
```

 ## 4\. Kernel Initialization

 The HTTP kernel is located at:

```
app/Http/Kernel.php
```

 The kernel is responsible for coordinating request processing.

 It creates the main HTTP and routing objects used by the application, including:

 - `Request`
- `Response`
- `Route`
- `RouteResolver`
- `Pager`

 The exact implementation of the lower-level request, response, routing, and view functionality comes from Coretex.

 ## 5\. Request Creation

 The Coretex `Request` object represents the incoming HTTP request.

 It provides access to information such as:

 - HTTP method
- GET parameters
- POST parameters
- request parameters
- cookies
- uploaded files
- server variables
- HTTP headers
- request attributes

 The kernel uses the current PHP request environment to create the request object.

 ## 6\. Response Creation

 The kernel also creates a Coretex `Response` object.

 The response represents the data that will eventually be sent back to the client.

 It can contain:

 - HTTP status code
- headers
- response body

 For example:

```
$response->html("<h1>Hello</h1>");
```

 or:

```
$response->json([
    "message" => "Hello"
]);
```

 ## 7\. Route Registration

 The kernel loads:

```
bundle/Routes.php
```

 This file contains the application's route definitions.

 For example:

```
$route->get("/", "home.temp.php");

$route->get("/users", [
    UserController::class,
    "index"
]);
```

 The routes are registered with the router before the current request is resolved.

 ## 8\. Route Matching

 The router compares the incoming request against the registered routes.

 A route contains at least:

 - an HTTP method
- a URL pattern
- a handler

 For example:

```
$route->get("/users/{id}", [
    UserController::class,
    "show"
]);
```

 A request such as:

```
GET /users/42
```

 can match the route.

 The router extracts dynamic route values and makes them available to the request processing layer.

 For the example above, the dynamic parameters are conceptually:

```
[
    "id" => "42"
]
```

 ## 9\. Route Information

 OwnWork stores routing information in request attributes.

 For example:

```
$currentRoute = $request->getAttribute("currentRoute");
```

 Dynamic route parameters can be accessed through:

```
$params = $request->getAttribute("dynamicParams");
```

 A controller can therefore access route parameters through the request object without directly interacting with the router.

 ## 10\. Middleware Pipeline

 After route matching, middleware can be executed before the final route handler.

 A middleware follows the general structure:

```
public static function handle(
    Request $request,
    Response $response,
    callable $next
) {
    return $next();
}
```

 Middleware can inspect the request:

```
if (!$request->has("token")) {
    // terminate the request
}
```

 or continue execution:

```
return $next();
```

 Conceptually:

```
Request
   │
   ▼
Middleware A
   │
   ▼
Middleware B
   │
   ▼
Route Handler
```

 If middleware returns a response without calling `$next()`, downstream handlers are not executed through that middleware path.

 ## 11\. Handler Resolution

 Once the middleware pipeline reaches the route handler, Coretex resolves the registered handler.

 OwnWork supports handlers such as closures, view names, and controller actions.

 ### Closure

```
$route->get("/hello", function () {
    return "Hello";
});
```

 ### View

```
$route->get("/", "home.temp.php");
```

 ### Controller

```
$route->get("/users", [
    UserController::class,
    "index"
]);
```

 The route resolver determines how each handler should be invoked.

 ## 12\. Controller Execution

 For a controller route:

```
$route->get("/users", [
    UserController::class,
    "index"
]);
```

 the resolver invokes the corresponding controller action.

 A typical action receives:

```
public function index(
    Request $request,
    Response $response
) {
    // ...
}
```

 The controller can then:

 - read request data
- access route parameters
- call services
- work with application models
- render a view
- create a response

 ## 13\. View Rendering

 A controller can render a view using the global `view()` helper:

```
return view("users.temp.php");
```

 Data can be passed to the view:

```
return view("users.temp.php", [
    "title" => "Users"
]);
```

 For `.temp.php` files, the template system processes the source template and executes the compiled PHP representation.

 Compiled view output is stored under:

```
storage/views/
```

 ## 14\. Response Generation

 A route handler can produce a response in several ways.

 For example, a controller can return a JSON response:

```
return $response->json([
    "message" => "Users"
]);
```

 Or an HTML response:

```
return $response->html(
    "<h1>Users</h1>"
);
```

 A rendered view can also become the response body.

 ## 15\. Response Dispatch

 Once request processing produces a `Response`, the kernel dispatches it.

 Dispatching sends the response's:

 - HTTP status
- headers
- body

 to PHP's HTTP output.

 Conceptually:

```
Response
   │
   ├── Status
   ├── Headers
   └── Body
        │
        ▼
    HTTP Output
```

 ## 16\. Error Handling

 Errors can occur during any stage of request processing.

 OwnWork integrates Coretex error handling and catches framework-specific exceptions in the kernel.

 Examples include:

```
PageNotFoundException
ViewNotFoundException
```

 When no matching page exists, the application produces a `404` response.

 When a requested view cannot be found, the framework handles the view exception and renders the corresponding error response.

 The global error handler can also handle PHP errors and uncaught exceptions.

 ## Complete Example

 Consider this route:

```
$route->get("/users/{id}", [
    UserController::class,
    "show"
]);
```

 and a request:

```
GET /users/42
```

 The lifecycle is:

```
GET /users/42
      │
      ▼
public/index.php
      │
      ▼
Bundler
      │
      ▼
Kernel
      │
      ▼
Routes.php
      │
      ▼
Match /users/{id}
      │
      ▼
dynamicParams = [
    "id" => "42"
]
      │
      ▼
Middleware
      │
      ▼
UserController::show()
      │
      ▼
Response / View
      │
      ▼
HTTP response
```

 ## Where Each Responsibility Lives

 | Stage | Main location |
| --- | --- |
| HTTP entry point | `public/index.php` |
| Application bootstrap | `bundle/Bundler.php` |
| Route definitions | `bundle/Routes.php` |
| Request orchestration | `app/Http/Kernel.php` |
| Middleware | `app/Middleware/` |
| Controllers | `app/Controller/` |
| Models | `app/Model/` |
| Services | `app/Service/` |
| Application views | `resources/views/` |
| Compiled views | `storage/views/` |
| HTTP/router primitives | Coretex |

## Framework Boundary

 The request lifecycle crosses the boundary between OwnWork and Coretex.

```
                 OwnWork
┌──────────────────────────────────────┐
│ public/index.php                     │
│        ↓                             │
│ Bundler                              │
│        ↓                             │
│ Kernel                               │
│        ↓                             │
│ Application routes/middleware        │
└───────────────┬──────────────────────┘
                │
                ▼
              Coretex
┌──────────────────────────────────────┐
│ Request                              │
│ Response                             │
│ Router                               │
│ RouteResolver                        │
│ View                                 │
│ Template                             │
│ Environment                          │
│ Error handling                       │
└──────────────────────────────────────┘
```

 This separation is important when investigating framework behavior: some APIs documented by OwnWork are implemented directly in the OwnWork repository, while others are provided by the Coretex dependency.

> next: `routing/routes.md`
