# Services

Services are application-level classes stored under:

```text
app/Service/
````

 OwnWork does not impose a service interface, base class, dependency-injection container, or persistence architecture.

 A service is simply an application class that can be used to keep reusable operations outside controllers.

 ## Creating a Service

 Use the worker:

```
php worker make service UserService
```

 The worker creates:

```
app/Service/UserService.php
```

 The generator is implemented directly by `worker` and uses:

```
resources/template/Service.php
```

 The current OwnWork service template contains:

```php
<?php

declare(strict_types = 1);

namespace App\Service;

use Dhruv125\Coretex\Viewer\View;

use Dhruv125\Coretex\Support\Request;
use Dhruv125\Coretex\Support\Response;

class DEFAULT_NAME {

    function __construct() {
        // Default Service
    }

    public function index(Request $request, Response $response) {

    }
}
```

 The worker replaces `DEFAULT_NAME` with the requested service name.

 ## Service Location

 Application services belong under:

```
app/Service/
```

 For example:

```
app/
└── Service/
    ├── UserService.php
    ├── OrderService.php
    └── EmailService.php
```

 The namespace is:

```
namespace App\Service;
```

 ## Service Generation

 The worker registers `service` as one of its application component generators.

 The relevant worker mappings are:

```
service
    ↓
app/Service/
    ↓
resources/template/Service.php
```

 The generated file is a normal PHP class.

 OwnWork does not add additional service metadata or registration.

 ## Service Methods

 A service can expose application operations through its methods.

 For example:

```php
<?php

namespace App\Service;

class UserService
{
    public function find($id)
    {
        // Application-specific operation.
    }

    public function create($data)
    {
        // Application-specific operation.
    }
}
```

 The framework does not prescribe the names or responsibilities of these methods.

 ## Using a Service from a Controller

 A controller can use a service directly:

```
use App\Service\UserService;

public function show(
    Request $request,
    Response $response
) {
    $params = $request->getAttribute("dynamicParams");

    $service = new UserService();

    $user = $service->find($params["id"]);

    return $response->json($user);
}
```

 The controller handles HTTP-specific work while the service performs the application operation.

 ## Service and HTTP Objects

 The generated service template currently imports:

```
use Dhruv125\Coretex\Support\Request;
use Dhruv125\Coretex\Support\Response;
```

 and its generated `index()` method accepts them:

```
public function index(
    Request $request,
    Response $response
) {
}
```

 This is the default generated structure.

 However, a service method does not have to use HTTP objects unless the application requires them.

 For reusable application logic, it is generally preferable to pass the required values rather than coupling every operation to the HTTP request.

 For example:

```
public function find($id)
{
    // ...
}
```

 rather than:

```
public function find(Request $request)
{
    // ...
}
```

 when the service only needs an ID.

 ## Service and Models

 Services can coordinate models:

```
Controller
    ↓
Service
    ↓
Model
    ↓
Persistence
```

 For example:

```php
<?php

namespace App\Service;

use App\Model\UserModel;

class UserService
{
    public function find($id)
    {
        $users = new UserModel();

        return $users->find($id);
    }
}
```

 The model remains responsible for application data and persistence operations.

 The service can contain operations that combine multiple model operations or other application dependencies.

 ## Services Without Models

 A service does not require a model.

 For example:

```php
<?php

namespace App\Service;

class EmailService
{
    public function send($recipient, $message)
    {
        // Application-specific implementation.
    }
}
```

 Services can coordinate any application-level operation that benefits from being separated from controllers.

 ## Static Service Methods

 OwnWork does not require services to be instantiated through a framework container.

 If an operation is stateless and does not require instance state, an application may expose it as a static method:

```php
<?php

namespace App\Service;

class UserService
{
    public static function find($id)
    {
        // Application-specific operation.
    }
}
```

 It can then be called directly:

```
$user = UserService::find($id);
```

 Static methods can be convenient for stateless utility-style service operations.

 They are not a special OwnWork feature, however, and the current OwnWork service generator itself generates an instance method:

```
public function index(
    Request $request,
    Response $response
) {
}
```

 Therefore, applications are free to choose between instance methods and static methods according to the service's design.

 ## Constructor Dependencies

 Services can use constructors when instance dependencies are required:

```
class UserService
{
    private $users;

