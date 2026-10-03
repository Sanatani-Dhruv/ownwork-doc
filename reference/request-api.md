# Request API

The OwnWork request object represents the incoming HTTP request.

Controllers can receive the request object as an argument:

```php
public function index(
    Request $request,
    Response $response
) {
    // ...
}
````

 ## Request Parameters

 Route parameters can be accessed with `param()`.

 Given the route:

```
Route::get("/users/{id}", [UserController::class, "show"]);
```

 the controller can read the `id` parameter:

```
public function show(
    Request $request,
    Response $response
) {
    $id = $request->param("id");
}
```

 For:

```
/users/25
```

 `$request->param("id")` provides:

```
25
```

 ## `param()`

 Get a route parameter:

```
$request->param("id");
```

 The parameter name must match the name used in the route definition.

 Example:

```
Route::get(
    "/posts/{postId}",
    [PostController::class, "show"]
);
```

 Access it with:

```
$postId = $request->param("postId");
```

 ## Request Data

 Request data should be read from the request object according to the type of data being handled by the application.

 For example, a controller can use the request to obtain a route parameter and then pass it to a service:

```
public function show(
    Request $request,
    Response $response
) {
    $id = $request->param("id");

    $user = $this->userService->find($id);

    return view("users/show.temp.php", [
        "user" => $user
    ]);
}
```

 ## Request and Controllers

 A controller action commonly receives both request and response objects:

```
public function index(
    Request $request,
    Response $response
) {
    // Handle request
}
```

 The request is used to inspect information supplied by the client.

 The response is used to construct the HTTP response returned by the application.

 ## Route Parameters

 Dynamic route parameters are defined using braces:

```
Route::get(
    "/users/{id}",
    [UserController::class, "show"]
);
```

 They are available through:

```
$request->param("id");
```

 Multiple parameters can be defined:

```
Route::get(
    "/users/{userId}/posts/{postId}",
    [PostController::class, "show"]
);
```

 The corresponding values can be accessed by name:

```
$userId = $request->param("userId");
$postId = $request->param("postId");
```

 ## Example Controller

```
class UserController
{
    public function show(
        Request $request,
        Response $response
    ) {
        $id = $request->param("id");

        $user = $this->userService->find($id);

        if (!$user) {
            return view("errors/404.temp.php");
        }

        return view("users/show.temp.php", [
            "user" => $user
        ]);
    }
}
```

 ## Request Lifecycle

 The request object is used after a registered route has matched the incoming request:

```
HTTP Request
    ↓
Router
    ↓
Route Parameters
    ↓
Controller
    ↓
Request Object
    ↓
Application Logic
    ↓
Response
```

 ## Quick Reference

| Method | Purpose |
| --- | --- |
| `$request->param("name")` | Get a named route parameter |

## Related APIs

 - Routing API
- Response API

> next: `reference/response-api.md`
