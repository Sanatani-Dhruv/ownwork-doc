# Views

Views are the presentation layer of an OwnWork application.

Application views are stored under:

```text
resources/views/
````

 OwnWork provides a view system through Coretex. The framework uses its own templating and view-transpilation pipeline rather than requiring a third-party template engine. The package describes this as a Blade-like templating engine and identifies `.temp.php` files as its template format.  Packagist

 ## View Directory

 The default application structure contains:

```
resources/
└── views/
```

 Application templates are placed inside this directory.

 For example:

```
resources/
└── views/
    ├── home.temp.php
    ├── users.temp.php
    └── users/
        ├── index.temp.php
        └── show.temp.php
```

 The exact organization of view files is up to the application.

 ## Creating a View

 Use the OwnWork worker to generate a view:

```
php worker make view home
```

 The generated view is created under:

```
resources/views/
```

 The worker uses the framework's view template located under:

```
resources/template/View.php
```

 This template is used as the starting point for generated application views.  Packagist

 ## `.temp.php` Files

 OwnWork templates use the `.temp.php` extension.

 For example:

```
resources/views/home.temp.php
```

 A template can contain HTML:

```
<h1>Hello World</h1>
```

 PHP-compatible template expressions and OwnWork's templating syntax can be used where supported by the templater.

 ## Rendering a View

 A view can be rendered using the global `view()` helper:

```
return view("home.temp.php");
```

 A controller can therefore render a view like this:

```php
<?php

namespace App\Controller;

use Dhruv125\Coretex\Support\Request;
use Dhruv125\Coretex\Support\Response;

class HomeController
{
    public function index(
        Request $request,
        Response $response
    ) {
        return view("home.temp.php");
    }
}
```

 The view helper locates the requested template in the application's views directory.

 ## Passing Data to a View

 The `view()` helper accepts data for the template.

 For example:

```
return view("home.temp.php", [
    "title" => "Home"
]);
```

 The template can access the supplied value:

```
<h1>{{ $title }}</h1>
```

 A larger data set can be passed in the same way:

```
return view("users/index.temp.php", [
    "title" => "Users",
    "users" => $users
]);
```

 ## View Data

 View data is normally supplied by the controller.

 The common flow is:

```
Controller
    ↓
view()
    ↓
Template data
    ↓
Template
    ↓
Rendered output
```

 For example:

```
public function index(
    Request $request,
    Response $response
) {
    $users = [
        "Alice",
        "Bob"
    ];

    return view("users/index.temp.php", [
        "users" => $users
    ]);
}
```

 The template can then render the supplied data.

 ## Template Example

 Controller:

```
public function index(
    Request $request,
    Response $response
) {
    return view("users/index.temp.php", [
        "title" => "Users",
        "users" => [
            "Alice",
            "Bob",
            "Charlie"
        ]
    ]);
}
```

 Template:

```
<h1>{{ $title }}</h1>

<ul>
    @foreach ($users as $user)
        <li>{{ $user }}</li>
    @endforeach
</ul>
```

 The template syntax is processed by OwnWork's templating engine.

 ## View Compilation

 OwnWork does not render every template directly on every request.

 The framework includes a view-transpilation process.

 Run the transpiler with:

```
php worker transpile
```

 The package also exposes the equivalent Composer script:

```
composer run transpile
```

 The transpiler converts application templates into their compiled representation.  Packagist

 ## Compiled Views

 Compiled view information is maintained under:

```
storage/views/
```

 The application also contains:

```
resources/views.json
```

 which maintains the mapping between template files and their compiled forms.  Packagist

 Conceptually:

```
resources/views/
       │
       │ .temp.php
       ▼
   Transpiler
       │
       ▼
storage/views/
       │
       ▼
    Renderer
       │
       ▼
  HTTP Response
```

 ## Running the Transpiler

 During development, run:

```
php worker transpile
```

 in a separate terminal while the development server is running:

```
php worker serve
```

 The package README also provides:

```
composer run dev
```

 for starting the development server and:

```
composer run transpile
```

 for the template transpiler.  Packagist

 If Node.js is installed, the project also provides an npm development workflow:

```
npm run dev
```

 which can run the development tooling together.  Packagist

 ## View Path

 Application views belong in:

```
resources/views/
```

 Do not place normal application views in:

```
resources/appviews/
```

 The `appviews` directory contains framework-provided views used by OwnWork itself, including error-related templates.  Packagist

 The distinction is:

```
resources/
├── appviews/    # OwnWork/framework views
└── views/       # Application views
```

 ## Framework Views

 OwnWork contains internal views for framework-level pages.

 The package structure includes:

```
resources/appviews/
├── error_layout.php
├── no-info-error.php
├── stackTrace-block.php
├── script/
│   └── script.js
└── styles/
    └── index.css
```

 These are used by the framework's error and exception presentation system.  Packagist

 Application developers normally should not use `resources/appviews/` for ordinary application pages.

 ## Views and Controllers

 Controllers decide which view should be rendered.

 For example:

```
class UserController
{
    public function index(
        Request $request,
        Response $response
    ) {
        return view("users/index.temp.php", [
            "title" => "Users"
        ]);
    }
}
```

 The corresponding structure is:

```
app/
└── Controller/
    └── UserController.php

