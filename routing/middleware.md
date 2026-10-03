# Middleware

Middleware allows application code to run during request processing before the final route handler is executed.

OwnWork uses the middleware facilities provided by Coretex.

Middleware can inspect the request, modify processing, stop the request, or pass execution to the next middleware or route handler.

## Middleware Pipeline

A request with middleware follows this general flow:

```text
Request
   │
   ▼
Global Middleware
   │
   ▼
Route Middleware
   │
   ▼
Route Handler
   │
   ▼
Response
````

 When multiple middleware are registered, they form a chain:

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
Middleware C
   │
   ▼
Controller
   │
   ▼
Response
```

 A middleware continues the chain by calling `$next()`.

 ## Middleware Signature

 A middleware handler receives the request, response, and continuation callback:

```
function (
    Request $request,
    Response $response,
    callable $next
) {
    return $next();
}
```

 The three arguments represent:

 | Argument | Purpose |
| --- | --- |
| `$request` | Current HTTP request |
| `$response` | Current HTTP response |
| `$next` | Continue to the next middleware/handler |

## Creating Middleware

 Use the OwnWork worker:

```
php worker make middleware AuthMiddleware
```

 The generated middleware is placed under:

```
app/Middleware/
```

 For example:

```
app/Middleware/AuthMiddleware.php
```

 The generated class belongs to the `App\Middleware` namespace.

 ## Basic Middleware

 A middleware can simply continue the request:

```
<?php

namespace App\Middleware;

use Dhruv125\Coretex\Support\Request;
use Dhruv125\Coretex\Support\Response;

class AuthMiddleware
{
    public static function handle(
        Request $request,
        Response $response,
        callable $next
    ) {
        return $next();
    }
}
```

 Calling:

```
return $next();
```

 passes control to the next item in the middleware pipeline.

 ## Middleware Before the Handler

 Code before `$next()` executes before the route handler:

```
public static function handle(
    Request $request,
    Response $response,
    callable $next
) {
    // Runs before the route handler.

    return $next();
}
```

 This is useful for request checks, authentication, logging, and other pre-processing operations.

 ## Middleware After the Handler

 Code can also run after `$next()` returns:

```
public static function handle(
    Request $request,
    Response $response,
    callable $next
) {
    $result = $next();

    // Runs after downstream processing.

    return $result;
}
```

 This creates a wrapping behavior around the downstream request:

```
Middleware
    │
    ├── Before logic
    │
    ▼
  $next()
    │
    ├── Next middleware
    │
    └── Route handler
    │
    ▼
  Return
    │
    └── After logic
```

 ## Stopping the Pipeline

 A middleware does not have to call `$next()`.

 It can return a response directly:

```
public static function handle(
    Request $request,
    Response $response,
    callable $next
) {
    if (!$request->has("token")) {
        return $response->json([
            "message" => "Unauthorized"
        ]);
    }

    return $next();
}
```

 When the middleware returns without calling `$next()`, downstream processing is skipped.

 This is commonly used for access-control checks.

 ## Global Middleware

 Global middleware applies to requests across the application.

 Register it through the route object:

```
$route->globalMiddleware(
    [AuthMiddleware::class, "handle"]
);
```

 A closure can also be registered:

```
$route->globalMiddleware(
    function ($request, $response, $next) {
        return $next();
    }
);
```

 Global middleware is stored separately from route-specific middleware.

 ## Global Middleware Parameters

 The global middleware API accepts an optional parameter array:

```
$route->globalMiddleware(
    [AuthMiddleware::class, "handle"],
    [
        "role" => "admin"
    ]
);
```

 The middleware definition and its parameters are retained together.

 This allows middleware configuration to be supplied when it is registered.

 ## Route Middleware

 Middleware can also be attached to individual routes.

 For example:

```
$route->middleware(
    "GET",
    "/users",
    [AuthMiddleware::class, "handle"]
);
```

 The middleware is attached to the specified route.

 The route itself remains responsible for defining its handler:

```
$route->get("/users", [
    UserController::class,
    "index"
]);
```

 ## Applying Middleware to Multiple Routes

 The route middleware API accepts an array of URLs:

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

 The middleware is added to each matching route definition.

 ## HTTP Method Matters

 When attaching middleware to a route, specify the HTTP method:

