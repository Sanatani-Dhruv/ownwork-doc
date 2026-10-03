# Request

OwnWork uses the `Request` object from Coretex to represent the current HTTP request.

The request object is passed through the application request lifecycle and is available to middleware and controller handlers.

```php
use Dhruv125\Coretex\Support\Request;
````

 ## Basic Usage

 A controller can receive the request object:

```
public function index(
    Request $request,
    Response $response
) {
    // Use $request here.
}
```

 The request provides access to data associated with the current HTTP request, including request parameters and framework attributes.

 ## Request Attributes

 OwnWork and Coretex use request attributes to store information derived during request processing.

 Read an attribute with:

```
$value = $request->getAttribute("name");
```

 A default value can also be supplied:

```
$value = $request->getAttribute(
    "name",
    null
);
```

 The underlying request object follows the PSR-7 request attribute model.  GitHub

 ## Route Parameters

 Dynamic route parameters are exposed through the `dynamicParams` request attribute.

 Given:

```
$route->get("/users/{id}", [
    UserController::class,
    "show"
]);
```

 a request to:

```
/users/42
```

 produces dynamic parameters equivalent to:

```
[
    "id" => "42"
]
```

 Read them with:

```
$params = $request->getAttribute(
    "dynamicParams"
);

$id = $params["id"];
```

 See Dynamic Parameters.

 ## Current Route

 The request lifecycle also exposes the matched route through the `currentRoute` attribute.

 For:

```
$route->get("/users/{id}", [
    UserController::class,
    "show"
]);
```

 the matched route is:

```
/users/{id}
```

 It can be accessed with:

```
$routePattern = $request->getAttribute(
    "currentRoute"
);
```

 The route pattern and extracted parameter values are separate pieces of request information.

 ## Query Parameters

 Query-string values are available through the request's underlying query data.

 For a request such as:

```
/users?page=2
```

 the query string contains:

```
page=2
```

 Applications can access the request query parameters through the request object provided by Coretex.

 When working with the exact query API exposed by the installed Coretex version, consult the corresponding Coretex `Request` implementation.

 ## Request Data

 OwnWork applications commonly retrieve request input through the request object.

 For example:

```
$name = $request->get("name");
```

 The exact methods available depend on the Coretex version installed by the OwnWork package.

 Do not assume that an API from another PHP framework is available in OwnWork.

 ## Headers

 HTTP headers belong to the request and are available through the underlying request implementation.

 Typical request information includes headers such as:

```
Accept
Content-Type
Authorization
User-Agent
```

 Header handling should use the methods provided by the installed Coretex request implementation.

 ## Cookies

 Cookies are part of the incoming HTTP request.

 When an application needs cookie data, use the request API supplied by Coretex rather than reading `$_COOKIE` directly where possible.

 This keeps HTTP access behind the framework request abstraction.

 ## Server Information

 The request originates from PHP's HTTP/server environment.

 Information such as:

```
REQUEST_METHOD
REQUEST_URI
HTTP_HOST
```

 is used by the framework during request processing.

 Application code should generally use the request abstraction rather than accessing PHP superglobals directly.

 ## Request and Middleware

 Middleware receives the same request object used during route processing:

```
public static function handle(
    Request $request,
    Response $response,
    callable $next
) {
    // Inspect request.

    return $next();
}
```

 This allows middleware to inspect:

 - route information
- request attributes
- incoming request data
- headers
- other HTTP information

 before passing control to the next handler.

 ## Request and Controllers

 Controllers receive the request after routing and middleware processing has established the request context.

 Example:

```
public function show(
    Request $request,
    Response $response
) {
    $params = $request->getAttribute(
        "dynamicParams"
    );

    $id = $params["id"];

    return $response->json([
        "id" => $id
    ]);
}
```

 The controller can combine request data with application services and models.

 ## Request Attributes vs Input

 Request attributes and request input have different purposes.

 ### Request input

 Input comes from the HTTP request itself.

 Examples include:

```
query parameters
form data
request body
```

 ### Request attributes

 Attributes are values attached to the request by the application or framework.

 Examples in OwnWork include:

```
dynamicParams
currentRoute
```

 Conceptually:

```
HTTP Request
├── Input
│   ├── Query
│   └── Body
│
└── Attributes
    ├── dynamicParams
    └── currentRoute