resources/
└── views/
    └── users/
        └── index.temp.php
```

 This provides a straightforward relationship between application controllers and presentation templates.

 ## Views and Models

 Views should normally receive prepared application data rather than performing persistence operations themselves.

 A typical MVC flow is:

```
Request
   ↓
Controller
   ↓
Service
   ↓
Model
   ↓
Data
   ↓
Controller
   ↓
View
   ↓
Response
```

 For example:

```
$users = $userService->findAll();

return view("users/index.temp.php", [
    "users" => $users
]);
```

 The template is then responsible for presenting `$users`.

 ## Views and Services

 A service can prepare application data before the controller passes it to the view:

```
$users = $userService->listUsers();

return view("users/index.temp.php", [
    "users" => $users
]);
```

 This keeps application operations outside the presentation layer.

 ## View File Naming

 OwnWork templates use the `.temp.php` suffix.

 Examples:

```
home.temp.php
login.temp.php
users.temp.php
users/index.temp.php
users/show.temp.php
```

 The naming convention is primarily for the OwnWork templating/transpilation system.

 ## Nested View Directories

 Views can be organized into directories:

```
resources/views/
├── home.temp.php
├── users/
│   ├── index.temp.php
│   └── show.temp.php
└── admin/
    ├── dashboard.temp.php
    └── users.temp.php
```

 Render a nested template by providing its relative view path:

```
return view("users/index.temp.php");
```

 This allows larger applications to organize templates by feature.

 ## Views and Static Assets

 Static files should be placed under:

```
public/
```

 The package structure identifies `public/` as the directory exposed to users and recommends placing static assets there.  Packagist

 A typical project can therefore look like:

```
public/
├── index.php
├── styles/
└── build/

resources/
└── views/
    └── home.temp.php
```

 Views generate application markup while `public/` contains files directly served to the client.

 ## Tailwind CSS

 A default OwnWork project includes Tailwind-related resources:

```
resources/css/tailwind.css
```

 and compiled CSS under:

```
public/styles/
```

 The package's default structure includes:

```
resources/css/tailwind.css
public/styles/tailwind.default.css
```

 as part of its frontend setup.  Packagist

 This styling system is independent from the view renderer.

 ## View Errors

 The Coretex layer provides exceptions related to view processing, including:

```
ViewNotFoundException
ViewJsonNotFoundException
```

 A missing or invalid view can therefore result in framework-level error handling.

 See Error Handling.

 ## View Compilation Workflow

 A complete development workflow can be represented as:

```
Create template
      ↓
resources/views/*.temp.php
      ↓
Run transpiler
      ↓
Compiled view
      ↓
Controller calls view()
      ↓
View renderer
      ↓
HTML output
      ↓
HTTP response
```

 During development, the transpiler can be run continuously alongside the development server.

 ## Example: Complete View

 Project:

```
app/
└── Controller/
    └── HomeController.php

resources/
└── views/
    └── home.temp.php
```

 Controller:

```php
<?php

namespace App\Controller;

use Dhruv125\Coretex\Support\Request;
use Dhruv125\Coretex\Support\Response;

class HomeController
{
    public function index(
        Request $request,
        Response $response
    ) {
        return view("home.temp.php", [
            "title" => "OwnWork",
            "message" => "Welcome!"
        ]);
    }
}
```

 Template:

```
<!DOCTYPE html>
<html>
<head>
    <title>{{ $title }}</title>
</head>
<body>
    <h1>{{ $title }}</h1>

    <p>{{ $message }}</p>
</body>
</html>
```

 Route:

```
$route->get("/", [
    HomeController::class,
    "index"
]);
```

 The resulting request flow is:

```
GET /
  ↓
Route
  ↓
HomeController::index()
  ↓
view("home.temp.php", ...)
  ↓
Template transpilation/rendering
  ↓
HTML
  ↓
Client
```

 ## View API Boundary

 OwnWork's view functionality is implemented through Coretex's viewer and templater components.

 The package structure identifies:

```
Coretex/
├── Templater/
│   └── Template.php
└── Viewer/
    └── View.php
```

 The `Template` component handles `.temp.php` template processing, while the `View` component handles view-related operations such as locating and displaying compiled views.  Packagist

 OwnWork integrates these components into the application lifecycle.

 ## Quick Reference

| Task | Command / API |
| --- | --- |
| Generate a view | `php worker make view <name>` |
| Application view directory | `resources/views/` |
| Template extension | `.temp.php` |
| Render a view | `view("file.temp.php")` |
| Pass view data | `view("file.temp.php", [...])` |
| Transpile views | `php worker transpile` |
| Compiled view directory | `storage/views/` |
| View mapping | `resources/views.json` |
| Framework views | `resources/appviews/` |

For template syntax and directives, see Templating.

> next: `views/templating.md`
