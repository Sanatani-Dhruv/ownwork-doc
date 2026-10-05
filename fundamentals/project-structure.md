# Project Structure

 An OwnWork project separates the public HTTP entry point, application code, framework bootstrap, source resources, generated files, and Composer dependencies.

 A typical project has the following structure:

```
ownwork/
├── app/
│   ├── Controller/
│   ├── Http/
│   │   └── Kernel.php
│   ├── Middleware/
│   ├── Model/
│   └── Service/
│
├── bundle/
│   ├── Bundler.php
│   ├── Helper.php
│   └── Routes.php
│
├── public/
│   ├── .htaccess
│   ├── build/
│   ├── index.php
│   └── styles/
│
├── resources/
│   ├── appviews/
│   ├── css/
│   ├── js/
│   ├── template/
│   └── views/
│
├── storage/
│   ├── views/
│   └── views.json
│
├── vendor/
│
├── .env
├── .env.example
├── composer.json
├── composer.lock
├── package.json
├── package-lock.json
└── worker
```

 The exact contents can change as application code and generated assets are added.

 ## `app/`

 The `app` directory contains application code and the application's HTTP kernel.

```
app/
├── Controller/
├── Http/
├── Middleware/
├── Model/
└── Service/
```

 ### `app/Controller/`

 Controllers contain application handlers invoked by routes.

 For example:

```
app/Controller/UserController.php
```

 Controllers are ordinary application classes and normally use the `App\Controller` namespace.

 A controller action can receive Coretex request and response objects:

```php
<?php

namespace App\Controller;

use Dhruv125\Coretex\Support\Request;
use Dhruv125\Coretex\Support\Response;

class UserController
{
    public function index(
        Request $request,
        Response $response
    ) {
        return "Users";
    }
}
```

 ### `app/Http/`

 This directory contains the OwnWork HTTP kernel:

```
app/Http/Kernel.php
```

 `Kernel` coordinates request processing, including route registration, route matching, middleware execution, and route resolution.

 ### `app/Middleware/`

 Application middleware is stored here.

 For example:

```
app/Middleware/AuthMiddleware.php
```

 Middleware can inspect a request, terminate processing by returning a response, or continue through the pipeline by calling `$next()`.

 ### `app/Model/`

 Models belong under:

```
app/Model/
```

 OwnWork provides the directory and worker generator for models, but it does not impose an ORM or a specific database implementation.

 For example:

```
app/Model/UserModel.php
```

 The persistence implementation is application-specific.

 ### `app/Service/`

 Application services belong under:

```
app/Service/
```

 Services can keep reusable application operations and business logic separate from controllers.

 For example:

```
app/Service/UserService.php
```

 OwnWork does not require a specific service base class or interface.

 ## `bundle/`

 The `bundle` directory contains application bootstrap, routes, and global helpers.

```
bundle/
├── Bundler.php
├── Helper.php
└── Routes.php
```

 ### `bundle/Bundler.php`

 `Bundler.php` bootstraps the application.

 It validates the basic project setup, loads Composer's autoloader, initializes Coretex's environment and error handling, and starts the OwnWork HTTP kernel.

 The public entry point loads this file before starting the application.

 ### `bundle/Routes.php`

 Application routes are registered here.

 For example:

```php
<?php

$route->get("/", "home.temp.php");
```

 A controller route can also be registered:

```php
<?php

$route->get("/users", [
    UserController::class,
    "index"
]);
```

 The `$route` object is supplied by the OwnWork kernel.

 ### `bundle/Helper.php`

 This file contains OwnWork's global helper functions.

 Examples include:

```php
<?php

approot();
```

```php
<?php

env("APP_NAME");
```

```php
<?php

view("home.temp.php");
```

```php
<?php

comp("button.php");
```

 The helper file is loaded through Composer's file autoload configuration.

 ## `public/`

 The `public` directory is the web-facing directory.

```
public/
├── .htaccess
├── build/
├── index.php
└── styles/
```

 Only files intended to be directly accessible by the web server should normally be placed under `public/`.

 ### `public/index.php`

 This is the application's front controller.

 Its job is to load the bundler and start the application:

```php
<?php

declare(strict_types = 1);

ob_start();

use Bundle\Bundler;

require __DIR__ . "/../bundle/Bundler.php";

$app = new Bundler();
$app->bundle();

ob_end_flush();
```

 ### `public/.htaccess`

 The Apache configuration file is located in the public directory.

 It can be used when deploying OwnWork with Apache for URL handling.

 ### `public/build/`

 This directory contains generated frontend build output.

 ### `public/styles/`

 This directory contains browser-accessible styles.

 The default project includes framework/default CSS resources here.

 ## `resources/`

 The `resources` directory contains source resources used by the application and development tooling.

