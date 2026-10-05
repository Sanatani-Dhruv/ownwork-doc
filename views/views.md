 # Views

 OwnWork provides a view system for rendering application-facing HTML.

 Application views are stored under:

```
resources/views/
```

 Views can be ordinary PHP files or `.temp.php` template files processed by OwnWork's template system.

 The main helper for rendering a view is:

```
view()
```

 ## View Directory

 Application views belong under:

```
resources/views/
```

 For example:

```
resources/views/
├── home.php
├── users.php
├── users.temp.php
└── profile.temp.php
```

 The `resources/views/` directory is separate from:

```
resources/appviews/
```

 `resources/appviews/` contains views used by OwnWork/Coretex itself, particularly framework error pages.

 Application templates should normally be placed under `resources/views/`.

 ## Creating a View

 A view can be created as a normal PHP file:

```
resources/views/home.php
```

 or as a template:

```
resources/views/home.temp.php
```

 For example:

```
<h1>Hello World</h1>
```

 The view can then be rendered from a controller or route.

 ## Rendering a View

 Use the global `view()` helper:

```
return view("home.php");
```

 For a template view:

```
return view("home.temp.php");
```

 The view name identifies the file under:

```
resources/views/
```

 For example:

```
return view("users.temp.php");
```

 resolves to:

```
resources/views/users.temp.php
```

 ## Passing Data to a View

 The `view()` helper accepts an optional data array.

 For example:

```
return view("users.temp.php", [
    "title" => "Users"
]);
```

 The supplied values are made available to the view when it is rendered.

 A template can then use the value:

```
<h1>{{ $title }}</h1>
```

 The exact template syntax is provided by the OwnWork/Coretex template system.

 ## View Data

 Multiple values can be passed to a view:

```
return view("profile.temp.php", [
    "name" => "Dhruv",
    "email" => "user@example.com"
]);
```

 The template can access the supplied variables:

```
<h1>{{ $name }}</h1>

<p>{{ $email }}</p>
```

 The data array provides the boundary between application code and the rendered view.

 ## Views from Controllers

 Controllers commonly render views as their final operation.

 For example:

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
        return view("users.temp.php", [
            "title" => "Users"
        ]);
    }
}
```

 The route can point to the controller:

```
$route->get("/users", [
    UserController::class,
    "index"
]);
```

 The request flow becomes:

```
HTTP Request
     ↓
Route
     ↓
Controller
     ↓
view()
     ↓
resources/views/users.temp.php
     ↓
Rendered HTML
     ↓
HTTP Response
```

 ## Views Directly from Routes

 A view can also be used directly as a route handler.

 For example:

```
$route->get("/", "home.temp.php");
```

 Here, the string handler identifies a view.

 The route resolver treats the string as a view handler rather than requiring a controller.

 This is useful for simple pages that do not require application logic.

 ## PHP Views

 OwnWork supports ordinary PHP views.

 For example:

```
resources/views/home.php
```

 with:

```
<h1>
    <?php echo $title; ?>
</h1>
```

 The PHP view can use normal PHP syntax.

 The view system therefore does not require every application page to use the `.temp.php` template format.

 ## Template Views

 OwnWork also supports `.temp.php` files.

 For example:

```
resources/views/home.temp.php
```

 A template can use OwnWork's template syntax:

```
<h1>{{ $title }}</h1>
```

 Template files are processed by the template system before their resulting PHP representation is executed.

 This provides a template-oriented syntax while still producing PHP-based view output.

 ## Template Compilation

 Template views are compiled before execution.

 The generated representations are stored under:

```
storage/views/
```

 The view system also maintains:

```
storage/views.json
```

 which maps template sources to their compiled representations.

 Conceptually:

```
resources/views/home.temp.php
             │
             ▼
       Template system
             │
             ▼
       storage/views/
             │
             ▼
        Rendered output
```

 Generated view files should generally not be edited manually.

 ## View Cache

 Compiled template output is generated data.

 When application templates change, the generated representation may need to be regenerated.

 OwnWork provides a worker command for clearing compiled view data:

```
php worker clear:viewcache
```

 This removes the generated view cache so that templates can be compiled again.

 ## View Paths

 View names are resolved relative to:

```
resources/views/
```

 For example:

```
view("home.php");
```

 refers to:

```
resources/views/home.php
```

 and:

```
view("users/profile.temp.php");
```

 refers to:

```
resources/views/users/profile.temp.php
```

 Applications can therefore organize views into directories:

```
resources/views/
├── home.temp.php
├── users/
│   ├── index.temp.php
│   ├── profile.temp.php
│   └── edit.temp.php
└── admin/
    ├── dashboard.temp.php
    └── users.temp.php
