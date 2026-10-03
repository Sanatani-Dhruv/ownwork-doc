# Routes

OwnWork uses the `Route` class provided by Coretex for HTTP route registration and matching.

Application routes are defined in:

```text
bundle/Routes.php
````

 The route object is available in that file as `$route`.

 ## Basic Route

 The simplest route maps a URL to a handler:

```
$route->get("/", "main.temp.php");
```

 The route above registers a `GET` request for `/`.

 The second argument is the route handler.

 ## HTTP Methods

 The router provides methods for the following HTTP methods:

 - `GET`
- `POST`
- `PUT`
- `PATCH`
- `DELETE`

 ### GET

```
$route->get("/users", "users.temp.php");
```

 ### POST

```
$route->post("/users", [
    UserController::class,
    "store"
]);
```

 ### PUT

```
$route->put("/users/{id}", [
    UserController::class,
    "update"
]);
```

 ### PATCH

```
$route->patch("/users/{id}", [
    UserController::class,
    "update"
]);
```

 ### DELETE

```
$route->delete("/users/{id}", [
    UserController::class,
    "destroy"
]);
```

 All five methods accept the same basic arguments:

```
$route->get(
    string $url,
    callable|array|string $handler
);
```

 The HTTP method determines which route table the route is registered in.

 ## View Routes

 A string handler can be used as a view route:

```
$route->get("/", "main.temp.php");
```

 When the route is matched, the route resolver treats the string as a view handler.

 This is useful for pages that do not require a controller.

 For example:

```
$route->get("/about", "about.temp.php");

$route->get("/contact", "contact.temp.php");
```

 ## Closure Routes

 A route can use a callable:

```
$route->get("/hello", function () {
    return "Hello World";
});
```

 The callable becomes the route handler.

 This is useful for small handlers where a separate controller is unnecessary.

 ## Controller Routes

 Controller actions can be registered using a two-element array:

```
use App\Controller\UserController;

