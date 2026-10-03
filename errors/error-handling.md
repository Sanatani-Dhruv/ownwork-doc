# Error Handling

OwnWork uses PHP exceptions and framework-specific exceptions to report errors.

Errors can occur during application startup, routing, request handling, view rendering, or other application operations.

## PHP Exceptions

OwnWork uses standard PHP exceptions as well as exceptions provided by the framework and its dependencies.

For example, the templating system throws an `ErrorException` when a requested template file does not exist:

```php
throw new \ErrorException("File '$filename' not found");
````

 ## Handling Exceptions

 Application code can use normal PHP exception handling:

```
try {
    // Application operation
} catch (\Exception $error) {
    // Handle the error
}
```

 For example:

```
try {
    $user = $service->find($id);
} catch (\Exception $error) {
    return view("errors/500.temp.php", [
        "message" => $error->getMessage()
    ]);
}
```

 ## View Errors

 The view system can throw errors when a requested view or view-related resource cannot be found.

 One of the view-related exceptions is:

```
ViewNotFoundException
```

 Another is:

```
ViewJsonNotFoundException
```

 These errors indicate that the requested view information could not be resolved.

 ## Missing Template

 When `Template::parse()` receives a file path that does not exist, it throws:

```
\ErrorException
```

 Example:

```
$template->parse("/path/to/missing.temp.php");
```

 results in an error indicating that the file was not found.

 ## Error Messages

 When handling an exception, the exception message can be accessed with:

```
$error->getMessage()
```

 Example:

```
try {
    // ...
} catch (\Exception $error) {
    echo $error->getMessage();
}
```

 ## HTTP Errors

 For HTTP requests, applications should return an appropriate response when an operation fails.

 For example:

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

 ## Error Views

 Applications can create dedicated error views:

```
resources/views/
└── errors/
    ├── 404.temp.php
    └── 500.temp.php
```

 Example:

```
<h1>404</h1>
<p>Page not found.</p>
```

 Then render the view when required:

```
return view("errors/404.temp.php");
```

 ## Development Errors

 During development, keeping detailed error information available can help identify problems.

 A development environment can be configured through environment variables:

```bash
# Show Detailed Error Page with exact error with code snippet at which error occured.
DEV_ENV=true
# Use Ownwork error handler, if this is not true, php will throw errors by itself, and DEV_ENV variable will be ignored.
OWNWORK_ERROR_HANDLER=true
```

 Production environments should use appropriate production settings:

```sh
# Show Default Error Page, like page with only message as '500 Internal Server Error'.
DEV_ENV=false
# Use Ownwork error handler, if this is not true, php will throw errors by itself, and DEV_ENV variable will be ignored.
OWNWORK_ERROR_HANDLER=true
```

 ## Catching Specific Exceptions

 When a specific exception class is available, it can be caught separately:

```
try {
    // ...
} catch (ViewNotFoundException $error) {
    return view("errors/404.temp.php");
} catch (\Exception $error) {
    return view("errors/500.temp.php");
}
```

 This allows different errors to receive different handling.

 ## Logging

 Errors that need investigation should be logged using the application's configured logging solution rather than displayed directly to users.

 Avoid exposing sensitive information such as:

 - passwords
- API keys
- database credentials
- internal file paths
- private application data

 ## Error Handling Flow

 A typical application flow is:

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
Error handling
   ↓
Response
```

 For a view-related error:

```
Controller
   ↓
view()
   ↓
View lookup / rendering
   ↓
ViewNotFoundException
   ↓
Error handling
   ↓
Error response
```

 ## Recommended Practice

 Keep normal application validation separate from unexpected exceptions.

 For expected conditions:

```
if (!$user) {
    // Let OwnWork show the 404 not found default page
    throw new PageNotFoundException("Page Not Found");
    // Or
    // Custom not found page
}
```

 For unexpected failures:

```
try {
    $result = $service->execute();
} catch (\Exception $error) {
    // Log and handle the failure
}
```

 Use specific exception handling where the application needs different behavior for different failure types.

> next: `reference/routing-api.md`
