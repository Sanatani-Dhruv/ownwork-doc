# Error Handling

 OwnWork uses PHP exceptions together with Coretex exceptions and a global error handler for application errors.

 Errors may occur during application startup, routing, request handling, view rendering, or application code.

 ## Global Error Handling

 OwnWork uses Coretex's global error handler for unhandled application errors.

 The handler is provided by:

```
Dhruv125\Coretex\Handler\GlobalErrorHandler
```

 It is responsible for handling errors that are not handled by application code.

 The Coretex package also provides a pager for displaying framework error pages:

```
Dhruv125\Coretex\Pager
```

 ## Error Handler Configuration

 OwnWork's error handling behavior is controlled through environment variables.

 ### OWNWORK_ERROR_HANDLER

 Controls whether OwnWork's error handler is used.

```
OWNWORK_ERROR_HANDLER=true
```

 When enabled, unhandled errors are handled by OwnWork/Coretex.

 When disabled, PHP handles errors normally.

 If `OWNWORK_ERROR_HANDLER` is disabled, `DEV_ENV` does not control the OwnWork error page.

 ### DEV_ENV

 Controls the amount of information shown by the OwnWork error handler.

 Development:

```
DEV_ENV=true
OWNWORK_ERROR_HANDLER=true
```

 Development errors can include detailed information such as the exception location and source-code context.

 Production:

```
DEV_ENV=false
OWNWORK_ERROR_HANDLER=true
```

 Production errors use the default error presentation rather than exposing detailed debugging information.

 ## PHP Exceptions

 Application code can use normal PHP exceptions:

```
try {
    $user = $service->find($id);
} catch (\Exception $error) {
    // Handle the exception.
}
```

 Exceptions can also be allowed to propagate to OwnWork's global error handler.

 For example:

```
public function show(
    Request $request,
    Response $response
) {
    $user = $this->userService->find(
        $request->param("id")
    );

    return view("users/show.temp.php", [
        "user" => $user
    ]);
}
```

 If an unhandled exception occurs during the request, the global error handler can process it.

 ## Coretex Exceptions

 Coretex provides exceptions used by OwnWork and its application lifecycle.

 Important framework exceptions include:

```
InternalErrorException
PageNotFoundException
ViewJsonNotFoundException
ViewNotFoundException
```

 These are located in:

```
Dhruv125\Coretex\Exceptions\
```

 ## PageNotFoundException

 Use `PageNotFoundException` when an application operation determines that a requested page does not exist.

```
throw new PageNotFoundException(
    "Page Not Found"
);
```

 For example:

```
$user = $service->find($id);

if (!$user) {
    throw new PageNotFoundException(
        "Page Not Found"
    );
}
```

 This allows the request to enter OwnWork's normal not-found error handling instead of requiring every controller to construct its own 404 response.

 ## View Exceptions

 The view system provides exceptions for missing view resources.

 ### ViewNotFoundException

 Indicates that a requested view could not be resolved.

```
ViewNotFoundException
```

 ### ViewJsonNotFoundException

 Indicates that required view mapping information could not be resolved.

```
ViewJsonNotFoundException
```

 These exceptions are part of Coretex:

```
Dhruv125\Coretex\Exceptions\
```

 ## Missing Templates

 The templating system can throw a PHP `ErrorException` when a requested template file does not exist.

 For example, attempting to parse a missing template:

```
$template->parse(
    "/path/to/missing.temp.php"
);
```

 can result in:

```
throw new \ErrorException(
    "File '$filename' not found"
);
```

 The resulting exception can then be handled by application code or by the global error handler.

 ## Catching Exceptions

 Applications can catch exceptions when custom handling is required:

```
try {
    $result = $service->execute();
} catch (\Exception $error) {
    // Handle the failure.
}
```

 The exception message is available through:

```
$error->getMessage();
```

 Specific exception classes can be handled separately:

```
try {
    $result = $service->execute();
} catch (ViewNotFoundException $error) {
    // Handle missing view.
} catch (\Exception $error) {
    // Handle other failures.
}
```

 ## Custom Error Pages

 Applications can provide their own error views when custom handling is appropriate.

 For example:

```
resources/views/
└── errors/
    ├── 404.temp.php
    └── 500.temp.php
```

 A controller can explicitly render an error view:

```
return view("errors/404.temp.php");
```

 However, for framework-level not-found handling, applications can use `PageNotFoundException` and allow OwnWork's error handling system to process the exception.

 ## Error Handling Flow

 A normal unhandled exception can follow this flow:

```
Request
   ↓
Route
   ↓
Controller
   ↓
Application operation
   ↓
Exception
   ↓
GlobalErrorHandler
   ↓
Pager / Error Page
   ↓
Response
```

 A not-found condition can follow:

```
Request
   ↓
Route / Controller
   ↓
PageNotFoundException
   ↓
GlobalErrorHandler
   ↓
Not Found response
```

 A missing template can follow:

```
Controller
   ↓
view()
   ↓
View / Template
   ↓
ErrorException
   ↓
GlobalErrorHandler
   ↓
Error response
```

 ## Expected vs Unexpected Errors

 Expected application conditions can be represented with appropriate framework exceptions.

 For example:

```
if (!$user) {
    throw new PageNotFoundException(
        "Page Not Found"
    );
}
```

 Unexpected failures can be allowed to propagate to the global handler:

```
$result = $service->execute();
```

 Or handled explicitly when the application needs custom behavior:

```
try {
    $result = $service->execute();
} catch (\Exception $error) {
    // Custom application handling.
}
```

 ## Error Information

 During development, detailed error information is useful for debugging:

```
DEV_ENV=true
OWNWORK_ERROR_HANDLER=true
```

 In production, avoid exposing internal implementation details:

```
DEV_ENV=false
OWNWORK_ERROR_HANDLER=true
```

 Do not expose sensitive information such as:

 - passwords
- API keys
- database credentials
- private application data
- unnecessary internal paths

 ## Recommended Practice

 Use framework exceptions when the application needs to communicate framework-level conditions:

```
throw new PageNotFoundException(
    "Page Not Found"
);
```

 Use normal PHP exceptions for application-specific failures:

```
throw new \InvalidArgumentException(
    "User ID is required."
);
```

 Allow unexpected exceptions to reach the global error handler unless the application has a specific reason to handle them locally.

 This keeps controllers and services focused on application behavior while OwnWork/Coretex handles unhandled request errors centrally.

 > next: `reference/routing-api.md`