```
$route->middleware(
    "GET",
    "/users",
    [AuthMiddleware::class, "handle"]
);
```

 A route registered for another HTTP method is a separate route.

 For example:

```
$route->get("/users", [
    UserController::class,
    "index"
]);

$route->post("/users", [
    UserController::class,
    "store"
]);
```

 Middleware registration is associated with the method and URL combination.

 ## Middleware with Dynamic Routes

 Middleware can be attached to routes containing dynamic parameters:

```
$route->get("/users/{id}", [
    UserController::class,
    "show"
]);

$route->middleware(
    "GET",
    "/users/{id}",
    [AuthMiddleware::class, "handle"]
);
```

 The middleware is associated with the route pattern:

```
/users/{id}
```

 The router subsequently resolves the actual request URL and extracts its dynamic parameters.

 ## Middleware Parameters

 Middleware definitions can contain parameters.

 For example:

```
$route->globalMiddleware(
    [AuthMiddleware::class, "handle"],
    [
        "role" => "admin"
    ]
);
```

 The parameter data is stored with the middleware registration.

 The middleware implementation can use the data according to its own application logic.

 ## Middleware Order

 Middleware is executed in the order in which it appears in the middleware pipeline.

 For example:

```
Middleware A
    ↓
Middleware B
    ↓
Middleware C
    ↓
Controller
```

 The request enters `A` first.

 If each middleware calls `$next()`, execution continues toward the controller.

 When control returns, the after-handler portions execute in reverse order:

```
A before
  B before
    C before
      Controller
    C after
  B after
A after
```

 This is the standard wrapping behavior produced by nested continuation callbacks.

 ## Authentication Example

 A simple authentication middleware might check a request value before allowing the request to continue:

```
<?php

namespace App\Middleware;

use Dhruv125\Coretex\Support\Request;
use Dhruv125\Coretex\Support\Response;

class AuthMiddleware
{
    public static function handle(
        Request $request,
        Response $response,
        callable $next
    ) {
        $token = $request->get("token");

        if (!$token) {
            return $response->json([
                "message" => "Unauthorized"
            ]);
        }

        return $next();
    }
}
```

 The exact authentication mechanism is application-specific. OwnWork's middleware layer does not automatically provide an authentication system.

 ## Logging Example

 Middleware can also be used for logging:

```
class LoggingMiddleware
{
    public static function handle(
        Request $request,
        Response $response,
        callable $next
    ) {
        // Log information about the request.

        $result = $next();

        // Log information about the completed request.

        return $result;
    }
}
```

 This allows request-related behavior to remain outside individual controllers.

 ## Middleware and Controllers

 Middleware should generally handle behavior that applies around request execution rather than application-specific controller operations.

 For example:

```
Middleware
├── Authentication
├── Authorization
├── Logging
└── Request checks

Controller
├── Application operation
├── Service calls
└── View/response creation
```

 The exact division is an application design decision.

 ## Inspecting Registered Middleware

 The router exposes globally registered middleware through:

```
$route->getGlobalMiddleware();
```

 This returns the middleware definitions registered with `globalMiddleware()`.

 Route-specific middleware is stored as part of the individual route definition.

 ## Middleware in the Request Lifecycle

 Middleware sits between route resolution and final handler execution.

 For example:

```
GET /users/42
      │
      ▼
Route matching
      │
      ▼
/users/{id}
      │
      ▼
Extract dynamicParams
      │
      ▼
Global middleware
      │
      ▼
Route middleware
      │
      ▼
UserController::show()
      │
      ▼
Response
```

 Dynamic route parameters are resolved before the matched route is processed by the request pipeline.

 ## Middleware Checklist

 When creating middleware:

- Put application middleware under `app/Middleware/`.
- Provide a callable handler.
- Accept the request, response, and `$next` continuation.
- Call `$next()` when the request should continue.
- Return a response directly when the request should be terminated.
- Register application-wide middleware with `globalMiddleware()`.
- Register route-specific middleware with `middleware()`.
- Keep authentication, authorization, logging, and similar cross-cutting concerns outside controllers where appropriate.

> next: `application/controllers.md`
