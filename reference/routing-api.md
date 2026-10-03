# Routing API

OwnWork routing is built around route definitions that connect HTTP methods and URL paths to controller actions.

## Route Definition

A route is defined by:

- HTTP method
- URL path
- controller
- controller method

A typical route looks like:

```php
Route::get("/", [HomeController::class, "index"]);
````

 The controller method is called when the matching request is received.

 ## HTTP Methods

 Routes can be registered for common HTTP methods:

```
Route::get("/users", [UserController::class, "index"]);
Route::post("/users", [UserController::class, "store"]);
Route::put("/users", [UserController::class, "update"]);
Route::patch("/users", [UserController::class, "update"]);
Route::delete("/users", [UserController::class, "destroy"]);
```

 Use the method that matches the intended HTTP operation.

 ## GET Routes

 Use `get()` for GET requests:

```
Route::get("/users", [UserController::class, "index"]);
```

 ## POST Routes

 Use `post()` for POST requests:

```
Route::post("/users", [UserController::class, "store"]);
```

 ## PUT Routes

 Use `put()` for PUT requests:

```
Route::put("/users/{id}", [UserController::class, "update"]);
```

 ## PATCH Routes

 Use `patch()` for PATCH requests:

```
Route::patch("/users/{id}", [UserController::class, "update"]);
```

 ## DELETE Routes

 Use `delete()` for DELETE requests:

```
Route::delete("/users/{id}", [UserController::class, "destroy"]);
```

 ## Route Paths

 A route path starts with `/`:

```
Route::get("/about", [PageController::class, "about"]);
```

 The path is matched against the incoming request URL.

 ## Dynamic Parameters

 Route parameters can be placed inside `{}`:

```
Route::get("/users/{id}", [UserController::class, "show"]);
```

 For:

```
/users/25
```

 the value of `id` is:

```
25
```

 See Dynamic Parameters.

 ## Controller Actions

 Routes point to controller methods:

```
Route::get("/users", [UserController::class, "index"]);
```

 Here:

```
UserController
      ↓
    index()
```

 is the action executed for the route.

 A controller can return a view:

```
public function index(
    Request $request,
    Response $response
) {
    return view("users/index.temp.php");
}
```

 ## Request and Response

 Controller actions can receive the request and response objects:

```
public function index(
    Request $request,
    Response $response
) {
    // ...
}
```

 The request contains information about the incoming HTTP request.

 The response is used when building an HTTP response.

 See:

 - Request API
- Response API

 ## Route Parameters in Controllers

 A route such as:

```
Route::get("/users/{id}", [UserController::class, "show"]);
```

 can be handled by:

```
public function show(
    Request $request,
    Response $response
) {
    $id = $request->param("id");

    // ...
}
```

 ## Middleware

 Middleware can be attached to routes when authentication, authorization, request processing, or other pre-controller behavior is required.

 Example:

```
Route::get(
    "/admin",
    [AdminController::class, "index"],
    [AuthMiddleware::class]
);
```

 See Middleware.

 ## Route Files

 Keep route definitions in the application's routing configuration.

 A typical route file can contain:

```
Route::get("/", [HomeController::class, "index"]);

Route::get("/users", [UserController::class, "index"]);
Route::get("/users/{id}", [UserController::class, "show"]);

Route::post("/users", [UserController::class, "store"]);
```

 ## Route Organization

 Routes can be grouped by application feature:

```
Route::get("/users", [UserController::class, "index"]);
Route::get("/users/{id}", [UserController::class, "show"]);

Route::get("/posts", [PostController::class, "index"]);
Route::get("/posts/{id}", [PostController::class, "show"]);
```

 ## Route Matching

 When a request arrives, the router checks the registered routes against:

 1. HTTP method
2. requested path
3. dynamic route parameters

 For example:

```
GET /users/25
```

 can match:

```
Route::get("/users/{id}", [UserController::class, "show"]);
```

 with:

```
id = 25
```

 ## Route API Summary

 | API | Purpose |
| --- | --- |
| `Route::get()` | Register a GET route |
| `Route::post()` | Register a POST route |
| `Route::put()` | Register a PUT route |
| `Route::patch()` | Register a PATCH route |
| `Route::delete()` | Register a DELETE route |

## Example

```
Route::get("/", [HomeController::class, "index"]);

Route::get("/users", [UserController::class, "index"]);
Route::get("/users/{id}", [UserController::class, "show"]);

Route::post("/users", [UserController::class, "store"]);

Route::put("/users/{id}", [UserController::class, "update"]);
Route::delete("/users/{id}", [UserController::class, "destroy"]);
```

> next: `reference/request-api.md`