$route->get("/users", [
    UserController::class,
    "index"
]);
```

 The first element is the controller class.

 The second element is the method to execute.

 The same form works with other HTTP methods:

```
$route->post("/users", [
    UserController::class,
    "store"
]);
```

 ## Controller Method Default

 Coretex's middleware API defaults a controller-style handler to the `index` method when only the class is supplied.

 For normal route registration, use the explicit two-element form:

```
$route->get("/users", [
    UserController::class,
    "index"
]);
```

 This makes the intended action explicit.

 ## Dynamic URLs

 Route URLs can contain dynamic parameters:

```
$route->get("/users/{id}", [
    UserController::class,
    "show"
]);
```

 A request such as:

```
/users/42
```

 matches the route and produces a dynamic parameter:

```
[
    "id" => "42"
]
```

 Multiple parameters are supported:

```
$route->get("/users/{id}/{name}", [
    UserController::class,
    "show"
]);
```

 For:

```
/users/42/dhruv
```

 the router produces:

```
[
    "id" => "42",
    "name" => "dhruv"
]
```

 See Dynamic Parameters for details.

 ## Route Matching

 The router first selects the route collection corresponding to the incoming HTTP method.

 It then checks each registered URL against the current request URL.

 Static routes are matched directly.

 For example:

```
$route->get("/users", "users.temp.php");
```

 matches:

```
/users
```

 but not:

```
/users/42
```

 Dynamic route parameters use a generated regular expression.

 For:

```
$route->get("/users/{id}", "user.temp.php");
```

 the `{id}` portion is replaced internally with a word-character pattern.

 The complete generated expression is anchored to the beginning and end of the URL, so the complete path must match.

 ## Route Registration Order

 Routes are stored in the order in which they are registered.

 During matching, the router iterates through the routes for the current HTTP method and stops when it finds a matching route.

 Therefore, when multiple route patterns could match a request, registration order can affect which route is selected.

 Example:

```
$route->get("/users/{id}", "user.temp.php");
$route->get("/users/admin", "admin.temp.php");
```

 The dynamic route is registered first, so `/users/admin` can match it before the later static route is reached.

 When route patterns may overlap, place the intended specific route before a broader dynamic route.

 ## Current Route

 When a route is successfully matched, Coretex returns the matched route pattern as `currentRoute`.

 For:

```
$route->get("/users/{id}", [
    UserController::class,
    "show"
]);
```

 a request to:

```
/users/42
```

 has a current route corresponding to:

```
/users/{id}
```

 OwnWork exposes this routing information through the request attributes used during request processing.

 ## Route Parameters

 The router returns dynamic values separately from the route pattern.

 A successful match contains:

```
[
    "params" => [
        "id" => "42"
    ],
    "currentRoute" => "/users/{id}"
]
```

 OwnWork makes these parameters available through the request:

```
$params = $request->getAttribute("dynamicParams");
```

 ## Route Middleware

 Routes have a middleware collection associated with them.

 A route registered with `get()`, `post()`, `put()`, `patch()`, or `delete()` starts with an empty middleware collection:

```
[
    "handler" => $handler,
    "middlewares" => []
]
```

 Middleware can subsequently be associated with a route through the router's middleware API.

 See Middleware.

 ## Global Middleware

 Middleware can also be registered globally:

```
$route->globalMiddleware(
    [AuthMiddleware::class, "handle"]
);
```

 A callable can also be registered:

```
$route->globalMiddleware(
    function ($request, $response, $next) {
        return $next();
    }
);
```

 Global middleware is stored separately from individual route middleware and is available to the application kernel.

 ## Middleware Parameters

 Global middleware accepts an optional parameters array:

```
$route->globalMiddleware(
    [AuthMiddleware::class, "handle"],
    [
        "role" => "admin"
    ]
);
```

 The parameters are stored together with the middleware definition.

 ## Multiple Routes for Middleware

 The `middleware()` API accepts either a single URL or an array of URLs.

 For example:

```
$route->middleware(
    "GET",
    "/users",
    [AuthMiddleware::class, "handle"]
);
```

 Multiple URLs can be supplied:

```
$route->middleware(
    "GET",
    [
        "/users",
        "/profile",
        "/settings"
    ],
    [AuthMiddleware::class, "handle"]
);
```

 The middleware is added to each specified route.

 ## Listing Registered Routes

 The router exposes all registered route patterns through:

```
$route->getAllRoutes();
```

 The result is grouped by HTTP method.

 For example:

```
[
    "GET" => [
        "/",
        "/users"
    ],
    "POST" => [
        "/users"
    ],
    "PATCH" => [
        "/users/{id}"
    ],
    "PUT" => [],
    "DELETE" => []
]
```

 Only registered route URLs are returned; handler details and middleware are not included in this listing.

 ## Completing Route Matching

 The router's `end()` method performs route matching against the current request.

```
$result = $route->end();
```

 For a successful match, the result contains:

```
[
    "middlewares" => [...],
    "handler" => ...,
    "params" => [...],
    "currentRoute" => "...",
    "routesArray" => [...]
]
```

 For an unsuccessful match, the router returns the match result indicating that no route was found.

 OwnWork's kernel uses this result as part of request processing.

 ## Route File Example

 A typical `bundle/Routes.php` file can look like:

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

 This gives the application separate handlers for common HTTP operations.

 ## Router API Summary

| Method | Purpose |
| --- | --- |
| `get()` | Register a GET route |
| `post()` | Register a POST route |
| `put()` | Register a PUT route |
| `patch()` | Register a PATCH route |
| `delete()` | Register a DELETE route |
| `middleware()` | Attach middleware to routes |
| `globalMiddleware()` | Register global middleware |
| `getGlobalMiddleware()` | Retrieve global middleware |
| `getAllRoutes()` | Retrieve registered route URLs |
| `end()` | Match the current request |

The underlying router is implemented by Coretex's `Dhruv125\Coretex\Router\Route` class.

> next: `routing/dynamic-parameters.md`
