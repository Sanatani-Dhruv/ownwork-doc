 # Architecture

 OwnWork is a small PHP application framework built around an HTTP request pipeline.

 OwnWork provides the application bootstrap, HTTP kernel, route registration, middleware orchestration, application structure, helpers, view workflow, and development tooling.

 Some lower-level HTTP, routing, view, environment, and error-handling functionality is provided by the `dhruv125/coretex` package.

 The architecture is therefore best understood as two cooperating layers:

```
OwnWork
   │
   ├── Application bootstrap
   ├── HTTP Kernel
   ├── Application structure
   ├── Middleware orchestration
   ├── Helpers
   ├── Worker CLI
   └── Coretex integration
          │
          ▼
      Coretex
      ├── Request
      ├── Response
      ├── Route
      ├── RouteResolver
      ├── View
      ├── Environment
      └── Error handling
```

 ## High-Level Request Architecture

 A request moves through the application approximately as follows:

```
HTTP Request
     │
     ▼
public/index.php
     │
     ▼
Bundler
     │
     ├── Composer autoloader
     ├── Environment
     └── Global error handler
     │
     ▼
Http\Kernel
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
Route Matching
     │
     ├── Dynamic parameters
     └── Route middleware
     │
     ▼
Global Middleware
     │
     ▼
Route Middleware
     │
     ▼
RouteResolver
     │
     ├── Callable
     ├── View
     └── Controller
     │
     ▼
Response
     │
     ▼
HTTP Output
```

 The application entry point is intentionally small. `Bundler` handles application initialization, while `Kernel` coordinates request processing. The actual HTTP request, response, routing, and route resolution classes used by the kernel come from Coretex.

 ## Application Entry Point

 The public entry point is:

```
public/index.php
```

 Its current implementation is:

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

 `public/index.php` loads `bundle/Bundler.php`, creates the `Bundler`, and starts the application by calling `bundle()`.

 The output buffer is started before application execution and flushed after the application finishes.

 You normally do not need to modify `public/index.php`.

 ## Bundler

 The bundler is located at:

```
bundle/Bundler.php
```

 `Bundler` is responsible for initializing the application before the HTTP kernel starts.

 Its constructor:

 1. Checks that `.env` exists.
2. Checks that `vendor/autoload.php` exists.
3. Loads Composer's autoloader.
4. Creates Coretex's `Environment`.
5. Loads the root `.env` configuration.
6. Creates Coretex's `GlobalErrorHandler`.
7. Stores the resulting error-level configuration for the handler.

 Its `bundle()` method then creates the OwnWork `Kernel` and calls `handle()`.

 Conceptually:

```
Bundler
   │
   ├── Check .env
   ├── Check Composer autoloader
   ├── Load Composer
   ├── Load environment
   ├── Initialize error handler
   └── Start Kernel
```

 `Bundler` does not register individual routes or execute controllers itself.

 If the required `.env` or Composer autoloader is missing, `Bundler` stops the application and displays the setup error page when the corresponding error view exists.

 ## HTTP Kernel

 The application kernel is located at:

```
app/Http/Kernel.php
```

 The kernel coordinates request processing.

 It creates:

 - Coretex `Request`
- Coretex `Response`
- Coretex `Route`
- Coretex `RouteResolver`
- Coretex `Pager`

 The relevant classes are imported directly from:

```
Dhruv125\Coretex
```

 The kernel then loads:

```
bundle/Routes.php
```

 using the application's root path.

 After the routes are registered, it calls the route object's `end()` method to determine whether the current HTTP request matches a registered route.

 ## Route Registration and Matching

 Routing itself is implemented by Coretex's `Route` class.

 OwnWork creates the route object in the kernel and exposes it to:

```
bundle/Routes.php
```

 Applications register routes such as:

```php
<?php

$route->get("/", "home.temp.php");
```

 or:

```php
<?php

$route->get("/users", [
    UserController::class,
    "index"
]);
```

 Coretex currently supports these HTTP methods:

```
GET
POST
PUT
PATCH
DELETE
```

 Route matching is performed against the request URL and HTTP method. Routes are checked in their registered order for the current method.

 Dynamic parameters use `{name}` syntax:

```php
<?php

$route->get("/users/{id}", [
    UserController::class,
    "show"
]);
```

 For a request such as:

```
/users/42
```

 the route matcher produces:

```
id = 42
```

 The kernel stores the resulting parameters on the request as:

