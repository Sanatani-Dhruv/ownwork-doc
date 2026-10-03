# Dynamic Parameters

OwnWork supports dynamic URL segments through the Coretex router.

A dynamic parameter is written inside a route using `{name}` syntax:

```php
$route->get("/users/{id}", [
    UserController::class,
    "show"
]);
````

 For a request such as:

```
GET /users/42
```

 the router matches the route and extracts:

```
[
    "id" => "42"
]
```

 ## Defining a Parameter

 Use curly braces around the parameter name:

```
$route->get("/users/{id}", [
    UserController::class,
    "show"
]);
```

 Here, `id` is the parameter name.

 The parameter name becomes the key in the extracted parameter array.

 ## Multiple Parameters

 A route can contain more than one dynamic parameter:

```
$route->get("/users/{id}/posts/{post}", [
    UserController::class,
    "showPost"
]);
```

 A request:

```
/users/42/posts/7
```

 produces:

```
[
    "id" => "42",
    "post" => "7"
]
```

 Parameters are extracted in the same order in which they appear in the route.

 ## How Matching Works

 Coretex identifies dynamic parameters using the pattern:

```
{parameter}
```

 The router replaces each dynamic segment with the following matching expression:

```
([\w]{1,})
```

 The resulting route expression is anchored at both ends:

```
^...$
```

 This means the complete request path must match the route pattern.

 For example:

```
/users/{id}
```

 is internally converted into a pattern equivalent to:

```
^\/users\/([\w]{1,})$
```

 A request such as:

```
/users/42
```

 matches.

 A request such as:

```
/users/42/profile
```

 does not match this route because the complete URL is required to match.

 ## What Values Can Be Matched

 The dynamic segment uses PHP's `\w` character class.

 Therefore, a dynamic parameter can contain word characters such as:

 - letters
- numbers
- underscore

 For example:

```
/users/dhruv
/users/42
/users/user_42
```

 can match:

```
$route->get("/users/{id}", [
    UserController::class,
    "show"
]);
```

 Characters outside the router's `\w` pattern are not accepted by the dynamic segment.

 ## Parameter Values Are Strings

 The router extracts values directly from the URL.

 For:

```
/users/42
```

 the extracted value is:

```
[
    "id" => "42"
]
```

 The value is a string rather than an automatically converted integer.

 If an application requires a numeric value, it should perform its own validation or conversion:

```
$id = (int) $params["id"];
```

 ## Accessing Parameters

 After a route has been matched, OwnWork makes the extracted parameters available through the request attributes.

 Use:

```
$params = $request->getAttribute("dynamicParams");
```

 For:

```
$route->get("/users/{id}", [
    UserController::class,
    "show"
]);
```

 and:

```
/users/42
```

 the controller can access:

```
$params = $request->getAttribute("dynamicParams");

$id = $params["id"];
```

 ## Controller Example

 A complete controller action can look like:

```
<?php

namespace App\Controller;

use Dhruv125\Coretex\Support\Request;
use Dhruv125\Coretex\Support\Response;

class UserController
{
    public function show(
        Request $request,
        Response $response
    ) {
        $params = $request->getAttribute("dynamicParams");

        $id = $params["id"];

        return $response->json([
            "id" => $id
        ]);
    }
}
```

 With:

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

 can produce:

```
{
    "id": "42"
}
```

 ## Multiple Parameter Example

 Given:

```
$route->get(
    "/users/{user}/posts/{post}",
    [
        UserController::class,
        "post"
    ]
);
```

 the controller can retrieve both values:

```
$params = $request->getAttribute("dynamicParams");

$userId = $params["user"];
$postId = $params["post"];
```

 For:

```
/users/42/posts/7
```

 the values are:

```
[
    "user" => "42",
    "post" => "7"
]
```

 ## Route Pattern vs Parameter Values

 The framework keeps the original route pattern separately from the extracted values.

 For:

```
$route->get("/users/{id}", [
    UserController::class,
    "show"
]);
```

 a successful match contains information equivalent to:

```
[
    "params" => [
        "id" => "42"
    ],
    "currentRoute" => "/users/{id}"
]
```

 The `currentRoute` remains the route definition, while `params` contains the values extracted from the actual request URL.

 ## Parameter Names

 Parameter names are taken directly from the route definition.

 For example:

```
$route->get("/products/{productId}", $handler);
```

 produces:

```
[
    "productId" => "123"
]
```

 The router does not rename parameters or convert them to another naming convention.

 Use the exact name defined in the route:

```
$params["productId"];
```

 ## Parameter Ordering

 The router extracts dynamic values according to their position in the route.

 For:

```
$route->get(
    "/categories/{category}/products/{product}",
    $handler
);
```

 the first dynamic value is assigned to `category` and the second to `product`.

 For:

```
/categories/books/products/42
```

 the resulting parameters are:

```
[
    "category" => "books",
    "product" => "42"
]
```

 ## Static and Dynamic Segments

 Dynamic parameters can be combined with static URL segments.

 For example:

```
$route->get(
    "/users/{id}/profile",
    [
        UserController::class,
        "profile"
    ]
);
```

 matches:

```
/users/42/profile
```

 but does not match:

```
/users/42
```

 or:

```
/users/42/profile/edit
```

 because the complete route must match.

 ## Dynamic Parameters Are Not Optional

 A parameter declared as:

```
/users/{id}
```

 requires a value.

 Therefore:

```
/users/42
```

 matches, while:

```
/users
```

 does not.

 If both forms are required, register separate routes:

```
$route->get("/users", [
    UserController::class,
    "index"
]);

$route->get("/users/{id}", [
    UserController::class,
    "show"
]);
```

 ## No Type Constraints

 OwnWork/Coretex does not provide route syntax for declaring parameter types such as:

```
{id:int}
{id:uuid}
{id:slug}
```

 The dynamic segment is matched using the router's `\w` pattern.

 Application-level validation should therefore be performed after the parameter is extracted.

 For example:

```
$params = $request->getAttribute("dynamicParams");

$id = $params["id"];

if (!ctype_digit($id)) {
    // Handle invalid identifier.
}
```

 ## No Automatic Model Binding

 A dynamic route parameter is only a URL value.

 For:

```
/users/{id}
```

 the router does not automatically load a `User` model.

 The application must perform that lookup itself:

```
$params = $request->getAttribute("dynamicParams");

$id = $params["id"];

// Application-specific model lookup.
```

 This keeps routing separate from the application's data layer.

 ## Route Resolution Result

 Internally, `Route::end()` returns the extracted dynamic parameters as `params`:

```
[
    "middlewares" => [...],
    "handler" => ...,
    "params" => [
        "id" => "42"
    ],
    "currentRoute" => "/users/{id}",
    "routesArray" => [...]
]
```

 OwnWork's HTTP kernel uses this result while resolving the matched route.

 ## Example: User Profile

 Route:

```
$route->get(
    "/users/{id}/profile",
    [
        UserController::class,
        "profile"
    ]
);
```

 Request:

```
GET /users/42/profile
```

 Matched route:

```
/users/{id}/profile
```

 Parameters:

```
[
    "id" => "42"
]
```

 Controller:

```
public function profile(
    Request $request,
    Response $response
) {
    $params = $request->getAttribute("dynamicParams");

    $id = $params["id"];

    // Load the user and render the profile.
}
```

 The router is responsible only for matching and extracting the value. What the application does with that value is the responsibility of the controller/service/model layer.

> next: `routing/middleware.md`
