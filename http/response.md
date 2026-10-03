# Response

OwnWork uses the `Response` object provided by Coretex to represent the HTTP response returned to the client.

```php
use Dhruv125\Coretex\Support\Response;
````

 A response can contain the HTTP status, headers, and body that are eventually sent back to the browser or API client.

 ## Basic Usage

 A controller can receive the response object:

```
public function index(
    Request $request,
    Response $response
) {
    return $response->html(
        "<h1>Hello World</h1>"
    );
}
```

 The response object is also available to middleware:

```
public static function handle(
    Request $request,
    Response $response,
    callable $next
) {
    return $next();
}
```

 ## HTML Responses

 Use `html()` when the response body should contain HTML:

```
return $response->html(
    "<h1>Hello World</h1>"
);
```

 A controller can therefore return:

```
public function index(
    Request $request,
    Response $response
) {
    return $response->html(
        "<h1>Users</h1>"
    );
}
```

 ## JSON Responses

 Use `json()` for JSON output:

```
return $response->json([
    "message" => "Hello"
]);
```

 For example:

```
public function index(
    Request $request,
    Response $response
) {
    return $response->json([
        "users" => []
    ]);
}
```

 The response is suitable for API-style endpoints.

 ## Response Body

 The response body is the content returned to the HTTP client.

 For example:

```
return $response->html(
    "<h1>Welcome</h1>"
);
```

 produces an HTML response body.

 A JSON response:

```
return $response->json([
    "name" => "Dhruv"
]);
```

 produces a JSON representation of the supplied data.

 ## Response Status

 An HTTP response can contain a status code such as:

```
200 OK
201 Created
400 Bad Request
401 Unauthorized
404 Not Found
500 Internal Server Error
```

 The exact response-building API for changing status codes should follow the `Response` implementation provided by the installed Coretex version.

 When an application needs a specific status code, use the response API rather than writing directly to PHP output.

 ## Headers

 HTTP response headers provide metadata about the response.

 Common examples include:

```
Content-Type
Location
Cache-Control
Authorization
```

 The underlying Coretex response is responsible for managing HTTP response headers.

 Applications should use the response abstraction rather than manually calling PHP's `header()` function for framework-generated responses.

 ## JSON API Example

 A typical API controller can return structured data:

```
public function show(
    Request $request,
    Response $response
) {
    $params = $request->getAttribute(
        "dynamicParams"
    );

    return $response->json([
        "id" => $params["id"],
        "status" => "ok"
    ]);
}
```

 Route:

```
$route->get("/users/{id}", [
    UserController::class,
    "show"
]);
```

 Request:

```
GET /users/42
```

 Response body:

```
{
    "id": "42",
    "status": "ok"
}
```

 ## Returning a View

 A controller does not need to manually construct an HTML response when using OwnWork's view system.

 Instead:

```
return view("users.temp.php");
```

 The view helper renders the application view and the resulting output participates in response processing.

 For example:

```
public function index(
    Request $request,
    Response $response
) {
    return view("users.temp.php", [
        "title" => "Users"
    ]);
}
```

 See Views and Templating.

 ## Returning a String

 A handler can return a string:

```
public function hello()
{
    return "Hello World";
}
```

 For explicit response construction, use the `Response` object:

```
public function hello(
    Request $request,
    Response $response
) {
    return $response->html(
        "Hello World"
    );
}
```

 The explicit form makes the intended HTTP response type clear.

 ## Response in Middleware

 Middleware can terminate a request by returning a response directly:

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

 If the middleware returns the response, the downstream handler is not reached through that middleware chain.

 ## Passing the Response Through Middleware

 When a middleware does not need to terminate the request, it normally calls:

```
return $next();
```

 Example:

```
public static function handle(
    Request $request,
    Response $response,
    callable $next
) {
    // Pre-processing.

    $result = $next();

    // Post-processing.

    return $result;
}
```

 The response generated downstream can therefore pass back through the middleware stack.

 ## Controller Response Flow

 A normal controller request looks like:

```
HTTP Request
     ↓
