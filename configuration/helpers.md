# Configuration Helpers

 OwnWork provides global helpers for application paths, environment values, views, components, and transpiled templates.

 ## approot()

 Returns the application root directory.

```
$root = approot();
```

 Use it to construct paths relative to the project root:

```
$viewPath = approot() . "/resources/views/";
$storagePath = approot() . "/storage/";
```

 ## env()

 Reads an environment variable:

```
$value = env("APP_NAME");
```

 For example:

```
$appName = env("APP_NAME");
$debug = env("APP_DEBUG");
```

 Given:

```
APP_NAME=My Application
APP_DEBUG=true
```

 the values can be accessed with:

```
env("APP_NAME");
env("APP_DEBUG");
```

 See Environment Configuration.

 ## view()

 Renders an application view:

```
return view("home.temp.php");
```

 Data can be passed as the second argument:

```
return view("users/index.temp.php", [
    "users" => $users
]);
```

 View paths are resolved from:

```
resources/views/
```

 ## comp()

 The `comp()` helper renders a component.

 It is also the helper used by the `@comp()` template directive.

```
comp("button");
```

 A component can receive arguments:

```
comp("button", [
    "text" => "Save"
]);
```

 In a template:

```
@comp("button", [
    "text" => "Save"
])
```

 See Components.

 ## getTempTranspiled()

 `getTempTranspiled()` resolves a transpiled template for the template include mechanism.

 The `@@includeTemp()` directive uses this helper internally.

 For example:

```
@@includeTemp("header.temp.php")
```

 is converted by the templating system into an include using `getTempTranspiled()`.

 See View Transpilation.

 ## Helpers in Controllers

 Helpers can be called directly from controllers:

```
public function index(
    Request $request,
    Response $response
) {
    $title = env("APP_NAME");

    return view("home.temp.php", [
        "title" => $title
    ]);
}
```

 ## Helpers in Views

 Helpers are also available inside templates:

```
<h1>{{ env("APP_NAME") }}</h1>
```

 Application paths can be constructed with:

```
@php
$path = approot() . "/storage/";
@endphp;
```

 ## Quick Reference

 | Helper | Purpose |
| --- | --- |
| `approot()` | Get the application root path |
| `env("KEY")` | Read an environment value |
| `view("file")` | Render a view |
| `comp(...)` | Render a component |
| `getTempTranspiled(...)` | Resolve a transpiled template |

> `errors/error-handling.md`