```
dynamicParams
```

 It also stores the matched route as:

```
currentRoute
```

 and the registered route list as:

```
routesArray
```

 These are request attributes available to application code.

 ## Route Resolver

 After route matching and middleware execution, the kernel passes the selected handler to Coretex's `RouteResolver`.

 The resolver supports three main handler forms.

 ### Callable Handler

 A route can use a callable:

```php
<?php

$route->get("/hello", function () {
    return "Hello from OwnWork";
});
```

 The resolver invokes the callable directly.

 When dynamic route parameters are present, the resolver uses the callable's parameter count to pass matching dynamic values to it.

 ### View Handler

 A string handler is treated as a view name:

```php
<?php

$route->get("/", "home.temp.php");
```

 The Coretex resolver passes the view name to its view system.

 ### Controller Handler

 A controller action can be registered as:

```php
<?php

$route->get("/users", [
    UserController::class,
    "index"
]);
```

 The resolver creates an instance of the controller and calls the specified method with:

```
Request
Response
Route parameters
```

 If only the controller class is supplied, the resolver falls back to an `index` method when that method exists.

 This means controller classes do not need to extend a framework-provided base controller.

 ## Middleware

 Middleware is executed between route matching and route resolution.

 OwnWork's kernel builds the middleware chain and executes it recursively.

 A middleware receives the request and response objects followed by a `$next` callable:

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

 A middleware can:

 - inspect the request
- modify processing
- return a response without continuing
- call `$next()` to continue to the next middleware

 The effective processing order is:

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
Route Handler
   │
   ▼
Response
```

 The kernel collects middleware attached to the matched route and then prepends global middleware before executing the complete chain.

 Coretex supports registering middleware against individual routes or groups of routes, as well as global middleware.

 ## Controllers

 Controllers normally live under:

```
app/Controller/
```

 A controller is an application class. OwnWork does not require controllers to extend a framework base class.

 A typical controller action receives:

```php
<?php

public function index(
    Request $request,
    Response $response
) {
    return "Hello";
}
```

 The request and response classes are provided by Coretex.

 When a route contains dynamic parameters, the route resolver also passes the parameter array as the third argument:

```php
<?php

public function show(
    Request $request,
    Response $response,
    array $params
) {
    return "User: " . $params["id"];
}
```

 Controllers are therefore responsible for application-specific request handling rather than framework configuration.

 ## Models

 Models belong under:

```
app/Model/
```

 This directory is part of OwnWork's application structure.

 OwnWork does not define a model base class or impose an ORM on model classes.

 The bundled helper contains an optional `get_db_instance()` function that can create a database instance when the optional `delight-im/db` package is installed and the corresponding `DB_*` environment variables are configured.

 Applications can also implement their own data-access layer or use another database package.

 Therefore, `app/Model/` is an application convention rather than a required OwnWork ORM layer.

 ## Services

 Services normally belong under:

```
app/Service/
```

 A service is an application-level class that can contain reusable operations or business logic.

 For example:

```
Controller
    │
    ▼
Service
    │
    ├── Model
    ├── Database
    └── External API
```

 OwnWork does not define a service base class or require a particular service interface.

 The `app/Service/` directory is an application organization convention supported by the worker's service generator.

 ## Views

 View source files normally live under:

```
resources/views/
```

 OwnWork's view workflow uses `.temp.php` files as template source.

 For example:

```
resources/views/home.temp.php
```

 The worker transpiles these templates into generated PHP files under:

```
storage/views/
```

 and maintains the compiled-view mapping in:

```
storage/views.json
```

 The template language is implemented by Coretex's template system.

 OwnWork therefore separates:

```
resources/views/
        │
        │ source templates
        ▼
   transpilation
        │
        ▼
storage/views/
        │
        │ generated PHP
        ▼
      output
```

 Generated files in `storage/views/` are build output and should not be edited as the source of a view.

 ## Helpers

 OwnWork's predefined helper functions are located in:

```
bundle/Helper.php
```

 The helper file provides functions including:

```php
<?php

approot();
```

```php
<?php

env("APP_NAME");
```

```php
<?php

view("home.temp.php");
```

```php
<?php

comp("button.php");
```

```php
<?php

