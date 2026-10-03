# Models

Models belong to the application layer and are stored under:

```text
app/Model/
````

 OwnWork provides a model generator through the `worker` command, but it does not impose a database ORM or a specific persistence implementation.

 The model layer is therefore intended to contain application-specific data and persistence logic.

 ## Creating a Model

 Generate a model with:

```
php worker make model UserModel
```

 The generated file is placed under:

```
app/Model/UserModel.php
```

 The generated class uses the `App\Model` namespace.

 A typical generated model has the following structure:

```
<?php

namespace App\Model;

class UserModel
{
    //
}
```

 The generated model is a normal PHP class.

 ## Model Responsibilities

 A model can contain application logic related to an application's data.

 Typical responsibilities include:

 - representing application data
- querying a database
- inserting records
- updating records
- deleting records
- validating data at the model boundary
- encapsulating persistence-related operations

 OwnWork does not require every model to implement a particular interface or extend a framework base class.

 ## No Built-in ORM

 OwnWork is intentionally minimal.

 The framework does not provide a built-in ORM as part of its model layer.

 The `app/Model/` directory is an application convention rather than a complete database abstraction.

 If an application needs database functionality, it can use an external PHP package or its own database implementation.

 The OwnWork README specifically describes additional database functionality as something that can be provided by other packages.  Packagist

 ## Simple Model

 A model can be a plain PHP class:

```
<?php

namespace App\Model;

class UserModel
{
    public function find($id)
    {
        // Application-specific data lookup.
    }
}
```

 The implementation of `find()` depends entirely on the application's persistence layer.

 ## Using a Model from a Controller

 A controller can instantiate a model directly:

```
<?php

namespace App\Controller;

use App\Model\UserModel;
use Dhruv125\Coretex\Support\Request;
use Dhruv125\Coretex\Support\Response;

class UserController
{
    public function show(
        Request $request,
        Response $response
    ) {
        $params = $request->getAttribute(
            "dynamicParams"
        );

        $model = new UserModel();

        $user = $model->find($params["id"]);

        return $response->json($user);
    }
}
```

 The controller handles the HTTP-facing operation while the model handles the application's data operation.

 ## Model and Service Layers

 For larger applications, model operations can be placed behind a service.

 For example:

```
Controller
    ↓
Service
    ↓
Model
    ↓
Database
```

 A controller might call:

```
$user = $userService->find($id);
```

 The service can then delegate persistence work to the model:

```
$user = $userModel->find($id);
```

 This is an application architecture choice rather than a requirement imposed by OwnWork.

 ## Model Generation Template

 The model generator uses the project's model template from:

```
resources/template/Model.php
```

 The worker uses this template when creating a new model.

 This means the generated class can serve as a starting point for application-specific model implementation.

 ## Model Location

 Keep application models under:

```
app/Model/
```

 For example:

```
app/
└── Model/
    ├── UserModel.php
    ├── ProductModel.php
    └── OrderModel.php
```

 The directory is autoloaded through the application's Composer configuration.

 ## Naming

 A common naming convention is to use the `Model` suffix:

```
UserModel.php
ProductModel.php
OrderModel.php
```

 with corresponding class names:

```
class UserModel
{
}

class ProductModel
{
}

class OrderModel
{
}
```

 The framework's generator accepts the requested model name and creates the corresponding PHP file.

 ## Models and Routes

 Models are not registered directly with the router.

 Routes point to handlers such as controllers:

```
$route->get("/users/{id}", [
    UserController::class,
    "show"
]);
```

 The controller can then use a model:

```
Route
  ↓
Controller
  ↓
Model
```

 This keeps URL routing separate from persistence logic.

 ## Models and Dynamic Parameters

 Dynamic route parameters are provided to the controller through the request:

```
$params = $request->getAttribute(
    "dynamicParams"
);

$id = $params["id"];
```

 The controller can pass the value to a model:

```
$user = $userModel->find($id);
```

 The router itself does not perform model lookup or automatic model binding.

 ## Example: Database-backed Model

 A database implementation can be introduced by the application.

 For example:

```
<?php

namespace App\Model;

class UserModel
{
    private $database;

    public function __construct($database)
    {
        $this->database = $database;
    }

    public function find($id)
    {
        // Use the application's database abstraction.
    }

    public function create(array $data)
    {
        // Insert application data.
    }

    public function update($id, array $data)
    {
        // Update application data.
    }

    public function delete($id)
    {
        // Delete application data.
    }
}
```

 The database object and implementation are intentionally left to the application.

 ## Example: Model and Service

 A service can encapsulate application operations:

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

    public function findUser($id)
    {
        return $this->users->find($id);
    }
}
```

 A controller can then use the service:

```
<?php

namespace App\Controller;

use App\Service\UserService;
use Dhruv125\Coretex\Support\Request;
use Dhruv125\Coretex\Support\Response;

class UserController
{
    public function show(
        Request $request,
        Response $response
    ) {
        $params = $request->getAttribute(
            "dynamicParams"
        );

        $service = new UserService();

        $user = $service->findUser(
            $params["id"]
        );

        return $response->json($user);
    }
}
```

 This produces a clear separation:

```
HTTP
 ↓
Controller
 ↓
Service
 ↓
Model
 ↓
Persistence
```

 ## Model Data Validation

 OwnWork does not provide a dedicated model-validation API.

 Applications should validate model input using their chosen validation approach.

 For example:

```
public function create(array $data)
{
    if (empty($data["name"])) {
        throw new \InvalidArgumentException(
            "Name is required."
        );
    }

    // Persist the data.
}
```

 The appropriate validation strategy depends on the application.

 ## Model Errors

 Models may throw exceptions when a persistence operation fails.

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

 The exception can propagate through the controller and request lifecycle where the application's error handling can process it.

 See Error Handling.

 ## Model Independence

 Models do not need to know about HTTP routing.

 A model should not need to access:

```
$route
$request
$response
```

 for ordinary persistence operations.

 Instead, HTTP-specific information should normally be extracted by the controller and passed into the application layer.

 For example:

```
$id = $params["id"];

$user = $userModel->find($id);
```

 rather than making the model responsible for reading the HTTP request.

 ## Recommended Separation

 A typical application can organize responsibilities as follows:

```
app/
├── Controller/
│   └── UserController.php
├── Model/
│   └── UserModel.php
└── Service/
    └── UserService.php
```

 With:

```
UserController
    │
    │ HTTP concerns
    ▼
UserService
    │
    │ application operations
    ▼
UserModel
    │
    │ persistence
    ▼
Database
```

 This separation is not mandatory, but it can help prevent controllers from becoming responsible for every part of an application's data layer.

 ## What OwnWork Provides

 For models, OwnWork provides:

 - `app/Model/` as the conventional model directory
- the `make model` worker command
- a model generation template
- Composer autoloading for application classes

 OwnWork does not provide:

 - an ORM
- automatic model binding
- a required database driver
- a required database schema
- a model base class
- a required repository pattern
- automatic validation

 These capabilities can be added by the application when needed.

 ## Model Workflow

 A typical model workflow is:

```
Create model
    ↓
php worker make model UserModel
    ↓
Implement persistence logic
    ↓
Create service if needed
    ↓
Use service/model from controller
    ↓
Return view or response
```

 This keeps the OwnWork model layer minimal while allowing applications to choose the database and persistence architecture that fits their requirements.

> next: `application/services.md`
