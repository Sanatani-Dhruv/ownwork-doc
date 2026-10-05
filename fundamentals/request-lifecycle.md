# Request Lifecycle

OwnWork processes an HTTP request through application bootstrap, route registration, route matching, middleware execution, handler resolution, and response dispatch.

Some of the HTTP and routing primitives used during this process are provided by the `dhruv125/coretex` dependency.

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
     ├── Check .env and Composer autoloader
     ├── Load Composer
     ├── Load environment
     └── Configure global error handler
     │
     ▼
Kernel
     │
     ├── Create Request
     ├── Create Response
     ├── Create Route
     ├── Create RouteResolver
     └── Create Pager
     │
     ▼
bundle/Routes.php
     │
     ▼
Route Matching
     │
     ├── Handler
     ├── Dynamic parameters
     ├── Route information
     └── Middleware
     │
     ▼
Middleware Pipeline
     │
     ▼
RouteResolver
     │
     ├── Closure
     ├── View
     └── Controller
     │
     ▼
Response / String
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

 The front controller does not define routes or process individual requests.

 Its responsibility is to start the OwnWork application.

 ## 3\. Application Bootstrap

 The bundler is located at:

```
bundle/Bundler.php
```

 Before the kernel is started, the bundler checks that both of the following exist:

```
.env
vendor/autoload.php
```

 If either is missing, OwnWork stops and displays a setup message instructing the developer to run:

```
composer run setup
```

 When the required files exist, the bundler loads Composer:

```
vendor/autoload.php
```

 It then creates Coretex's environment handler and loads the application's environment configuration.

 The environment loader returns an error level which is passed to Coretex's global error handler.

 Conceptually:

```
Bundler
   │
   ├── Check setup
   ├── Composer autoloader
   ├── Environment
   ├── Global error handler
   └── Kernel
```

 The relevant bootstrap code is contained in:

```
bundle/Bundler.php
```

 ## 4\. Kernel Initialization

 The HTTP kernel is located at:

```
app/Http/Kernel.php
```

 When the kernel is constructed, OwnWork creates:

 - Coretex `Request`
- Coretex `Response`
- Coretex `Route`
- Coretex `RouteResolver`
- Coretex `Pager`

 Conceptually:

```
Kernel
   │
   ├── Request
   ├── Response
   ├── Route
   ├── RouteResolver
   └── Pager
```

 The kernel is responsible for coordinating the application's request processing.

 ## 5\. Route Registration

 When `Kernel::handle()` runs, it loads:

```
bundle/Routes.php
```

 For example:

```php
<?php

$route->get("/", "main.temp.php");

$route->get("/id/{id}/{name}", [
    UserController::class,
    "index"
]);
```

 The `$route` object is created by the kernel and is available while the routes file is loaded.

 The routes are therefore registered before the current request is resolved.

 ## 6\. Route Matching

 After loading the route definitions, the kernel calls the route object's matching process.

 The router determines the matching handler and collects information about the current request.

 For example, a route such as:

```
/id/{id}/{name}
```

 can match:

```
/id/42/dhruv
```

 The resulting dynamic parameters are stored by the kernel.

 Conceptually:

```
{
    "id": "42",
    "name": "dhruv"
}
```

 If no handler is found, the kernel throws a Coretex `PageNotFoundException`.

 The kernel catches that exception and returns the framework's not-found page with HTTP status:

```
404
```

 ## 7\. Request Attributes

 OwnWork stores routing information on the Coretex `Request` object.

 The kernel sets these attributes:

```
currentRoute
routesArray
dynamicParams
```

 For example:

```php
<?php

$currentRoute = $request->getAttribute("currentRoute");

$params = $request->getAttribute("dynamicParams");
```

 Dynamic route parameters therefore become available to application code through the request object.

 ## 8\. Middleware Pipeline

 After route matching, OwnWork executes the middleware associated with the matched route.

 Global middleware registered on the route object is also added to the middleware pipeline.

 The kernel executes middleware recursively.

 Conceptually:

```
Request
   │
   ▼
Global Middleware
   │
   ▼
Route Middleware
   │
   ▼
Next Middleware
   │
   ▼
Route Handler
```

 A middleware receives the request, response, and a callback used to continue execution.

 For example:

```php
<?php

function (
    Request $request,
    Response $response,
    callable $next
) {
    return $next();
}
```

 A middleware can terminate the request instead of calling `$next()`.

 For example:

```php
<?php

if (!$request->has("token")) {
    return "Unauthorized";
}

return $next();
```

 If `$next()` is called, execution continues through the remaining middleware until the route handler is reached.

 ## 9\. Route Handler Resolution

 After middleware completes, the kernel passes the route handler to Coretex's `RouteResolver`.

 OwnWork supports handler forms such as:

 ### Closure

