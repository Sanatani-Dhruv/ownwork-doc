# Controller
---

- Controllers are the Hands of your OwnWork application which controls the data flow and logic of app between Model and Views.
- Your Application's Controllers are situated in `app/Controller/` directory.
- There is default Controller already provided for you, it looks like:

```php
<?php
namespace App\Controller;

use App\Viewer\View;

class UserController {
	private $args; // This will store Dynamic variables Extracted from url
	function __construct($dv) {
		$this->args = $dv;
		// Default Controller
	}

	public function index() {
		// Base Method
	}
}
```

## Let's Understand This:

> The `App\Controller\UserController` is our Class, which we will use to understand Controller.
> This Class File is placed in `app/Controller/UserController.php`

1. Take a Look at showDetail() method.

- Remember, it's the same method we called from Route.php which is present at `bundle/Routes.php`.

[Know more about Routing](./Route.md)

- 

- Example:

```php
<?php
$route->get('/welcome', 'welcome.php');

// Or

$route->get('/hello', [
    UserController::class, 'ShowHelloWorld', [
        'foo', 'bar'
    ]
]);
```

2. `Route::post(string $request_uri, $viewName_methodCall)`

- It defines HTTP POST Requests for your App.
- It's Working is Similar to `Router::get()`.

# Incomplete Doc
