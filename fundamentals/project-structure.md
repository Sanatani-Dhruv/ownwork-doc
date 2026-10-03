# Project Structure

An OwnWork project separates the public HTTP entry point, application code, framework bootstrap, resources, generated files, and Composer dependencies.

A typical project has the following structure:

```text
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
````

 The exact contents of a project can change as application code and generated assets are added.

 ## `app/`

 The `app` directory contains application code.

```
app/
├── Controller/
├── Http/
├── Middleware/
├── Model/
└── Service/
```

 These directories are intended for the application's controllers, HTTP kernel, middleware, models, and services.

 ### `app/Controller/`

 Controllers contain application handlers that are invoked by routes.

 Example:

```
app/Controller/UserController.php
```

 A generated controller uses the `App\Controller` namespace.

 Controllers commonly receive Coretex request and response objects:

```
public function index(
    Request $request,
    Response $response
) {
    // ...
}
```

 ### `app/Http/`

 The HTTP directory contains the application's HTTP kernel:

```
app/Http/Kernel.php
```

 `Kernel` coordinates the request-processing pipeline, including route loading, route resolution, middleware execution, and response dispatch.

 ### `app/Middleware/`

 Application middleware is stored here.

 Example:

```
app/Middleware/AuthMiddleware.php
```

 Middleware can inspect requests, terminate requests by returning a response, or continue execution with `$next()`.

 ### `app/Model/`

 Models belong in:

```
app/Model/
```

 OwnWork provides the directory and worker generator for models but does not impose a database ORM implementation.

 For example:

```
app/Model/UserModel.php
```

 The actual persistence implementation is application-specific.

 ### `app/Service/`

 Application services belong in:

```
app/Service/
```

 Services are useful for keeping reusable application operations separate from controllers.

 For example:

```
app/Service/UserService.php
```

 OwnWork does not require a specific service base class or interface.

 ## `bundle/`

 The `bundle` directory contains code used to initialize and configure the application.

```
bundle/
├── Bundler.php
├── Helper.php
└── Routes.php
```

 ### `bundle/Bundler.php`

 `Bundler.php` is responsible for bootstrapping OwnWork.

 It loads the Composer autoloader, initializes the environment and error handling, and starts the application kernel.

 The public entry point loads this file before starting the application.

 ### `bundle/Routes.php`

 Application routes are defined here.

 Example:

```
$route->get("/", "home.temp.php");
```

 Controller routes can also be registered:

```
$route->get("/users", [
    UserController::class,
    "index"
]);
```

 ### `bundle/Helper.php`

 This file contains OwnWork's global helper functions.

 Examples include:

```
approot();
```

```
env("APP_NAME");
```

```
view("home.php");
```

```
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

 ### `public/index.php`

 This is the application's front controller.

 Its job is to load the bundler and start the application:

```
require __DIR__ . "/../bundle/Bundler.php";

$app = new Bundler();
$app->bundle();
```

 ### `public/.htaccess`

 The Apache configuration file belongs in the public directory.

 It can be used by Apache deployments for URL handling.

 ### `public/build/`

 This directory contains generated frontend build output.

 ### `public/styles/`

 This directory contains browser-accessible styles.

 The default project includes framework-generated/default CSS resources here.

 Only files intended to be directly accessible by clients should be placed under `public/`.

 ## `resources/`

 The `resources` directory contains source resources used by the application and its development tooling.

```
resources/
├── appviews/
├── css/
├── js/
├── template/
└── views/
```

 ### `resources/views/`

 Application views are stored here.

 Example:

```
resources/views/home.temp.php
```

 OwnWork supports both ordinary PHP views and `.temp.php` template views.

 ### `resources/appviews/`

 These are views used by OwnWork/Coretex itself, particularly error-related pages.

 They are different from application views under `resources/views/`.

 Examples include:

```
resources/appviews/
├── error_layout.php
├── no-info-error.php
├── stackTrace-block.php
├── script/
└── styles/
```

 Applications normally do not need to modify these files unless they intentionally customize framework error presentation.

 ### `resources/template/`

 This directory contains templates used by the `worker make` commands.

 For example, the project includes templates for generated application components.

 Conceptually:

```
resources/template/
├── Controller.php
├── Middleware.php
├── Model.php
├── Service.php
└── View.php
```

 When a developer runs a command such as:

```
php worker make controller UserController
```

 the worker uses the corresponding template to generate the application file.

 ### `resources/css/`

 Contains source CSS resources.

 The default project includes the Tailwind CSS source file here.

 ### `resources/js/`

 Contains application JavaScript source files.

 Frontend build tooling can process these resources into browser-accessible output.

 ## `storage/`

 The storage directory contains generated or runtime files.

 The view system uses:

```
storage/views/
```

 for compiled template output.

 A view mapping file is also used:

```
storage/views.json
```

 The mapping connects template source files with their compiled representations.

 Generated files in `storage/` should generally not be treated as application source code.

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

 OwnWork also installs Coretex under the Composer dependency tree.

 Do not manually edit files inside `vendor/`.

 If dependencies need to change, modify the Composer configuration and run Composer.

 ## `.env`

 The `.env` file contains environment-specific configuration.

 For example:

```
APP_NAME=Ownwork

DEV_ENV=true
OWNWORK_ERROR_HANDLER=true
```

 Environment configuration is loaded during application startup.

 The `.env` file is environment-specific and should not normally be committed when it contains secrets.

 ## `.env.example`

 `.env.example` provides the initial environment configuration template.

 The OwnWork setup command creates `.env` from this file when `.env` does not exist.

 ## `composer.json`

 `composer.json` defines the PHP package configuration, dependencies, autoloading, and Composer scripts.

 OwnWork declares Coretex as a dependency:

```
"dhruv125/coretex": "^1.0"
```

 It also defines scripts for common development operations.

 ## `package.json`

 `package.json` defines the optional Node.js development tooling.

 The project uses it for the frontend development workflow, including JavaScript bundling, Tailwind CSS, and integration with the OwnWork development tools.

 Node.js is not required for the PHP framework itself.

 ## `worker`

 The `worker` file is the OwnWork command-line manager.

 Run commands with:

```
php worker <command>
```

 Examples:

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

 The worker is used for both development tasks and application code generation.

 ## Dependency Boundary

 OwnWork itself and its Coretex dependency have different responsibilities.

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

 Coretex is installed through Composer and lives under `vendor/`.

 Application code should normally interact with the public framework APIs rather than modifying the dependency's source files.

 ## Source vs Generated Files

 It is useful to distinguish source files from generated files.

 ### Application source

```
app/
bundle/
resources/views/
resources/css/
resources/js/
```

 ### Framework/application tooling

```
resources/template/
worker
```

 ### Public/generated assets

```
public/build/
public/styles/
```

 ### Generated runtime files

```
storage/views/
storage/views.json
```

 ### External dependencies

```
vendor/
```

 This separation makes it easier to understand which files should be edited directly and which files are produced by OwnWork or Composer.

> next: `fundamentals/request-lifecycle.md`