```

 This distinction is important when handling dynamic route parameters.

 ## Reading Multiple Attributes

 The request object can expose its complete attribute collection:

```
$attributes = $request->getAttributes();
```

 A single attribute can be retrieved with:

```
$value = $request->getAttribute(
    "dynamicParams"
);
```

 The underlying PSR-7 request interface defines `getAttributes()` and `getAttribute()` for derived request information.  GitHub

 ## Adding Request Attributes

 The underlying request abstraction supports adding a derived request attribute:

```
$request = $request->withAttribute(
    "key",
    $value
);
```

 This follows the immutable request-message behavior defined by PSR-7.

 Whether an application should add custom attributes depends on where the request is being processed.

 ## Removing Request Attributes

 The underlying request abstraction also supports:

```
$request = $request->withoutAttribute(
    "key"
);
```

 This removes a derived request attribute from the returned request instance.

 ## Request Object and Immutability

 The underlying Coretex request delegates request-message operations to its PSR-7 request implementation.

 Operations such as:

```
withAttribute()
withoutAttribute()
withParsedBody()
```

 return an updated request instance rather than modifying the original PSR-7 message in place.  GitHub

 When using these methods, keep the returned instance:

```
$request = $request->withAttribute(
    "user",
    $user
);
```

 rather than assuming the original request was modified.

 ## Request Lifecycle

 The request object participates in the OwnWork lifecycle:

```
HTTP Request
     ↓
public/index.php
     ↓
Bundler
     ↓
Kernel
     ↓
Request
     ↓
Route Matching
     ↓
Middleware
     ↓
Controller
     ↓
Response
```

 The request is therefore the main object through which request-specific information moves through the application.

 ## Example: Reading Route Data

 Route:

```
$route->get("/products/{id}", [
    ProductController::class,
    "show"
]);
```

 Controller:

```
<?php

namespace App\Controller;

use Dhruv125\Coretex\Support\Request;
use Dhruv125\Coretex\Support\Response;

class ProductController
{
    public function show(
        Request $request,
        Response $response
    ) {
        $params = $request->getAttribute(
            "dynamicParams"
        );

        $id = $params["id"];

        return $response->json([
            "product_id" => $id
        ]);
    }
}
```

 Request:

```
GET /products/25
```

 The controller receives:

```
[
    "id" => "25"
]
```

 through the `dynamicParams` request attribute.

 ## Example: Request in Middleware

```
<?php

namespace App\Middleware;

use Dhruv125\Coretex\Support\Request;
use Dhruv125\Coretex\Support\Response;

class LoggingMiddleware
{
    public static function handle(
        Request $request,
        Response $response,
        callable $next
    ) {
        $route = $request->getAttribute(
            "currentRoute"
        );

        // Perform request-related processing.

        return $next();
    }
}
```

 The middleware can inspect the request without requiring the controller to perform the same infrastructure work.

 ## Direct PHP Superglobals

 PHP exposes incoming HTTP information through superglobals such as:

```
$_GET
$_POST
$_SERVER
$_COOKIE
$_FILES
```

 OwnWork applications should generally prefer the framework's `Request` abstraction for application code.

 For example:

```
$request->getAttribute("dynamicParams");
```

 is preferable to making a controller responsible for reconstructing route parameters from `$_SERVER`.

 This keeps application code independent of the exact PHP request environment.

 ## Request API Boundary

 The `Request` class used by OwnWork is supplied by Coretex.

 OwnWork's kernel creates and passes the request through the application lifecycle, while the lower-level HTTP request operations are implemented by the Coretex request layer.

 This distinction matters when documenting or using request methods:

```
OwnWork
├── Kernel
│   └── Request lifecycle
│
└── Coretex
    └── Request API
```

 The installed Coretex version should be treated as the source of truth for methods not specifically added or used by OwnWork.

 ## Quick Reference

 | Operation | API |
| --- | --- |
| Get request attribute | `$request->getAttribute()` |
| Get all attributes | `$request->getAttributes()` |
| Add attribute | `$request->withAttribute()` |
| Remove attribute | `$request->withoutAttribute()` |
| Get dynamic route parameters | `$request->getAttribute("dynamicParams")` |
| Get matched route | `$request->getAttribute("currentRoute")` |

For the OwnWork-specific routing attributes, see Dynamic Parameters.

> next: `http/response.md`
