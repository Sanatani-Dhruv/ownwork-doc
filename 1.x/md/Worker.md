# Worker
---

- Worker is Your full time command line helper, which will solve your manual tasks like Creation of Controller `UserController.php`
- Your Worker Script is located at root directory of application `/worker`

## Usage

- Just See what worker offers by running Command as:

```bash
$ php worker
Usage: php worker [OPTION]...
Executing Commands for easy components management - OwnWork

OPTIONS:

make - Making Components, it's arguments:
   > controller - Make Controller File
   > model - Make Model File
   > view - Make View File
```
> You can also run command as `./worker`.
> Make sure to run command from root directory of OwnWork

## Options

1. `make`

- `make` option is used to create components like:
   - Models
   - Controllers
   - Views
- Sub-Options which `make` command accepts are:
   - controller: Making Controller in `app/Controller/`
   - model: Making Model in `app/Model/`
   - view: Making View in `resources/views/`

> Each of This Sub-Option Can Accept Another Argument which would be name of Component,
> Or else the script will ask you to provide a name for your Component interactively

- Worker uses the default template provided in `app/Helper/Template/` for components.
- It's changes template according to Name provided to it.

Suppose you ran command `php worker make controller FormController`, it's output would be:
```bash
$ php worker make controller FormController

Controller with name `FormController` created at:
-> app/Controller/FormController.php

```

the FormController.php would look like

```php
<?php
namespace App\Controller;

use App\Viewer\View;

class FormController {
	function __construct() {
		// Constructor
	}

	public function index() {
		// Base Method
	}
}
```
