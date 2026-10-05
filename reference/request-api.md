# Request API

 The OwnWork request object is provided by Coretex:

```
use Dhruv125\Coretex\Support\Request;
```

 The current Coretex `Request` implementation wraps the PHP request globals and provides APIs for query data, POST data, cookies, files, server values, request attributes, headers, route information, and request metadata.

 ## Request Object

 Controllers receive the request and response objects:

```
use Dhruv125\Coretex\Support\Request;
use Dhruv125\Coretex\Support\Response;

public function index(
    Request $request,
    Response $response
) {
    // ...
}
```

 When constructed, the request captures:

 - `$_GET`
- `$_POST`
- `$_REQUEST`
- `$_FILES`
- `$_COOKIE`
- `$_SERVER`

 It also initializes request attributes and determines the current URL from `REQUEST_URI`.

 ## `method()`

 Returns the HTTP request method in uppercase.

```
$method = $request->method();
```

 For example:

```
GET
POST
PUT
DELETE
```

 The implementation reads `REQUEST_METHOD` and defaults to `GET` when it is not present.

 ## `currentUrl`

 The request exposes the current URL path through the public `currentUrl` property:

```
$url = $request->currentUrl;
```

 It is initialized from `REQUEST_URI`.

 ## GET Data

 ### `get()`

 Retrieve one or more GET parameters:

```
$id = $request->get("id");
```

 For example, for:

```
/users?id=25
```

 you can use:

```
$id = $request->get("id");
```

 Multiple values can be requested at once:

```
$data = $request->get([
    "name",
    "email"
]);
```

 The returned array contains the requested keys. Missing values are returned as `null`. String values are trimmed.

 ### `allGet()`

 Retrieve all GET parameters:

```
$query = $request->allGet();
```

 This returns the complete captured `$_GET` array.

 ## POST Data

 ### `post()`

 Retrieve one or more POST values:

```
$name = $request->post("name");
```

 Multiple values can be requested:

```
$data = $request->post([
    "name",
    "email"
]);
```

 String values are trimmed, and missing values are returned as `null`.

 ### `allPost()`

 Retrieve all POST values:

```
$data = $request->allPost();
```

 This returns the captured `$_POST` array.

 ## Combined Request Data

 ### `input()`

 `input()` provides access to a specific request source:

```
$request->input("get", "name");
$request->input("post", "name");
$request->input("cookie", "session");
$request->input("request", "name");
$request->input("server", "REQUEST_METHOD");
```

 Supported sources are:

```
get
post
cookie
request
server
```

 An unsupported source causes an `InternalErrorException`. Non-server string values are trimmed.

 For example:

```
$name = $request->input("post", "name");
```

 The `request` source accesses the captured `$_REQUEST` data:

```
$value = $request->input("request", "value");
```

 ## Checking Input

 ### `has()`

 Check whether a request value exists:

```
if ($request->has("email")) {
    // ...
}
```

 The method checks the captured `$_REQUEST` data.

 ### `missing()`

 Check whether a request value does not exist:

```
if ($request->missing("email")) {
    // ...
}
```

 It is the inverse of `has()`.

 ### `filled()`

 Check whether a request value exists and contains a meaningful value:

```
if ($request->filled("email")) {
    // ...
}
```

 Strings containing only whitespace are considered empty. Arrays must contain at least one item. Other non-null values are considered filled.

 ## Cookies

 ### `allCookie()`

 Retrieve all captured cookies:

```
$cookies = $request->allCookie();
```

 ### `input()`

 A specific cookie can be read with:

```
$session = $request->input(
    "cookie",
    "session"
);
```

 The request also supports retrieving cookie values internally through its data-access API.

 ## Files

 ### `file()`

 Retrieve an uploaded file:

```
$file = $request->file("avatar");
```

 If the requested file does not exist, `null` is returned.

 Multiple files can be requested:

```
$files = $request->file([
    "avatar",
    "document"
]);
```

 The returned array contains each requested file, using `null` for missing entries.

 ### `allFile()`

 Retrieve all uploaded files:

```
$files = $request->allFile();
```

 This returns the captured `$_FILES` array.

 ## Server Values

 ### `allServer()`

 Retrieve all captured server variables:

```
$server = $request->allServer();
```

 ### `input()`

 A specific server value can be accessed with:

```
$method = $request->input(
    "server",
    "REQUEST_METHOD"
);
```

 Server values are not automatically trimmed by `input()`.

 ## All Request Data

 ### `all()`

 Retrieve the captured request collections:

```
$data = $request->all();
```

 The returned array contains:

