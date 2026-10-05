# Helpers API

 OwnWork provides global helper functions through `bundle/Helper.php`. The file is registered through Composer's `autoload.files`, so these helpers are available throughout the application.

 ## `approot()`

 Returns the application root directory.

```
$root = approot();
```

 Use it when constructing paths relative to the application:

```
$viewPath = approot() . "/resources/views/";
$storagePath = approot() . "/storage/";
```

 ## `env()`

 Reads or writes an environment variable.

```
$value = env("APP_NAME");
```

 The helper accepts:

```
env($data, $get = true);
```

 For reading:

```
$appName = env("APP_NAME");
```

 For writing:

```
env("APP_NAME=My Application", false);
```

 OwnWork's current helper delegates to PHP's `getenv()` and `putenv()`.

 ## `out()`

 Escapes a string for HTML output.

```
echo out($value);
```

 It uses:

```
htmlspecialchars($data)
```

 This is useful when displaying a value that should be treated as HTML text rather than markup.

 ## `view()`

 Renders a view through Coretex.

```
view("home.temp.php");
```

 Data can be passed as the second argument:

```
view("users/index.temp.php", [
    "users" => $users
]);
```

 The helper delegates to `Dhruv125\Coretex\Viewer\View::instantView()`.

 ## `getTempTranspiled()`

 Resolves and includes a transpiled template.

```
getTempTranspiled("header.temp.php");
```

 It accepts:

```
getTempTranspiled(
    string $string,
    bool $needPath = true,
    array $args = []
);
```

 The helper delegates to Coretex's `View::includeTemp()`. It is used by the template system for `@@includeTemp(...)`.

 ## `comp()`

 Renders a component.

```
comp("button");
```

 Arguments can be supplied:

```
comp("button", [
    "text" => "Save"
]);
```

 The helper accepts an optional component directory:

```
comp(
    "button",
    ["text" => "Save"],
    "/path/to/components/"
);
```

 When no directory is supplied, OwnWork looks under:

```
resources/views/component/
```

 The supplied arguments are extracted into the component before the component file is required. A missing component throws an `ErrorException`.

 The template directive:

```
@comp("button", [
    "text" => "Save"
])
```

 uses this helper.

 ## `url()`

 Provides information about the current request URL.

 Get the current path:

```
url("get");
```

 Get the complete request URI:

```
url("getFull");
```

 The current OwnWork helper supports these actions:

```
get
getFull
```

 For example:

```
$currentPath = url("get");
$fullUrl = url("getFull");
```

 `url("get")` resolves the path from `$_SERVER["REQUEST_URI"]`, while `url("getFull")` returns the complete request URI.

 ## `pre()`

 Prints a value inside a `<pre>` element.

```
pre($data);
```

 It internally uses `print_r()`:

```
pre($users);
```

 This is primarily useful for simple debugging during development.

 ## `get_db_instance()`

 Returns a database instance when the optional `delight-im/db` dependency is available and configured.

```
$db = get_db_instance();
```

 The helper checks for the database package and supports the configured:

```
DB_DRIVER=sql
```

 or:

```
DB_DRIVER=sqlite
```

 For SQL configuration it reads values such as:

```
DB_HOST
DB_NAME
DB_USER
DB_PASS
```

 For SQLite, OwnWork creates the database under:

```
storage/db/
```

 when required.

 This helper is optional application infrastructure rather than a required part of OwnWork's MVC flow.

 ## `clean()`

 The current OwnWork helper also provides:

```
clean($value);
```

 It trims a string and returns `null` when the resulting value is empty.

```
$value = clean($input);
```

 Its behavior is equivalent to:

```
trim($value) !== ""
    ? trim($value)
    : null;
```



 Checks whether an array contains an empty/falsy element.

```
if (isArrElemEmpty($values)) {
    // ...
}
```

 It returns:

```
true
```

 when any element evaluates as falsy, otherwise:

```
false
```



 Separates an array into elements that are considered empty and elements that contain values.

```
[$filled, $empty] = separateEmptyElements($values);
```

 The returned array contains:

```
[
    $fullArray,
    $emptyKeys
]
```

 For example:

```
[$full, $empty] = separateEmptyElements([
    "name" => "Dhruv",
    "email" => "",
]);
```

 The first element contains the non-empty entries and the second contains the keys of empty entries.

 ## `printArr()`

 Prints an array inside a `<pre>` element.

```
printArr($array);
```

 It is a convenience debugging helper around `print_r()`.

 ## `isUrl()`

 Checks whether the current URL matches a supplied URL.

```
isUrl("/users");
```

 The second argument determines which `url()` value is compared:

```
isUrl("/users", "get");
```

 Internally it compares the supplied value with:

```
url($action)
```



 OwnWork also depends on Coretex for its request, response, routing, environment, error handling, view, and templating infrastructure. The OwnWork repository explicitly installs `dhruv125/coretex` as its framework dependency.

 The OwnWork helper layer therefore should not be documented as though `approot()`, `view()`, `comp()`, and `getTempTranspiled()` are the only available framework functions.

 ## Quick Reference

 | Helper | Purpose |
| --- | --- |
| `approot()` | Get the application root |
| `env()` | Read or write an environment variable |
| `out()` | Escape a value with `htmlspecialchars()` |
| `view()` | Render a view |
| `getTempTranspiled()` | Include a transpiled template |
| `comp()` | Render a component |
| `url()` | Get the current request path or URI |
| `pre()` | Print debug output |
| `get_db_instance()` | Get the optional database instance |
| `clean()` | Trim a value and convert empty strings to `null` |
| `isArrElemEmpty()` | Check for empty/falsy array elements |
| `separateEmptyElements()` | Separate filled values and empty keys |
| `printArr()` | Print an array for debugging |
| `isUrl()` | Check the current URL |

## Related Documentation

 - Environment Configuration
- Request API
- Response API
- Views
- Components
- View Transpilation

 > next: `README.md`
