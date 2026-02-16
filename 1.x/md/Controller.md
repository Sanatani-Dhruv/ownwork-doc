# Controller
---

- Controllers are the Hands of your OwnWork application which moves controls the data flow and logic of app between Model and Views.
- Your Application's Controllers are situated in `app/Controller/` directory.
- There is default Controller already provided for you, it looks like:

```php

<?php
namespace App\Controller;

use App\Viewer\View;

class UserController {
	private $view;
	function __construct() {
		$this->view = new View();
	}

	public function showDetail($name, $id) {
		echo "<title>ShowDetail Page</title>";
		echo "ID is " . ((isset($id)) ? "Set" : "Not Set");
		echo "<br>";
		echo "ID is $id";
	}

	public function welcome() {
		$name = "Shyam";
		View::instantView('welcome.php', [
			'name' => $name
		]);
	}
}
```

## Let's Understand This:

> The `App\Controller\UserController` is our Class, which we will use to understand Controller.
> This Class is placed in `app/Controller/UserController.php`

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
