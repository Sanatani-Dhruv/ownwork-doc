# Configuration Helpers

OwnWork provides global helpers for accessing application configuration, environment values, and application paths.

## `approot()`

Returns the application root directory.

```php
$root = approot();
````

 Use it when constructing paths inside the application:

```
$viewPath = approot() . "/resources/views/";
```

 For example:

```
$storagePath = approot() . "/storage/";
```

 ## `env()`

 Reads an environment variable.

```
$value = env("APP_NAME");
```

 Example:

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

 ## `view()`

 The `view()` helper renders an application view.

```
return view("home.temp.php");
```

 A view can receive data:

```
return view("users/index.temp.php", [
    "users" => $users
]);
```

 The view path is relative to:

```
resources/views/
```

 ## `comp()`

 The `comp()` helper is used by the `@comp()` template directive.

 In a template:

```
@comp("button")
```

 is transpiled into a call to:

```
comp("button");
```

 Arguments can be passed to the component:

```
@comp("button", [
    "text" => "Save"
])
```

 ## `getTempTranspiled()`

 OwnWork's template system uses:

```
getTempTranspiled(...)
```

 for transpiled-template includes created by:

```
@@includeTemp(...)
```

 Example:

```
@@includeTemp("header.temp.php")
```

 The template parser converts this directive into an include using `getTempTranspiled()`.

 ## `approot()` in Paths

 When accessing files from the application root:

```
approot() . "/config/app.php"
```

 When accessing application views:

```
approot() . "/resources/views/home.temp.php"
```

 When accessing compiled views:

```
approot() . "/storage/views/"
```

 ## Using Helpers in Controllers

 Helpers can be used directly from controllers:

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

 ## Using Helpers in Views

 Helpers can also be used from templates:

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
| `getTempTranspiled(...)` | Resolve a transpiled template include |

> next: `errors/error-handling.md`
