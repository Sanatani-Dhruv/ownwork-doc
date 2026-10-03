# Services

Services are application-level classes used to keep reusable business operations separate from controllers.

OwnWork provides a conventional service directory:

```text
app/Service/
````

 The framework does not require a specific service base class, interface, or dependency-injection pattern.

 ## Creating a Service

 Generate a service with:

```
php worker make service UserService
```

 The generated file is placed under:

```
app/Service/UserService.php
```

 The generated class belongs to the `App\Service` namespace.

 A typical service is a normal PHP class:

```
<?php

namespace App\Service;

class UserService
{
    //
}
```

 ## Why Use Services

 A controller is responsible for handling an HTTP operation. As application logic grows, putting all business logic directly into controllers can make them difficult to maintain.

 A service can provide a separate application-level boundary:

```
HTTP Request
     ↓
Controller
     ↓
Service
     ↓
Model / External API / Other application code
     ↓
Controller
     ↓
HTTP Response
```

 For example, instead of putting user creation logic directly in a controller:

```
public function store(
    Request $request,
    Response $response
) {
    // Validate input.
    // Create user.
    // Send email.
    // Log operation.
    // ...
}
```

 the controller can delegate the operation:

```
public function store(
    Request $request,
    Response $response
) {
    $service = new UserService();

    $user = $service->create(
        $request->get("name")
    );

    return $response->json($user);
}
```

 ## Basic Service

 A service can expose methods for application operations:

```
<?php

namespace App\Service;

class UserService
{
    public function create($name)
    {
        // Application-specific user creation.
    }

    public function find($id)
    {
        // Application-specific user lookup.
    }
}
```

 The service does not need to know how the request reached it.

 ## Service Generation

 The worker service generator uses the project's service template under:

```
resources/template/
```

 The generated class is written to:

```
app/Service/
```

 This follows the same application-code generation approach used for controllers and models.

 ## Using a Service from a Controller

 Import the service:

```
use App\Service\UserService;
```

 Create the service and call its operation:

```
public function show(
    Request $request,
    Response $response
) {
    $params = $request->getAttribute(
        "dynamicParams"
    );

    $service = new UserService();

    $user = $service->find(
        $params["id"]
    );

    return $response->json($user);
}
```

 The controller remains responsible for HTTP concerns while the service handles the application operation.

 ## Services and Models

 A service can delegate persistence operations to a model.

 For example:

```
app/
├── Controller/
│   └── UserController.php
├── Model/
│   └── UserModel.php
└── Service/
    └── UserService.php
```

 The service can use the model:

```
<?php

namespace App\Service;

use App\Model\UserModel;

class UserService
{
    private $users;

    public function __construct()
    {
        $this->users = new UserModel();
    }

    public function find($id)
    {
        return $this->users->find($id);
    }
}
```

 The resulting application flow is:

```
Controller
    ↓
UserService
    ↓
UserModel
    ↓
Persistence
```

 The exact persistence implementation belongs to the application.

 ## Services and Dynamic Parameters

 Dynamic route parameters should normally be extracted at the HTTP boundary.

 For example:

```
$route->get("/users/{id}", [
    UserController::class,
    "show"
]);
```

 The controller retrieves the parameter:

```
$params = $request->getAttribute(
    "dynamicParams"
);

$id = $params["id"];
```

 The service receives the value:

```
$service = new UserService();

$user = $service->find($id);
```

 The service does not need to know that `$id` originally came from a URL.

 ## Services and Request Objects

 Services do not need to receive the HTTP request simply because a controller does.

 Prefer extracting the required data in the controller:

```
$name = $request->get("name");

$service->create($name);
```

 rather than coupling the service directly to HTTP:

```
$service->create($request);
```

 unless the application deliberately wants that design.

 This keeps the service usable outside an HTTP controller.

 ## Services and Responses

 A service should normally return application data or an application result rather than constructing an HTTP response.

 For example:

```
$user = $service->find($id);

return $response->json($user);
```

 Here:

```
Service → application result
Controller → HTTP response
```

 This separation allows the same service operation to be reused by different application entry points.

 ## Example: User Creation

 A service can encapsulate multiple operations:

```
<?php

namespace App\Service;

