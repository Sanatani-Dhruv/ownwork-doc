# Response API

The OwnWork response object represents the HTTP response being produced by the application.

Controller actions can receive the response object as an argument:

```php
public function index(
    Request $request,
    Response $response
) {
    // ...
}
````

 ## Response in Controllers

 A controller commonly receives both request and response objects:

```
public function index(
    Request $request,
    Response $response
) {
    // Handle request and create response
}
```

 The request contains information about the incoming request, while the response is used when constructing the result returned to the client.

 ## Returning a View

 A controller can return a rendered view:

```
return view("home.temp.php");
```

 With data:

```
return view("users/index.temp.php", [
    "users" => $users
]);
```

 The resulting view is returned as the application's HTTP response.

 ## Response Headers

 HTTP response headers can be configured through the response object when required by the application.

 A typical response flow is:

```
Controller
    ↓
Response
    ↓
HTTP Headers
    ↓
HTTP Body
    ↓
Client
```

 ## Response Body

 The response body contains the content returned to the client.

 For a view response:

```
return view("home.temp.php");
```

 the rendered HTML becomes the response content.

 ## JSON Responses

 When an endpoint needs to return structured data, the response should contain the appropriate JSON representation.

 For example, application code may return data such as:

```
$data = [
    "status" => "success",
    "users" => $users
];
```

 The exact JSON response API should follow the response methods available in the installed OwnWork version.

 ## Response Status

 HTTP status codes communicate the result of a request.

 Common status codes include:

```
200 OK
201 Created
204 No Content
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
500 Internal Server Error
```

 Use an appropriate status code for the result being returned.

 ## Error Responses

 A controller can return an error view when an expected resource does not exist:

```
public function show(
    Request $request,
    Response $response
) {
    $user = $this->userService->find(
        $request->param("id")
    );

    if (!$user) {
        return view("errors/404.temp.php");
    }

    return view("users/show.temp.php", [
        "user" => $user
    ]);
}
```

 ## Request and Response Together

 A typical controller action uses both objects:

```
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
```

 The request supplies the route parameter, application logic retrieves the resource, and the controller returns the resulting view.

 ## Response Flow

```
HTTP Request
    ↓
Route
    ↓
Controller
    ↓
Request data
    ↓
Application logic
    ↓
Response
    ↓
HTTP Client
```

 ## Related APIs

- Request API
- Routing API
- Views

> next: `reference/helpers-api.md`