```
[
    "requests" => ...,
    "cookies" => ...,
    "files" => ...,
    "gets" => ...,
    "posts" => ...,
    "server" => ...
]
```

 This represents the request data captured when the `Request` object was constructed.

 ## Headers

 ### `getHeaders()`

 Retrieve HTTP headers:

```
$headers = $request->getHeaders();
```

 If PHP's `getallheaders()` function exists, Coretex uses it. Otherwise, it builds the headers from `$_SERVER`, including `HTTP_*` values and the standard `CONTENT_TYPE`, `CONTENT_LENGTH`, and `CONTENT_MD5` entries.

 For example:

```
$headers = $request->getHeaders();

$contentType = $headers["Content-Type"] ?? null;
```

 ## Request Attributes

 Request attributes provide a way for framework components such as the router or middleware to attach additional information to the request.

 ### `setAttribute()`

 Set an attribute:

```
$request->setAttribute(
    "user",
    $user
);
```

 The method returns the request object, so calls can be chained:

```
$request
    ->setAttribute("user", $user)
    ->setAttribute("admin", true);
```



 Retrieve an attribute:

```
$user = $request->getAttribute("user");
```

 If the attribute does not exist, `null` is returned.

 ### `hasAttribute()`

 Check whether an attribute exists:

```
if ($request->hasAttribute("user")) {
    // ...
}
```

 Unlike a simple truthiness check, this checks whether the attribute key exists.

 ### `removeAttribute()`

 Remove an attribute:

```
$request->removeAttribute("user");
```

 The method returns the request object.

 ## Route Attributes

 OwnWork's router can store route-related information on the request through request attributes.

 For example, application code can retrieve an attribute:

```
$routes = $request->getAttribute(
    "routesArray"
);
```

 The `inRoutes()` API uses the `routesArray` request attribute to determine whether a route exists for a particular HTTP method.

 ## `inRoutes()`

 Check whether a route exists for a supplied HTTP method:

```
$request->inRoutes(
    "/users",
    "GET"
);
```

 It returns a boolean.

 The method:

 - reads the `routesArray` request attribute
- compares the supplied method case-insensitively by converting it to uppercase
- checks the routes registered for that method

 If `routesArray` has not been set, an `InternalErrorException` is thrown.

 ## Route Parameters

 Route parameters are framework-level request attributes rather than a method currently defined directly on Coretex's `Request` class.

 OwnWork can expose matched route information through request attributes. When working with the version of OwnWork that sets a `dynamicParams` attribute, parameters can be retrieved with:

```
$params = $request->getAttribute(
    "dynamicParams"
);

$id = $params["id"] ?? null;
```

 Applications should use the route data actually attached by the OwnWork router rather than assuming that `param()` exists on the Coretex `Request` class.

 ## Request Data Example

 A controller can combine the request APIs:

```
public function store(
    Request $request,
    Response $response
) {
    if ($request->missing("name")) {
        // ...
    }

    $name = $request->post("name");
    $email = $request->post("email");

    $method = $request->method();

    return $response->json([
        "name" => $name,
        "email" => $email,
        "method" => $method
    ]);
}
```

 ## Uploaded File Example

```
public function upload(
    Request $request,
    Response $response
) {
    $file = $request->file("avatar");

    if ($file === null) {
        // No file supplied.
    }

    // Process the uploaded file.
}
```

 ## Request API Summary

 | Method / Property | Purpose |
| --- | --- |
| `$request->method()` | Get the HTTP method |
| `$request->currentUrl` | Get the current URL path |
| `$request->get($name)` | Get GET data |
| `$request->post($name)` | Get POST data |
| `$request->input($source, $key)` | Get data from a specific request source |
| `$request->allGet()` | Get all GET data |
| `$request->allPost()` | Get all POST data |
| `$request->allCookie()` | Get all cookies |
| `$request->allFile()` | Get all uploaded files |
| `$request->allServer()` | Get all server values |
| `$request->all()` | Get all captured request collections |
| `$request->file($name)` | Get an uploaded file |
| `$request->has($key)` | Check whether request data exists |
| `$request->missing($key)` | Check whether request data is missing |
| `$request->filled($key)` | Check whether request data is filled |
| `$request->getHeaders()` | Get HTTP headers |
| `$request->setAttribute($name, $value)` | Set a request attribute |
| `$request->getAttribute($name)` | Get a request attribute |
| `$request->hasAttribute($name)` | Check whether an attribute exists |
| `$request->removeAttribute($name)` | Remove an attribute |
| `$request->inRoutes($route, $method)` | Check whether a route exists for a method |

These APIs are from the current `Dhruv125\Coretex\Support\Request` implementation used by OwnWork.

 > next: `reference/response-api.md`