Route
     ↓
Middleware
     ↓
Controller
     ↓
Response
     ↓
HTTP Client
```

 For a view:

```
HTTP Request
     ↓
Controller
     ↓
view()
     ↓
Template
     ↓
HTML
     ↓
HTTP Response
```

 For an API endpoint:

```
HTTP Request
     ↓
Controller
     ↓
$response->json()
     ↓
JSON
     ↓
HTTP Response
```

 ## Response and Views

 OwnWork's view system is responsible for producing rendered output.

 A view can be returned from a controller:

```
return view("home.temp.php");
```

 The template system processes `.temp.php` files and compiled view output is stored under:

```
storage/views/
```

 The rendered content is then used as part of the HTTP response.

 ## Response and Errors

 Errors and exceptions can result in error responses.

 For example, a missing route can result in a `404` response.

 OwnWork/Coretex provides exceptions including:

```
PageNotFoundException
ViewNotFoundException
ViewJsonNotFoundException
InternalErrorException
```

 The global error handler and application kernel process these failures according to the framework's error-handling configuration.

 See Error Handling.

 ## Response Lifecycle

 The response is created and populated during request processing.

 Conceptually:

```
Request
   ↓
Kernel
   ↓
Route
   ↓
Middleware
   ↓
Handler
   ↓
Response
   ↓
Dispatch
```

 The final response is dispatched after the route handler and middleware pipeline complete.

 ## HTML Example

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
        return $response->html(
            "<h1>Welcome to OwnWork</h1>"
        );
    }
}
```

 Route:

```
$route->get("/", [
    HomeController::class,
    "index"
]);
```

 ## JSON Example

```php
<?php

namespace App\Controller;

use Dhruv125\Coretex\Support\Request;
use Dhruv125\Coretex\Support\Response;

class ApiController
{
    public function status(
        Request $request,
        Response $response
    ) {
        return $response->json([
            "framework" => "OwnWork",
            "status" => "ok"
        ]);
    }
}
```

 Route:

```
$route->get("/api/status", [
    ApiController::class,
    "status"
]);
```

 ## Response Responsibilities

 The response layer is responsible for representing the server's output.

 It covers concepts such as:

 - response body
- response type
- HTTP status
- HTTP headers
- JSON output
- HTML output

 The controller decides what application result should be returned, while the response object provides the HTTP representation of that result.

 ## Response vs Request

 The two objects have opposite roles:

 | Object | Direction | Purpose |
| --- | --- | --- |
| `Request` | Client → Server | Represents incoming HTTP data |
| `Response` | Server → Client | Represents outgoing HTTP data |

Typical controller code uses both:

```
public function index(
    Request $request,
    Response $response
) {
    $name = $request->get("name");

    return $response->json([
        "name" => $name
    ]);
}
```

 The request provides the input and the response provides the output.

 ## Response API Boundary

 The `Response` class used by OwnWork is provided by the Coretex dependency.

 OwnWork integrates the response object into its application kernel, while Coretex provides the lower-level HTTP response implementation.

 Conceptually:

```
OwnWork
├── Kernel
│   └── Request lifecycle
│
└── Coretex
    └── Response
        ├── Body
        ├── Headers
        ├── Status
        ├── HTML
        └── JSON
```

 For methods beyond the response helpers directly used by OwnWork, the installed Coretex version should be treated as the source of truth.

 ## Quick Reference

 | Operation | API |
| --- | --- |
| Create HTML response | `$response->html()` |
| Create JSON response | `$response->json()` |
| Return response from controller | `return $response->...` |
| Return response from middleware | `return $response->...` |
| Continue middleware | `return $next()` |

The response object is the application's main abstraction for producing HTTP output.

> next: `views/views.md`
