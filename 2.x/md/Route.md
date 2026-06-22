# Router
---

- Routing defines how to handle incoming HTTP Requests like, what to do when your app hit with a `GET /welcome` request.
- Your Application Routes are defined in `bundle/Routes.php`.
- There Are Some Default Route Set, they look like:

```php
<?php
namespace Bundle;

use App\Helper\Router\Route;
use App\Controller\UserController;

$route = new Route();

$route->get("/", "main.temp.php");

$route->end();
```

## Let's Understand This:

> The class `App\Router\Route` is Your Main Router Class, placed at `app/Router/Route.php`.

1. `Route::get(string $request_uri, $viewName_methodCall, array $arguments)`

- It defines HTTP GET Requests for your App.
- It's Arguments:
   - `$request_uri` : Incoming Url request from client ex. `/welcome`
      - This URL string supports dynamic url matching, read about dynamic URL [HERE](#dynamic-Url).
   - `$viewName_methodCall` : It can accept two type of argument:
      - string: It will be treated as View name. We will Learn about Views later.
      - array: It will be treated as Method call:
         - 0th element would be Full class name (type: string)
         - 1st element would be method name(type: string)
         - 2nd element would be array of arguments going to be passed into method (type: depends on your defined method).
   - `$arguments` : It accepts associative array of elements which will be used as variable's in form key and value pair in the view.

- Example:

```php
<?php
$route->get('/welcome', 'welcome.php', [
   'id' => 15,
   'name' => 'Shyam'
]);

// Or

$route->get('/hello', [
    UserController::class, 'showDetail', [
        'foo' => 'bar',
        'baz' => 'bas'
    ]
]);
```

2. `Route::post(string $request_uri, $viewName_methodCall)`

- It defines HTTP POST Requests for your App.
- It's Working is Similar to `Router::get()`.
- Everything is similar to `Router::get()`, except it is for a POST request for that URL.

3. `Route::redirect(string $oriUrl, string $redirect, bool $fullurl = false)`

- It defines any Redirection you want to make other part of this url or another site.
- It uses `header("Location: $redirect")` under the hood to redirect to that url
- It's Arguments:
   - `$oriUrl` : Incoming Url request from client ex. `/welcome`
   - `$redirect` : Url at which yoh want to redirect to, it can be either in same website or external website:
   - `$fullurl` : This argument must be `true`, if you even want to check for certain get requests, or else it will only use base url without GET query strings. (default: `false`)

- Example:

```php
<?php
# It will only redirect, if user exactly hit `/welcome?redirect=true` endpoint, else not
$route->redirect("/welcome?redirect=true", "/grand/Welcome", true);

# It will redirect, if user hit `/welcome` endpoint. (The Query parameters are ignored)
$route->redirect("/welcome?redirect=true", "/grand/Welcome"); # Redirect to "/grand/Welcome"

$route->redirect("/welcome?nothing=false", "/grand/Welcome"); # Same as above
```

4. `Route::end()`

- This method will actually check if current request matches to endpoint and serve the further views or call provided methods.
- This Method must be called to actually begin registering the Routes defined above. or else they won't be defined as actual routes, so user will get a `404 Not found` page.

## Dynamic Url

- First argument of Method like `Route::get()` and `Route::post()` supports dynamic url matching.
- You can pass on variable name in curly braces to match any string inside that braces and then get value of the matched value.
- Suppose Example:

```php
<?php
# File: bundle/Routes.php
namespace Bundle;

use App\Helper\Router\Route;
use App\Controller\UserController;

$route = new Route();

$route->get("/welcome/{userName}", [
   UserController::class, "index"
]);

$route->end();
```
---
```php
<?php
# File: app/Controller/UserController.php
namespace App\Controller;

use App\Viewer\View;

class UserController {
	private $args; // This will store Dynamic variables Extracted from url
	function __construct($dv) {
		$this->args = $dv;
		// Default Controller
	}

	public function index() {
       print_r($this->args);
	}
}
```

- Here, URL: `/welcome/{userName}` will match any url like:
   - `/welcome/someone`
   - `/welcome/something`
   - `/welcome/somebeing`
   - `/welcome/somename`

- You can access the value of variables like `$userName` as the values of variables of curly braces are passed as array elements into constructor of method.
- I know it is hard to understand in this text adventure, you can take a look at this Code snippet of UserController class.
- The constructor of the class is taking argument `$dv` which is array of dynamically passed values extracted from url.
- Now, if user hits the endpoint `/welcome/Ram`, the output of index method would be:
```bash
# Endpoint: /welcome/Ram
Array
(
   [userName] => Ram
)

# Endpoint: /welcome/Shyam
Array
(
   [userName] => Shyam
)

# Endpoint: /welcome/Xyz123
Array
(
   [userName] => Xyz123
)
```

- Hope you understood by the example. now you can move to understand [Controller](./Controller.md).
