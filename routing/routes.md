# Routes

OwnWork uses the `Route` class provided by Coretex for registering and matching HTTP routes.

Application routes are defined in:

```text
bundle/Routes.php
````

 The `$route` object is available when `bundle/Routes.php` is loaded by OwnWork's HTTP kernel.

 ## Basic Route

 The simplest route maps a URL to a handler:

```php
<?php

$route->get("/", "main.temp.php");
```

 This registers a `GET` route for `/`.

 The second argument is the route handler.

 ## HTTP Methods

 Coretex's route class provides methods for:

 - `GET`
- `POST`
- `PUT`
- `PATCH`
- `DELETE`

 ### GET

```php
<?php

$route->get("/users", "users.temp.php");
```

 ### POST

```php
<?php

$route->post("/users", [
    UserController::class,
    "store"
]);
```

 ### PUT

```php
<?php

$route->put("/users/{id}", [
    UserController::class,
    "update"
]);
```

 ### PATCH

```php
<?php

$route->patch("/users/{id}", [
    UserController::class,
    "update"
]);
```

 ### DELETE

```php
<?php

$route->delete("/users/{id}", [
    UserController::class,
    "destroy"
]);
```

 The route methods accept a URL and a handler.

 The HTTP method determines which route collection is used during matching.

 ## View Routes

 A string handler can be used as a view route:

```php
<?php

$route->get("/", "main.temp.php");
```

 When the route is resolved, Coretex's route resolver treats the string handler as a view.

 This is useful for simple pages that do not require a controller.

 For example:

```php
<?php

$route->get("/about", "about.temp.php");

$route->get("/contact", "contact.temp.php");
```

 The corresponding templates are located under:

```
resources/views/
```

 For example:

```
resources/views/about.temp.php
resources/views/contact.temp.php
```

 ## Closure Routes

 A route can use a callable:

```php
<?php

$route->get("/hello", function () {
    return "Hello World";
});
```

 The callable becomes the route handler.

 This is useful for small handlers where creating a separate controller would add unnecessary structure.

 ## Controller Routes

 Controller actions can be registered using a two-element array:

```php
<?php

use App\Controller\UserController;

$route->get("/users", [
    UserController::class,
    "index"
]);
```

 The first element is the controller class.

 The second element is the method that should be executed.

 The same form can be used with other HTTP methods:

```php
<?php

$route->post("/users", [
    UserController::class,
    "store"
]);
```

 ## Dynamic URLs

 Route URLs can contain dynamic parameters:

```php
<?php

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

```php
<?php

$route->get("/users/{id}/{name}", [
    UserController::class,
    "show"
]);
```

 For:

```
/users/42/dhruv
```

 the matched parameters are conceptually:

```
[
    "id" => "42",
    "name" => "dhruv"
]
```

 The parameters are exposed to the application through the request attributes created during request handling.

 See `routing/dynamic-parameters.md` for details.

 ## Route Matching

 The router keeps separate route collections for the supported HTTP methods.

 When the request is processed, Coretex matches the request method against the corresponding collection and then checks the registered URL patterns.

 A static route such as:

```php
<?php

$route->get("/users", "users.temp.php");
```

 matches:

```
/users
```

 but does not match:

```
/users/42
```

 Dynamic parameters are represented using `{parameter}` syntax.

 For example:

```php
<?php

$route->get("/users/{id}", "user.temp.php");
```

 The `{id}` portion is converted internally into a regular-expression component used to match the corresponding URL segment.

 The generated expression is anchored so that the request URL must match the complete route pattern.

 ## Route Registration Order

 Routes are stored in the order in which they are registered.

 During matching, Coretex checks the registered routes for the current HTTP method in that order and stops when a matching route is found.

 Therefore, overlapping routes can be affected by registration order.

 For example:

```php
<?php

$route->get("/users/{id}", "user.temp.php");

$route->get("/users/admin", "admin.temp.php");
```

 Because the dynamic route is registered first, `/users/admin` can be matched by the dynamic route before the later static route is reached.

 When routes overlap, register the intended specific route before a broader dynamic route.

 ## Current Route

 When a route is matched, OwnWork stores the matched route information on the request.

 For:

```php
<?php

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

 Application code can access this through the request attributes:

```php
<?php

$currentRoute = $request->getAttribute("currentRoute");
```

 ## Route Parameters

 Dynamic parameter values are stored separately from the route pattern.

 For:

```php
<?php

$route->get("/users/{id}", [
    UserController::class,
    "show"
]);
```

 and:

```
/users/42
```

 the matching information contains the equivalent of:

```
[
    "params" => [
        "id" => "42"
    ],
    "currentRoute" => "/users/{id}"
]
```

 OwnWork places the dynamic parameters into the request attributes:

```php
<?php

$params = $request->getAttribute("dynamicParams");
```

 This allows controllers and middleware to access route parameters without directly interacting with the router.

 ## Route Middleware

 Each registered route starts with its own middleware collection.

 Conceptually, a route is stored with:

```
[
    "handler" => ...,
    "middlewares" => []
]
```

 Middleware can subsequently be associated with the route through the router's middleware API.

 See `routing/middleware.md` for the middleware API and execution behavior.

 ## Global Middleware

 Coretex also allows middleware to be registered globally:

```php
<?php

$route->globalMiddleware(
    [AuthMiddleware::class, "handle"]
);
```

 A callable can also be registered:

```php
<?php

$route->globalMiddleware(
    function ($request, $response, $next) {
        return $next();
    }
);
```

 Global middleware is stored separately from individual route middleware and is made available to OwnWork's kernel during request processing.

 ## Middleware Parameters

 Global middleware can receive an optional parameters array:

```php
<?php

$route->globalMiddleware(
    [AuthMiddleware::class, "handle"],
    [
        "role" => "admin"
    ]
);
```

 The parameters are stored together with the middleware definition.

 ## Multiple Routes for Middleware

 The `middleware()` API can target either a single route URL or multiple URLs.

 For example:

```php
<?php

$route->middleware(
    "GET",
    "/users",
    [AuthMiddleware::class, "handle"]
);
```

 Multiple URLs can be supplied:

```php
<?php

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

 The router exposes registered route URLs through:

```php
<?php

$routes = $route->getAllRoutes();
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

 This listing contains the registered route URLs rather than the complete handler and middleware definitions.

 ## Completing Route Matching

 The router's `end()` method performs matching against the current request:

```php
<?php

$result = $route->end();
```

 A successful match contains routing information including:

```
[
    "middlewares" => [...],
    "handler" => ...,
    "params" => [...],
    "currentRoute" => "...",
    "routesArray" => [...]
]
```

 OwnWork's kernel uses this result to continue request processing.

 If no route matches, the kernel handles the resulting `PageNotFoundException` and produces the application's 404 response.

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
| `getGlobalMiddleware()` | Retrieve registered global middleware |
| `getAllRoutes()` | Retrieve registered route URLs |
| `end()` | Match the current request |

The underlying router is implemented by Coretex's `Dhruv125\Coretex\Router\Route` class.

 > next: `routing/dynamic-parameters.md`
