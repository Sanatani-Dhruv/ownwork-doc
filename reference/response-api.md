# Response API

 The OwnWork response object is provided by Coretex:

```
use Dhruv125\Coretex\Support\Response;
```

 It stores the response body, HTTP status code, headers, and optional payload, and provides helpers for JSON, HTML, and empty responses.

 ## Response in Controllers

 Controllers can receive a `Response` object:

```
public function index(
    Request $request,
    Response $response
) {
    // ...
}
```

 The response object can be configured and returned from the controller.

```
return $response->json([
    "message" => "Hello"
]);
```

 ## Status Code

 ### setCode()

 Set the HTTP status code:

```
$response->setCode(201);
```

 The method returns the same response object, allowing chaining:

```
return $response
    ->setCode(201)
    ->json([
        "message" => "Created"
    ]);
```

 ### getCode()

 Retrieve the configured status code:

```
$code = $response->getCode();
```

 The default status code is:

```
200
```

 ## Response Body

 ### setBody()

 Set the response body:

```
$response->setBody("Hello World");
```

 It accepts a string and returns the response object.

```
return $response
    ->setCode(200)
    ->setBody("Hello World");
```

 ### getBody()

 Retrieve the current response body:

```
$body = $response->getBody();
```

 ## JSON Responses

 ### json()

 Create a JSON response:

```
return $response->json([
    "message" => "Success",
    "status" => true
]);
```

 A status code can be supplied as the second argument:

```
return $response->json(
    [
        "message" => "Created"
    ],
    201
);
```

 `json()`:

 - sets the response status code
- sets `Content-Type` to `application/json; charset=UTF-8`
- JSON-encodes the supplied array
- stores the encoded value as the response body

 JSON encoding uses `JSON_THROW_ON_ERROR`, so JSON encoding failures throw an exception.

 ## HTML Responses

 ### html()

 Create an HTML response:

```
return $response->html(
    "<h1>Hello</h1>"
);
```

 A status code can be supplied:

```
return $response->html(
    "<h1>Created</h1>",
    201
);
```

 The method sets:

```
Content-Type: text/html; charset=UTF-8
```

 and stores the HTML as the response body.

 ## No Content

 ### noContent()

 Create a `204 No Content` response:

```
return $response->noContent();
```

 This sets:

```
Status: 204
Body: ""
```

 ## Headers

 ### setHeader()

 Set an individual response header:

```
$response->setHeader(
    "X-App-Version",
    "1.0"
);
```

 By default, an existing value is replaced.

```
$response->setHeader(
    "Content-Type",
    "application/json"
);
```

 The third argument controls replacement:

```
$response->setHeader(
    "X-Tag",
    "one",
    false
);

$response->setHeader(
    "X-Tag",
    "two",
    false
);
```

 When `$replace` is `false`, multiple values are stored for the same header.

 ### setHeaders()

 Set multiple headers:

```
$response->setHeaders([
    "Cache-Control" => "no-cache",
    "X-App" => "OwnWork"
]);
```

 It also accepts the `$replace` argument:

```
$response->setHeaders(
    [
        "X-Tag" => "example"
    ],
    false
);
```

 The method returns the response object.

 ### getHeaders()

 Retrieve the configured headers:

```
$headers = $response->getHeaders();
```

 The returned value is an array containing the currently configured response headers.

 ## Content Type

 ### setContentType()

 Set the response content type:

```
$response->setContentType(
    "application/xml"
);
```

 The default charset is UTF-8:

```
application/xml; charset=UTF-8
```

 A different charset can be supplied:

```
$response->setContentType(
    "text/plain",
    "ISO-8859-1"
);
```



 ### isJson()

 Mark the response as JSON:

```
$response->isJson();
```

 This sets:

```
Content-Type: application/json; charset=UTF-8
```

 Pass `false` to remove the `Content-Type` header:

```
$response->isJson(false);
```

 The method returns the response object.

 ## Payload

 The response also maintains a payload array.

 ### setPayload()

 Set multiple payload values:

```
$response->setPayload([
    "status" => "success",
    "message" => "Done"
]);
```

 A single payload value can also be supplied:

```
$response->setPayload(
    "status",
    "success"
);
```

 The method returns the response object.

 A callable can be supplied to filter values when setting an array payload:

```
$response->setPayload(
    [
        "status" => "success",
        "debug" => true
    ],
    "",
    function ($key, $value) {
        return $key !== "debug";
    }
);
```

 ## getPayload()

 Retrieve the current payload:

```
$payload = $response->getPayload();
```

 The payload is an array.

 If the response body is empty when `dispatch()` is called and the payload contains values, Coretex JSON-encodes the payload and uses it as the response body.

 ## Chaining Response Methods

 Response methods return the response object, so they can be chained:

```
return $response
    ->setCode(201)
    ->setHeader(
        "X-Resource",
        "user"
    )
    ->json([
        "message" => "User created"
    ]);
```

 Another example:

```
return $response
    ->setCode(200)
    ->setContentType("text/plain")
    ->setBody("Hello World");
```

 ## Dispatching the Response

 ### dispatch()

 `dispatch()` sends the configured response:

```
$response->dispatch();
```

 It:

 1. sets the HTTP status code
2. sends the configured headers
3. handles special `204` and `304` responses
4. uses the payload as JSON when the body is empty and a payload exists
5. outputs the response body



 A `204 No Content` or `304 Not Modified` response does not output a body.

 ## JSON API Example

```
public function store(
    Request $request,
    Response $response
) {
    $user = [
        "id" => 1,
        "name" => "Dhruv"
    ];

    return $response->json(
        [
            "status" => "success",
            "user" => $user
        ],
        201
    );
}
```

 ## HTML API Example

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

 ## Custom Headers Example

```
public function index(
    Request $request,
    Response $response
) {
    return $response
        ->setHeader(
            "Cache-Control",
            "no-cache"
        )
        ->json([
            "users" => []
        ]);
}
```

 ## Empty Response Example

```
public function destroy(
    Request $request,
    Response $response
) {
    // Delete resource.

    return $response->noContent();
}
```

 ## Response API Summary

 | Method | Purpose |
| --- | --- |
| `$response->isJson()` | Set JSON content type |
| `$response->isJson(false)` | Remove JSON content type |
| `$response->setContentType($type, $charset)` | Set content type |
| `$response->getHeaders()` | Get response headers |
| `$response->setHeader($name, $value, $replace)` | Set one response header |
| `$response->setHeaders($headers, $replace)` | Set multiple response headers |
| `$response->setPayload($payload, $value, $handler)` | Set response payload |
| `$response->getPayload()` | Get response payload |
| `$response->setCode($code)` | Set HTTP status code |
| `$response->getCode()` | Get HTTP status code |
| `$response->setBody($body)` | Set response body |
| `$response->getBody()` | Get response body |
| `$response->json($data, $statusCode)` | Create JSON response |
| `$response->html($html, $statusCode)` | Create HTML response |
| `$response->noContent()` | Create a `204 No Content` response |
| `$response->dispatch()` | Send the configured response |

## Response Flow

```
HTTP Request
     ↓
Route
     ↓
Controller
     ↓
Request
     ↓
Application Logic
     ↓
Response
     ↓
Status + Headers + Body
     ↓
dispatch()
     ↓
HTTP Client
```

 The `Response` class is implemented by the Coretex dependency used by OwnWork. The APIs documented here correspond to the current Coretex implementation rather than response methods from OwnWork's older documentation.

 > next: `reference/helpers-api.md`