```
resources/
├── appviews/
├── css/
├── js/
├── template/
└── views/
```

 ### `resources/views/`

 Application view source files are stored here.

 For example:

```
resources/views/home.temp.php
```

 OwnWork supports ordinary PHP views as well as `.temp.php` template views.

 ### `resources/appviews/`

 `appviews` contains views used by OwnWork/Coretex itself, particularly framework error-related pages.

 Examples include:

```
resources/appviews/
├── error_layout.php
├── no-info-error.php
├── stackTrace-block.php
├── script/
└── styles/
```

 These are different from application views under `resources/views/`.

 Applications normally do not need to modify these files unless they intentionally customize the framework's error presentation.

 ### `resources/template/`

 This directory contains templates used by the `worker make` commands.

 For example, generated application components are based on templates such as:

```
resources/template/
├── Controller.php
├── Middleware.php
├── Model.php
├── Service.php
└── View.php
```

 When a developer runs:

```
php worker make controller UserController
```

 the worker uses the corresponding template to generate the application file.

 ### `resources/css/`

 Contains source CSS resources.

 The default project includes the Tailwind CSS source here.

 ### `resources/js/`

 Contains application JavaScript source files.

 Frontend tooling can process these resources into browser-accessible build output.

 ## `storage/`

 The `storage` directory contains generated application files.

 The template system uses:

```
storage/views/
```

 for compiled `.temp.php` templates.

 The view mapping is stored in:

```
storage/views.json
```

 Generated files under `storage/` should generally not be edited as application source code.

 ## `vendor/`

 Composer installs PHP dependencies into:

```
vendor/
```

 The directory contains:

```
vendor/autoload.php
```

 which is loaded during application bootstrap.

 OwnWork's Coretex dependency is also installed through Composer.

 Do not manually edit files inside `vendor/`.

 If a dependency needs to change, modify the Composer configuration and run Composer.

 ## `.env`

 The `.env` file contains environment-specific configuration loaded by Coretex during OwnWork startup.

 For example:

```
APP_NAME=Ownwork

DEV_ENV=true
OWNWORK_ERROR_HANDLER=true
```

 The exact environment variables available to an application depend on the OwnWork/Coretex functionality being used.

 The `.env` file is environment-specific and should not normally be committed when it contains secrets.

 ## `.env.example`

 `.env.example` provides the initial environment configuration template.

 The OwnWork setup command creates `.env` from this file when `.env` does not already exist.

 ## `composer.json`

 `composer.json` defines the PHP package configuration, dependencies, autoloading, and Composer scripts.

 OwnWork declares Coretex as a dependency:

```
dhruv125/coretex
```

 It also defines scripts used for common development operations.

 ## `package.json`

 `package.json` defines the optional Node.js development tooling.

 It is used for frontend development tasks such as JavaScript bundling, Tailwind CSS, and integration with OwnWork's development workflow.

 Node.js is not required for the PHP framework itself.

 ## `worker`

 The `worker` file is OwnWork's command-line manager.

 Run commands with:

```
php worker <command>
```

 Examples include:

```
php worker serve
```

```
php worker make controller UserController
```

```
php worker transpile
```

```
php worker clear:viewcache
```

 The worker is used for development operations, template processing, and application code generation.

 ## Dependency Boundary

 An OwnWork application sits above OwnWork and its Coretex dependency:

```
Application
│
├── app/
├── bundle/
├── public/
├── resources/
└── storage/
        │
        ▼
    OwnWork
        │
        ▼
    Coretex
        │
        ├── Router
        ├── Request
        ├── Response
        ├── View
        ├── Template
        ├── Environment
        └── Error handling
```

 Coretex is installed through Composer and lives under:

```
vendor/
```

 Application code should normally use the framework's public APIs rather than modifying dependency source files.

 ## Source vs Generated Files

 It is useful to distinguish files that developers normally edit from files generated by tooling.

 ### Application source

```
app/
bundle/
resources/views/
resources/css/
resources/js/
```

 ### Framework and application tooling

```
resources/template/
worker
```

 ### Public/generated assets

```
public/build/
public/styles/
```

 ### Generated view output

```
storage/views/
storage/views.json
```

 ### External dependencies

```
vendor/
```

 The distinction is important because source files should be edited directly, while generated output should normally be regenerated by the appropriate OwnWork or Composer command.

> `fundamentals/request-lifecycle.md`