```

 ## View Organization

 There is no required application-wide naming convention for views.

 A project can organize templates according to its needs.

 For example:

```
resources/views/
├── layouts/
├── users/
├── products/
├── orders/
└── dashboard/
```

 The important distinction is that application views belong under:

```
resources/views/
```

 while framework-owned views belong under:

```
resources/appviews/
```

 ## Views and Components

 Reusable pieces of application markup can be separated from complete page views.

 OwnWork provides the `comp()` helper for components:

```
comp("button.php");
```

 Components are covered separately in:

```
views/components.md
```

 A typical relationship is:

```
View
 ├── Layout
 ├── Component
 └── Component
```

 Views represent pages or larger rendered sections, while components can provide reusable pieces of markup.

 ## Views and Controllers

 A controller should normally decide which view should be rendered based on the application operation.

 For example:

```
public function profile(
    Request $request,
    Response $response
) {
    $params = $request->getAttribute(
        "dynamicParams"
    );

    return view("users/profile.temp.php", [
        "id" => $params["id"]
    ]);
}
```

 The controller handles HTTP and application coordination.

 The view handles presentation.

 A simple separation is:

```
Controller
    ↓
Application data
    ↓
View
    ↓
HTML
```

 ## Views and Services

 Services can prepare application data before a controller passes it to a view.

 For example:

```
Controller
    ↓
UserService
    ↓
User data
    ↓
View
```

 The view should generally focus on rendering the data rather than performing persistence operations.

 ## Views and Models

 Views should not normally be responsible for database access.

 Instead:

```
Controller
    ↓
Service / Model
    ↓
Data
    ↓
View
```

 For example:

```
$user = $service->findUser($id);

return view("users/profile.temp.php", [
    "user" => $user
]);
```

 The template can then render the supplied data.

 ## View Errors

 If a requested view does not exist, the view system can raise a framework view exception.

 For example:

```
return view("missing.temp.php");
```

 If the corresponding file cannot be resolved, Coretex's view/error handling processes the failure.

 View errors are therefore part of the application's normal error-handling pipeline.

 See:

```
errors/error-handling.md
```

 ## Application Views vs Framework Views

 OwnWork separates application views from framework views.

 ### Application views

```
resources/views/
```

 These are controlled by the application developer.

 ### Framework views

```
resources/appviews/
```

 These are used by OwnWork/Coretex, including error-related presentation.

 Applications should normally create their own views under:

```
resources/views/
```

 rather than modifying framework views.

 ## View Generation

 OwnWork's worker includes view generation support.

 The worker uses templates under:

```
resources/template/
```

 when generating application resources.

 Generated application views belong under:

```
resources/views/
```

 The exact generated content depends on the worker template and the command being used.

 ## Complete Example

 A route:

```
$route->get("/users/{id}", [
    UserController::class,
    "show"
]);
```

 can be handled by:

```
public function show(
    Request $request,
    Response $response
) {
    $params = $request->getAttribute(
        "dynamicParams"
    );

    $id = $params["id"];

    return view("users/profile.temp.php", [
        "id" => $id
    ]);
}
```

 The corresponding view:

```
resources/views/users/profile.temp.php
```

 can contain:

```
<h1>User Profile</h1>

<p>User ID: {{ $id }}</p>
```

 The resulting flow is:

```
GET /users/42
      ↓
Route matching
      ↓
UserController::show()
      ↓
dynamicParams
      ↓
view("users/profile.temp.php")
      ↓
Template compilation
      ↓
Rendered HTML
      ↓
HTTP Response
```

 ## View Responsibilities

 Views are primarily responsible for presentation.

 Typical view responsibilities include:

 - rendering HTML
- displaying application data
- using template syntax
- including reusable components
- presenting application state to the user

 Views should generally not be responsible for:

 - route registration
- request lifecycle management
- database persistence
- authentication decisions
- application-wide business operations

 Those responsibilities belong elsewhere in the application.

 ## View Workflow

 A typical OwnWork view workflow is:

```
Create view
    ↓
resources/views/
    ↓
Create PHP or .temp.php template
    ↓
Controller or route calls view()
    ↓
Template processing
    ↓
Compiled representation
    ↓
Rendered output
    ↓
HTTP response
```

 The next view-specific topics cover the individual parts of this system:

```
views/templating.md
views/components.md
views/transpilation.md
```

> `views/templating.md`