    public function __construct(UserModel $users)
    {
        $this->users = $users;
    }

    public function find($id)
    {
        return $this->users->find($id);
    }
}
```

 The service can then be created by the application:

```
$service = new UserService(
    new UserModel()
);
```

 OwnWork does not provide or require a dependency-injection container for services.

 ## Service and Response Objects

 A service generally does not need to construct an HTTP response when it is being used as an application layer.

 For example:

```
$user = $service->find($id);

return $response->json($user);
```

 This keeps the responsibilities separated:

```
Service
    ↓
Application result

Controller
    ↓
HTTP response
```

 If an application intentionally designs a service around HTTP operations, it can use Coretex's request and response objects as supported by the generated service template.

 ## Service and Route Parameters

 Route parameters belong to the HTTP layer.

 For example:

```
$route->get("/users/{id}", [
    UserController::class,
    "show"
]);
```

 The controller obtains the dynamic parameter:

```
$params = $request->getAttribute(
    "dynamicParams"
);

$id = $params["id"];
```

 The value can then be passed to a service:

```
$user = UserService::find($id);
```

 or:

```
$service = new UserService();

$user = $service->find($id);
```

 The service does not need to know that the value originated from a route.

 ## Service Responsibilities

 Services are useful for application operations such as:

 - coordinating multiple models
- implementing reusable business operations
- coordinating external APIs
- handling application workflows
- keeping controllers small
- providing stateless operations through static methods where appropriate

 The exact responsibility of a service is application-defined.

 ## Controller, Service, and Model

 A common OwnWork application can separate responsibilities as:

```
HTTP Request
     ↓
Controller
     ↓
Service
     ↓
Model
     ↓
Persistence
```

 The boundaries are conventions rather than framework-enforced layers.

 A small application may use:

```
Controller
    ↓
Model
```

 without introducing a service.

 A larger operation may use:

```
Controller
    ↓
Service
    ├── Model
    ├── External API
    └── Other Services
```

 ## Services and External Dependencies

 A service can coordinate external dependencies:

```
OrderService
    ├── OrderModel
    ├── PaymentGateway
    └── NotificationService
```

 For example:

```
public function placeOrder($data)
{
    $order = $this->orders->create($data);

    $this->payment->charge($order);

    $this->notifications->send($order);

    return $order;
}
```

 The implementations of these dependencies are outside OwnWork's service layer.

 ## Service Naming

 A conventional naming scheme uses the `Service` suffix:

```
UserService.php
OrderService.php
EmailService.php
PaymentService.php
```

 with:

```
class UserService
{
}
```

 The suffix is a convention and is not enforced by OwnWork.

 ## Service Exceptions

 Services may throw normal PHP exceptions:

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

 The exception can propagate through the application's normal request and error-handling flow.

 OwnWork does not define a special service exception type.

 ## Service Independence

 A service can be designed independently of the HTTP layer.

 For example:

```
$user = UserService::find($id);
```

 or:

```
$service = new UserService();

$user = $service->find($id);
```

 This allows the same application operation to be reused by controllers, commands, jobs, or other application code without requiring a particular HTTP entry point.

 ## What OwnWork Provides

 For services, OwnWork provides:

 - `app/Service/` as the conventional service directory
- `php worker make service ...`
- `resources/template/Service.php` as the generation template
- Composer autoloading for application classes

 OwnWork does not provide:

 - a service base class
- a service interface
- a required dependency-injection container
- automatic service registration
- a required service architecture
- a required repository pattern
- a required ORM

 These decisions belong to the application.

 ## Service Workflow

 A typical service workflow is:

```
Create service
    ↓
php worker make service UserService
    ↓
Implement application operation
    ↓
Use models or other dependencies when needed
    ↓
Call service from application code
    ↓
Return or use the application result
```

 OwnWork intentionally keeps the service layer minimal. The worker generates the class, while the application decides how services should be structured and used.

> `cli/worker.md`
