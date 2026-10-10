# Routing API

 OwnWork routing is provided through Coretex's `Route` class and is configured from:

```
bundle/Routes.php
```

 The application's `Kernel` creates the route object and loads `bundle/Routes.php` before resolving the matching handler.

 ## Route Definition

 Routes are registered on the `$route` instance:

```php
$route->get("/", "main.temp.php");
```

 A controller action can also be used:

```php
$route->get("/users", [
    UserController::class,
    "index"
]);
```

 The second argument is the route handler.

 ## GET Routes

 Register a GET route with:

```php
$route->get("/users", [
    UserController::class,
    "index"
]);
```

 GET routes can also directly resolve to a view:

```php
$route->get("/", "main.temp.php");
```

 ## POST Routes

 Register a POST route with:

```php
$route->post("/users", [
    UserController::class,
    "store"
]);
```

 ## PUT Routes

```php
$route->put("/users/{id}", [
    UserController::class,
    "update"
]);
```

 ## PATCH Routes

```php
$route->patch("/users/{id}", [
    UserController::class,
    "update"
]);
```

 ## DELETE Routes

```php
$route->delete("/users/{id}", [
    UserController::class,
    "destroy"
]);
```

 ## Route Paths

 Route paths begin with `/`:

```php
$route->get("/about", [
    PageController::class,
    "about"
]);
```

 The route is matched against the incoming request path.

 ## Dynamic Parameters

 Dynamic parameters use `{name}` syntax:

```php
$route->get("/users/{id}", [
    UserController::class,
    "show"
]);
```

 A request such as:

```
/users/12
```

 provides the dynamic parameter:

```
id = 12
```

 OwnWork stores the matched parameters on the request as:

```
$request->getAttribute("dynamicParams");
```

 For example:

```php
$params = $request->getAttribute(
    "dynamicParams"
);

$id = $params["id"];
```

 See `routing/dynamic-parameters.md`.

 ## Multiple Dynamic Parameters

 A route can contain multiple parameters:

```php
$route->get("/users/{id}/{name}", [
    UserController::class,
    "show"
]);
```

 For example:

```
/users/12/someone
```

 produces parameters equivalent to:

```php
[
    "id" => "12",
    "name" => "someone"
]
```

 ## Controller Actions

 Controller handlers are normally represented as:

```php
[
    UserController::class,
    "index"
]
```

 For example:

```php
$route->get("/users", [
    UserController::class,
    "index"
]);
```

 The route resolver receives the handler and invokes the controller action with the application's request and response objects.

 A controller can therefore use:

```php
public function index(
    Request $request,
    Response $response
) {
    return view("users/index.temp.php");
}
```

 ## Direct View Handlers

 A route can use a view name as its handler:

```
$route->get("/", "main.temp.php");
```

 This is useful for simple routes that do not require controller logic.

 ## Middleware

 Routes can carry middleware handlers.

 The route result contains middleware information, and OwnWork's `Kernel` executes the middleware chain before resolving the final route handler.

 A route can therefore be used with middleware where request processing must happen before the controller or other handler.

 See `routing/middleware.md`.

 ## Route Resolution

 When a request is handled, OwnWork:

```
Request
    ↓
Route registration
    ↓
Route matching
    ↓
Dynamic parameters
    ↓
Middleware
    ↓
Route handler
    ↓
Response
```

 The `Kernel` calls:

```
$result = $route->end();
```

 to obtain the matched route information.

 The resulting information includes the route handler and matched parameters.

 ## Route Request Attributes

 OwnWork places routing information on the request.

 The current route is available as:

```php
$request->getAttribute("currentRoute");
```

 The registered route information is available as:

```php
$request->getAttribute("routesArray");
```

 Dynamic parameters are available as:

```php
$request->getAttribute("dynamicParams");
```

 These attributes are populated by the application kernel after route matching.

 ## Missing Routes

 If no route handler is available after matching, OwnWork throws:

```
PageNotFoundException
```

 The kernel catches this exception and produces a 404 response through the framework pager.

 Conceptually:

```
No matching handler
        ↓
PageNotFoundException
        ↓
HTTP 404
        ↓
OwnWork not-found page
```

 ## Route File

 The application's route definitions belong in:

```
bundle/Routes.php
```

 A typical route file can contain:

```php
<?php

namespace Bundle;

use App\Controller\UserController;

$route->get("/", "main.temp.php");

$route->get("/users", [
    UserController::class,
    "index"
]);

$route->get("/users/{id}", [
    UserController::class,
    "show"
]);

$route->post("/users", [
    UserController::class,
    "store"
]);
```

 The file is loaded by `App\Http\Kernel` during request handling.

 ## Route API Summary

 | API | Purpose |
| --- | --- |
| `$route->get()` | Register a GET route |
| `$route->post()` | Register a POST route |
| `$route->put()` | Register a PUT route |
| `$route->patch()` | Register a PATCH route |
| `$route->delete()` | Register a DELETE route |
| `$route->end()` | Resolve registered routes for the current request |

## Example

```php
$route->get("/", "main.temp.php");

$route->get("/users", [
    UserController::class,
    "index"
]);

$route->get("/users/{id}", [
    UserController::class,
    "show"
]);

$route->post("/users", [
    UserController::class,
    "store"
]);

$route->put("/users/{id}", [
    UserController::class,
    "update"
]);

$route->patch("/users/{id}", [
    UserController::class,
    "update"
]);

$route->delete("/users/{id}", [
    UserController::class,
    "destroy"
]);
```

 >  `reference/request-api.md`