```php
<?php

$route->get("/hello", function () {
    return "Hello";
});
```

 ### View

```php
<?php

$route->get("/", "main.temp.php");
```

 ### Controller

```php
<?php

$route->get("/users", [
    UserController::class,
    "index"
]);
```

 The `RouteResolver` determines how the registered handler should be executed.

 ## 10\. Controller Execution

 For a controller route:

```php
<?php

$route->get("/users", [
    UserController::class,
    "index"
]);
```

 the resolver invokes the specified controller action.

 A controller action commonly receives Coretex request and response objects:

```php
<?php

public function index(
    Request $request,
    Response $response
) {
    // ...
}
```

 The controller can then read request information, access route parameters, call application services, work with models, render views, or return a response.

 ## 11\. View Resolution

 A route can directly resolve to a view name:

```php
<?php

$route->get("/", "main.temp.php");
```

 A controller can also use the global `view()` helper:

```php
<?php

return view("users.temp.php");
```

 Data can be supplied to the view:

```php
<?php

return view("users.temp.php", [
    "title" => "Users"
]);
```

 For `.temp.php` templates, the Coretex template system handles compilation and the generated view files are stored under:

```
storage/views/
```

 The mapping between source templates and compiled views is stored in:

```
storage/views.json
```

 Template compilation itself is performed by the OwnWork/Coretex development tooling rather than during every normal request.

 ## 12\. Handler Result

 After the route resolver executes the handler, the kernel receives its result.

 The current kernel explicitly handles two important result types.

 A string is placed into the response body:

```php
<?php

return "Hello from OwnWork";
```

 A Coretex `Response` object is dispatched directly.

 For example:

```php
<?php

return $response->json([
    "message" => "Hello"
]);
```

 The exact response helpers are provided by Coretex.

 ## 13\. Response Dispatch

 When a route handler returns a string, the kernel places that string into its response and dispatches it.

 When a handler returns a Coretex `Response`, the kernel dispatches that response directly.

 Conceptually:

```
Route Handler
     │
     ▼
Result
     │
     ├── String
     │     ↓
     │   Response body
     │
     └── Response
           ↓
       Response dispatch
```

 The response is then sent to the client.

 ## 14\. Error Handling

 OwnWork configures Coretex's global error handler during application bootstrap.

 The handler is created by:

```
bundle/Bundler.php
```

 using the environment-derived error level.

 The kernel additionally handles specific framework exceptions.

 A missing route results in:

```
PageNotFoundException
```

 which is converted to:

```
HTTP 404
```

 and displayed through the Coretex `Pager`.

 A missing view results in:

```
ViewNotFoundException
```

 which is converted by the kernel into:

```
HTTP 500
```

 and displayed using the framework's view-not-found error page.

 The framework's error pages are located in:

```
resources/appviews/
```

 ## Complete Example

 Consider this route:

```php
<?php

$route->get("/id/{id}/{name}", [
    UserController::class,
    "index"
]);
```

 and a request:

```
GET /id/42/dhruv
```

 The lifecycle is approximately:

```
GET /id/42/dhruv
      │
      ▼
public/index.php
      │
      ▼
Bundler
      │
      ├── Load environment
      └── Configure error handler
      │
      ▼
Kernel
      │
      ├── Request
      ├── Response
      ├── Route
      └── RouteResolver
      │
      ▼
bundle/Routes.php
      │
      ▼
Route matching
      │
      ▼
dynamicParams
      │
      ├── id = 42
      └── name = dhruv
      │
      ▼
Middleware
      │
      ▼
RouteResolver
      │
      ▼
UserController::index()
      │
      ▼
Response / String
      │
      ▼
HTTP output
```

 ## Where Each Responsibility Lives

 | Stage | Main location |
| --- | --- |
| HTTP entry point | `public/index.php` |
| Application bootstrap | `bundle/Bundler.php` |
| Route definitions | `bundle/Routes.php` |
| Request orchestration | `app/Http/Kernel.php` |
| Middleware registration/execution | `app/Http/Kernel.php` and route configuration |
| Controllers | `app/Controller/` |
| Models | `app/Model/` |
| Services | `app/Service/` |
| Application views | `resources/views/` |
| Compiled views | `storage/views/` |
| Framework error pages | `resources/appviews/` |
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
│ Route                                │
│ RouteResolver                        │
│ Pager                                │
│ Environment                          │
│ GlobalErrorHandler                   │
│ View / Template system               │
└──────────────────────────────────────┘
```

 This distinction is important when reading the source or debugging behavior.

 OwnWork's `Bundler` and `Kernel` coordinate the application, while several lower-level HTTP, routing, view, environment, and error-handling operations are implemented by Coretex.

 > next: `routing/routes.md`
