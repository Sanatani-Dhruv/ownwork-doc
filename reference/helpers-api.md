# Helpers API

OwnWork provides global helper functions used throughout the application and view system.

## `approot()`

Returns the application root path.

```php
$root = approot();
````

 It can be used to construct paths relative to the application:

```
$viewPath = approot() . "/resources/views/";
```

 The framework itself uses `approot()` when resolving application directories such as:

```
resources/views/
storage/
storage/views/
```

 ## `env()`

 Reads an environment value.

```
$value = env("APP_NAME");
```

 Example:

```
$appName = env("APP_NAME");
```

 Environment values can be used for application configuration without hard-coding environment-specific values into application code.

 ## `view()`

 Renders an application view.

```
return view("home.temp.php");
```

 A view can receive data:

```
return view("users/index.temp.php", [
    "users" => $users
]);
```

 View paths are resolved relative to the application's:

```
resources/views/
```

 directory.

 ## `comp()`

 Renders a component from a view.

 The template directive:

```
@comp("button")
```

 is transpiled into a PHP call to:

```
comp("button")
```

 Arguments can be passed to the component:

```
@comp("button", [
    "text" => "Save"
])
```

 ## `getTempTranspiled()`

 Resolves a transpiled template for inclusion.

 The template directive:

```
@@includeTemp("header.temp.php")
```

 is converted by the template parser to an include using:

```
getTempTranspiled(...)
```

 This helper is used by the template compilation system when working with compiled `.c.php` views.

 ## Helper Usage

 Helpers can be called directly from PHP:

```
$root = approot();
$name = env("APP_NAME");

return view("home.temp.php", [
    "name" => $name
]);
```

 They can also be used from templates where appropriate:

```
<h1>{{ env("APP_NAME") }}</h1>
```

 ## Quick Reference

 | Helper | Purpose |
| --- | --- |
| `approot()` | Get the application root path |
| `env("KEY")` | Read an environment value |
| `view("file")` | Render a view |
| `comp(...)` | Render a component |
| `getTempTranspiled(...)` | Resolve a transpiled template |

## Related Documentation

 - Environment Configuration
- Views
- Components
- View Transpilation

> next: `README.md`
