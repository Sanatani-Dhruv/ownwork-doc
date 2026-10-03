# Architecture

OwnWork is a small PHP application framework built around an HTTP request pipeline.

The framework provides the application bootstrap, routing integration, middleware pipeline, controller resolution, views, template compilation, helpers, and development tooling.

Some lower-level HTTP and routing functionality is provided by the `dhruv125/coretex` package.

## High-Level Architecture

A request moves through the application approximately as follows:

```text
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
     └── Error handler
     │
     ▼
Http\Kernel
     │
     ├── Request
     ├── Response
     ├── Router
     └── RouteResolver
     │
     ▼
Route Matching
     │
     ▼
Middleware Pipeline
     │
     ▼
Route Handler
     │
     ├── Closure
     ├── View
     └── Controller
     │
     ▼
Response
     │
     ▼
HTTP Output
````

 The application entry point is intentionally small. Most request orchestration is performed by `Bundler` and `Kernel`.

 ## Application Entry Point

 The public entry point is:

```
public/index.php
```

 Its responsibility is to load the application bundler and start it:

```
<?php

require __DIR__ . "/../bundle/Bundler.php";

$app = new Bundler();
$app->bundle();
```

 This makes `public/` the web-facing portion of the application.

 ## Bundler

 The bundler is located at:

```
bundle/Bundler.php
```

 Its main responsibilities are application startup tasks.

 The bootstrap process includes:

 1. Checking that the application has been configured.
2. Loading Composer's autoloader.
3. Loading the environment configuration.
4. Configuring error handling.
5. Creating the application kernel.
6. Starting request handling.

 Conceptually:

```
Bundler
   │
   ├── Validate setup
   ├── Load Composer
   ├── Load environment
   ├── Configure errors
   └── Start Kernel
```

 The bundler is not responsible for implementing individual routes or controllers.

 ## HTTP Kernel

 The application kernel is located at:

```
app/Http/Kernel.php
```

 The kernel coordinates the HTTP request.

 It creates the main request-processing objects and connects them together.

 The kernel works with:

 - `Request`
- `Response`
- `Route`
- `RouteResolver`
- `Pager`

 The kernel then loads the application's route definitions from:

```
bundle/Routes.php
```

 and processes the current request.

 ## Router

 Routing functionality is supplied by Coretex.

 OwnWork creates a router and registers the application's routes through:

```
bundle/Routes.php
```

 Typical route declarations look like:

```
$route->get("/", "home.temp.php");
```

 or:

```
$route->get("/users", [
    UserController::class,
    "index"
]);
```

 The router determines which route corresponds to the incoming HTTP method and URL.

 ## Route Resolver

 After a route has been matched, the route resolver determines how its handler should be executed.

 OwnWork supports handler forms including:

```
function () {
    return "Hello";
}
```

 a view name:

```
"home.temp.php"
```

 and a controller action:

```
[
    UserController::class,
    "index"
]
```

 This separates route matching from route execution.

 ## Middleware

 Middleware forms the processing layer between route matching and the final handler.

 A middleware receives:

```
function (
    Request $request,
    Response $response,
    callable $next
)
```

 It can:

 - inspect the request
- modify processing
- return a response immediately
- call `$next()` to continue processing

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
   │
   ▼
Response
```

 This makes middleware suitable for concerns such as authentication, request checks, logging, and other cross-cutting behavior.

 ## Controllers

 Controllers live under:

```
app/Controller/
```

 A controller action receives the request and response objects:

```
public function index(
    Request $request,
    Response $response
) {
    // ...
}
```

 Controllers are application code rather than framework configuration.

 A controller can perform application operations and return a response or rendered view.

 ## Models

 Models belong under:

```
app/Model/
```

 The directory is part of OwnWork's application structure, but OwnWork does not impose a built-in ORM or database abstraction on model classes.

 Applications are therefore free to implement their own model layer or integrate an external database/ORM library.

 ## Services

 Services belong under:

```
app/Service/
```

 A service is an application-level class intended to keep reusable operations and business logic separate from controllers.

 For example:

```
Controller
    │
    ▼
Service
    │
    ▼
Model / External API / Other dependency
```

 OwnWork does not require a particular service interface.

 ## Views

 Views live under:

```
resources/views/
```

 OwnWork supports normal PHP views and `.temp.php` templates.

 A `.temp.php` file is processed by the template engine before being executed as PHP.

 Compiled templates are stored under:

```
storage/views/
```

 This separates template source from generated output.

 ## Helpers

 Framework/application helper functions are loaded from:

```
bundle/Helper.php
```

 Examples include:

```
approot();
```

```
env("APP_NAME");
```

```
view("home.php");
```

```
comp("button.php");
```

```
out($value);
```

 These functions provide convenient access to common application operations without requiring an object to be manually instantiated.

 ## Worker

 The `worker` script provides development and code-generation commands.

 It can:

 - start the development server
- generate controllers
- generate middleware
- generate models
- generate services
- generate views
- transpile templates
- clear compiled view files

 For example:

```
php worker make controller UserController
```

 The worker uses templates stored in:

```
resources/template/
```

 ## Coretex Dependency

 OwnWork delegates several low-level framework responsibilities to:

```
dhruv125/coretex
```

 Coretex provides functionality used by OwnWork for areas such as:

 - HTTP requests
- HTTP responses
- routing
- route resolution
- views
- template compilation
- environment handling
- error handling

 This means the OwnWork architecture is best understood as two layers:

```
OwnWork
├── Application bootstrap
├── Kernel
├── Application structure
├── Helpers
├── Worker
└── Framework integration
        │
        ▼
    Coretex
    ├── Request
    ├── Response
    ├── Router
    ├── Route resolver
    ├── View system
    ├── Template engine
    ├── Environment
    └── Error handling
```

 Understanding this separation is useful when reading the source code or debugging framework behavior.

 ## Application Responsibilities

 OwnWork intentionally leaves much of the application-specific behavior to the developer.

 A typical application can organize responsibilities like this:

| Layer | Responsibility |
| --- | --- |
| `public/` | HTTP entry point and public assets |
| `bundle/` | Bootstrap, routes, helpers |
| `app/Controller/` | HTTP/application orchestration |
| `app/Middleware/` | Request pipeline behavior |
| `app/Model/` | Application data layer |
| `app/Service/` | Reusable application logic |
| `resources/views/` | Presentation |
| `resources/template/` | Worker generation templates |
| `storage/views/` | Compiled template output |

The framework provides the plumbing while application code defines the application's behavior.

> next: `fundamentals/project-structure.md`