use App\Model\UserModel;

class UserService
{
    private $users;

    public function __construct()
    {
        $this->users = new UserModel();
    }

    public function create($name, $email)
    {
        if (empty($name)) {
            throw new \InvalidArgumentException(
                "Name is required."
            );
        }

        if (empty($email)) {
            throw new \InvalidArgumentException(
                "Email is required."
            );
        }

        return $this->users->create([
            "name" => $name,
            "email" => $email
        ]);
    }
}
```

 The controller can remain focused on HTTP handling:

```
public function store(
    Request $request,
    Response $response
) {
    $service = new UserService();

    $user = $service->create(
        $request->get("name"),
        $request->get("email")
    );

    return $response->json($user);
}
```

 ## Services and External Dependencies

 A service can also coordinate external application dependencies.

 For example:

```
Controller
    ↓
OrderService
    ├── OrderModel
    ├── PaymentGateway
    └── NotificationService
```

 The service can act as the application-level coordinator:

```
public function placeOrder($data)
{
    $order = $this->orders->create($data);

    $this->payment->charge($order);

    $this->notifications->send(
        $order
    );

    return $order;
}
```

 The actual implementations of these dependencies are application-specific.

 ## Constructor Dependencies

 A service can accept dependencies through its constructor:

```
class UserService
{
    private $users;

    public function __construct(
        UserModel $users
    ) {
        $this->users = $users;
    }
}
```

 It can then be instantiated by the application:

```
$service = new UserService(
    new UserModel()
);
```

 OwnWork does not require a dependency-injection container for services.

 If an application uses a container, it can integrate that container independently.

 ## Services Without Models

 Not every service needs a model.

 For example, a mail-related service could contain:

```
class EmailService
{
    public function send(
        $recipient,
        $message
    ) {
        // Application-specific email operation.
    }
}
```

 The service directory can contain any application-level reusable operation that benefits from being separated from controllers.

 ## Service Naming

 A common naming convention is to use the `Service` suffix:

```
UserService.php
OrderService.php
EmailService.php
PaymentService.php
```

 with corresponding classes:

```
class UserService
{
}

class OrderService
{
}

class EmailService
{
}

class PaymentService
{
}
```

 The suffix is a convention rather than a framework requirement.

 ## Service Exceptions

 Services can throw exceptions when an application operation cannot be completed.

 For example:

```
public function find($id)
{
    if (!$id) {
        throw new \InvalidArgumentException(
            "User ID is required."
        );
    }

    // ...
}
```

 The exception can propagate back through the controller and request lifecycle.

 Application-wide error handling can then handle the exception according to the application's configuration.

 See Error Handling.

 ## Service Testing

 Because a service does not inherently depend on HTTP request or response objects, its operations can be tested independently of the HTTP layer.

 For example, the application can test:

```
$service = new UserService();

$result = $service->create(
    "Dhruv",
    "dhruv@example.com"
);
```

 The exact testing framework is not imposed by OwnWork.

 ## When to Use a Service

 A service is useful when an operation:

 - contains business logic
- is reused by multiple controllers
- coordinates multiple models
- coordinates external dependencies
- would make a controller unnecessarily large
- should be usable independently of HTTP

 For very small operations, a separate service may not be necessary.

 ## Controller vs Service vs Model

 The three layers can be separated by responsibility:

 | Layer | Primary responsibility |
| --- | --- |
| Controller | HTTP input and output |
| Service | Application/business operations |
| Model | Data and persistence operations |

A typical request can therefore look like:

```
Request
   ↓
Controller
   ↓
Service
   ↓
Model
   ↓
Database
   ↓
Model
   ↓
Service
   ↓
Controller
   ↓
Response
```

 This architecture is a convention available to OwnWork applications rather than a mandatory framework structure.

 ## Service Workflow

 A typical workflow is:

```
Create service
    ↓
php worker make service UserService
    ↓
Add application operation
    ↓
Use models or other dependencies
    ↓
Call service from controller
    ↓
Return result from controller
```

 The OwnWork package currently identifies itself as a minimal MVC framework and provides the application directories and generators needed to build this structure, while leaving additional application functionality to the developer.  root.packagist.org

> next: `http/request.md`