out($value);
```

 It also provides additional helpers such as `getTempTranspiled()`, `url()`, `pre()`, `get_db_instance()`, `clean()`, `isArrElemEmpty()`, `separateEmptyElements()`, `printArr()`, and `isUrl()`.

 For example, `view()` delegates to Coretex's `View::instantView()`, while `getTempTranspiled()` delegates to Coretex's `View::includeTemp()`.

 The helper layer is therefore an OwnWork convenience layer around application and Coretex functionality.

 ## Worker

 The `worker` script provides OwnWork's command-line development and code-generation functionality.

 It can:

 - start the development server
- generate controllers
- generate middleware
- generate models
- generate services
- generate views
- transpile templates
- clear generated view files

 For example:

```
php worker make controller UserController
```

 Generated application components use templates stored under:

```
resources/template/
```

 The worker is also responsible for the development view-transpilation workflow.

 ## Coretex Dependency

 OwnWork depends on:

```
dhruv125/coretex
```

 Coretex currently provides several lower-level pieces used directly by OwnWork.

 ### HTTP

 OwnWork's kernel uses Coretex:

```
Dhruv125\Coretex\Support\Request
Dhruv125\Coretex\Support\Response
```

 ### Routing

 OwnWork's kernel uses:

```
Dhruv125\Coretex\Router\Route
Dhruv125\Coretex\Router\RouteResolver
```

 ### View System

 OwnWork's helper layer delegates view operations to:

```
Dhruv125\Coretex\Viewer\View
```

 ### Environment

 OwnWork's `Bundler` creates:

```
Dhruv125\Coretex\Environment\Environment
```

 to load the application's `.env` configuration.

 ### Error Handling

 OwnWork's `Bundler` creates:

```
Dhruv125\Coretex\Handler\GlobalErrorHandler
```

 for global PHP error and exception handling.

 This distinction is important:

 > **OwnWork integrates these Coretex components into its application lifecycle; they are not all implemented inside the OwnWork repository itself.**

 When investigating framework behavior, check the OwnWork source first for application-specific orchestration, then check Coretex when the relevant class is imported from `Dhruv125\Coretex`.

 ## OwnWork and Coretex Responsibilities

 The separation can be summarized as:

 | Responsibility | OwnWork | Coretex |
| --- | --- | --- |
| Application entry point | ✓ |  |
| Bundler | ✓ |  |
| HTTP Kernel | ✓ |  |
| Application directory conventions | ✓ |  |
| Worker CLI | ✓ |  |
| Application helpers | ✓ |  |
| Request object |  | ✓ |
| Response object |  | ✓ |
| Route registration/matching |  | ✓ |
| Route resolution |  | ✓ |
| View implementation |  | ✓ |
| Environment loading |  | ✓ |
| Global error handler |  | ✓ |

This separation is important when reading the source or debugging behavior.

 ## Application Responsibilities

 OwnWork intentionally leaves much of the application-specific behavior to the developer.

 A typical application can organize responsibilities like this:

 | Layer | Responsibility |
| --- | --- |
| `public/` | HTTP entry point and public files |
| `bundle/` | Bootstrap, routes, and helpers |
| `app/Http/` | OwnWork HTTP kernel |
| `app/Controller/` | Application request handling |
| `app/Middleware/` | Request pipeline behavior |
| `app/Model/` | Application data layer |
| `app/Service/` | Reusable application logic |
| `resources/views/` | View source templates |
| `resources/template/` | Worker generation templates |
| `storage/views/` | Generated view output |

The framework provides the application plumbing while application code defines the application's behavior.

 ## Request Lifecycle Summary

 The complete lifecycle can be summarized as:

```
HTTP Request
     │
     ▼
public/index.php
     │
     ▼
Bundler
     │
     ├── Load Composer
     ├── Load .env through Coretex
     └── Initialize Coretex error handler
     │
     ▼
Kernel
     │
     ├── Create Request
     ├── Create Response
     ├── Create Route
     └── Create RouteResolver
     │
     ▼
bundle/Routes.php
     │
     ▼
Route::end()
     │
     ├── Match HTTP method
     ├── Match URL
     └── Extract dynamic parameters
     │
     ▼
Middleware Pipeline
     │
     ├── Global middleware
     └── Route middleware
     │
     ▼
RouteResolver
     │
     ├── Callable
     ├── View
     └── Controller
     │
     ▼
Response
     │
     ▼
HTTP Output
```

 The key architectural boundary is that **OwnWork controls the application lifecycle and orchestration, while Coretex supplies several of the lower-level HTTP framework primitives.**

> `fundamentals/project-structure.md`
