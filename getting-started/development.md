# Development

 OwnWork provides a development workflow for running the PHP application, transpiling `.temp.php` views, and building frontend assets.

 ## Development Server

 The simplest way to start the PHP application is:

```
composer run dev
```

 The `dev` Composer script starts PHP's built-in development server with:

 - Host: `localhost`
- Port: `8000`
- Document root: `public/`

 The default server address is:

```
http://localhost:8000
```

 The same server can be started directly through the OwnWork worker:

```
php worker serve
```

 To use another port:

```
php worker serve 8080
```

 The worker passes the selected port to PHP's built-in server and continues to use `public/` as the document root.

 ## Why `public/` Is the Document Root

 OwnWork uses:

```
public/index.php
```

 as its front controller.

 Application source code, templates, configuration, and other project files remain outside the web document root.

 A request therefore begins with:

```
Browser
   ↓
public/index.php
   ↓
Bundler
   ↓
Environment configuration
   ↓
Global error handler
   ↓
Kernel
   ↓
Route matching
   ↓
Middleware
   ↓
Route handler
   ↓
Response
```

 `public/index.php` creates `Bundler` and calls `bundle()`.

 `Bundler` first checks that `.env` and `vendor/autoload.php` exist. It then creates Coretex's `Environment`, loads the environment file, configures PHP's error reporting, and initializes Coretex's `GlobalErrorHandler` before starting the OwnWork `Kernel`.

 ## Environment Configuration

 OwnWork loads environment configuration from the root `.env` file.

 A new project is initialized with `.env.example` during:

```
composer run setup
```

 The current example environment file contains:

```
APP_NAME=Ownwork

DB_DRIVER=sql
DB_NAME=mariadb
DB_HOST=127.0.0.1
DB_USER=root
DB_PASS=

DEV_ENV=true

OWNWORK_ERROR_HANDLER=true
ERROR_PAGE_LOCATION=resources/appviews/
```

 The environment file is loaded by Coretex's `Environment::setenv()` implementation. The environment loader reads:

```
.env
```

 from the application root and makes its values available through `$_ENV`.

 ### `DEV_ENV`

 `DEV_ENV` controls development error display.

 When:

```
DEV_ENV=true
```

 Coretex enables PHP error and startup-error display and sets PHP error reporting to:

```
E_ALL ^ E_DEPRECATED
```

 When development mode is not enabled, Coretex does not enable the development error display settings.

 ### `OWNWORK_ERROR_HANDLER`

 `OWNWORK_ERROR_HANDLER` controls whether OwnWork's global error and exception handler is registered.

 When it is enabled, Coretex's `GlobalErrorHandler` registers:

 - a PHP error handler
- a PHP exception handler

 The handler writes errors and exceptions to:

```
storage/error.log
```

 It then displays either a development error page or a generic `500 Internal Server Error`, depending on `DEV_ENV`.

 ### `ERROR_PAGE_LOCATION`

 The example environment file contains:

```
ERROR_PAGE_LOCATION=resources/appviews/
```

 OwnWork's `GlobalErrorHandler` currently uses:

```
resources/appviews/
```

 as the location of its internal error view components. The current handler initializes this location directly from the application root rather than reading `ERROR_PAGE_LOCATION` from `$_ENV`.

 Therefore, `ERROR_PAGE_LOCATION` should not currently be documented as an active configurable error-page path.

 ## Error Handling During Development

 When `OWNWORK_ERROR_HANDLER=true`, OwnWork registers Coretex's global error handler.

 For PHP errors, the handler logs the error and, when `DEV_ENV=true`, displays a development error page containing information such as:

 - error message
- source file
- source line
- relevant source code
- environment values
- stack information

 For uncaught exceptions, the development error page can also display the exception trace and source context.

 Errors and exceptions are logged to:

```
storage/error.log
```

 When `DEV_ENV` is not enabled, the handler responds with HTTP status `500` and displays the application's generic error page from:

```
resources/appviews/no-info-error.php
```

 if that file exists. Otherwise it displays:

```
500 Internal Server Error
```

 The global error handler is implemented by Coretex and initialized by OwnWork's `Bundler`.

 > **Development warning:** the development error handler can expose source code, stack traces, and environment values. Do not enable development error display in a production environment.

 ## View Development

 OwnWork uses `.temp.php` files as template source files.

 These templates are transpiled into generated PHP files under:

```
storage/views/
```

 Run the development transpiler with:

```
php worker transpile
```

 The transpiler continuously checks the view source directory for changes.

 When changes are detected, it clears the generated view cache, recompiles the templates, and updates:

```
storage/views.json
```

 The `storage/views.json` file maps source template paths to their compiled view files.

 The source templates remain in:

```
resources/views/
```

 and the generated PHP files are stored in:

```
storage/views/
```

 ## Build View Templates

 For a one-time production-oriented transpilation pass, run:

```
php worker transpile build
```

 The `build` argument causes the worker to:

 1. Scan the existing view storage.
2. Scan the source views.
3. Compile the template files.
4. Write the compiled-view mapping to `storage/views.json`.
5. Exit.

 Unlike normal transpile mode, it does not continue watching for changes.

 ## Clear Compiled Views

 To remove generated compiled view files, run:

```
php worker clear:viewcache
```

 This clears files inside:

```
storage/views/
```

 It does not remove the source templates in:

```
resources/views/
```

 The worker's clear operation currently removes the compiled files but does not remove `storage/views.json`. If the view mapping also needs to be regenerated, run the transpiler afterward:

```
php worker transpile build
```

 or:

```
php worker transpile
```

 ## Node.js Development Workflow

 Node.js and npm are optional for the PHP framework itself.

 The project also includes frontend tooling for CSS and JavaScript.

 Install the frontend dependencies with:

```
npm install
```

 The current `package.json` defines these frontend commands:

```
npm run tw:dev
```

 for the Tailwind CSS development watcher,

```
npm run tw:build
```

 for the Tailwind CSS build,

```
npm run js:run
```

 for the JavaScript development/watch process, and:

```
npm run js:build
```

 for the JavaScript build.

 The project also provides:

```
npm run dev
```

 as the integrated frontend development command.

 The exact process configuration should be treated as defined by the project's current `package.json`.

 ## Recommended Development Loop

 A typical development workflow is:

```
1. Start the PHP development server.
2. Start the view transpiler when working on `.temp.php` views.
3. Start the frontend watcher when working on CSS or JavaScript.
4. Edit routes, controllers, middleware, views, or application code.
5. Refresh the browser.
6. Inspect the response and development error output.
7. Clear generated views if compiled output becomes stale.
```

 For PHP application development:

```
composer run dev
```

 For view development:

```
php worker transpile
```

 For frontend development:

```
npm install
npm run dev
```

 The PHP server, view transpiler, and frontend processes are separate processes unless you use a project-level command that combines them.

 ## Generating Application Code

 The `worker` command can generate common application components.

 Controller:

```
php worker make controller UserController
```

 Middleware:

```
php worker make middleware AuthMiddleware
```

 Model:

```
php worker make model UserModel
```

 Service:

```
php worker make service UserService
```

 View:

```
php worker make view users
```

 The worker supports these component types:

 - `controller`
- `middleware`
- `model`
- `service`
- `view`

 Generated files are created from templates under:

```
resources/template/
```

 and placed in:

```
app/Controller/
app/Middleware/
app/Model/
app/Service/
resources/views/
```

 If a component with the requested filename already exists, the worker does not overwrite it.

 ## Project Development Structure

 During development, the important directories are:

```
app/
├── Controller/
├── Http/
├── Middleware/
├── Model/
└── Service/

bundle/
├── Bundler.php
├── Helper.php
└── Routes.php

public/
└── index.php

resources/
├── appviews/
├── css/
├── js/
├── template/
└── views/

storage/
└── views/
```

 Their roles are:

 - `app/` — application classes such as controllers, HTTP kernel code, middleware, models, and services.
- `bundle/` — OwnWork bootstrap code, helpers, and application route configuration.
- `public/` — the web-facing front controller and public files.
- `resources/appviews/` — internal application error-page components used by the global error handler.
- `resources/template/` — templates used by the worker when generating application components.
- `resources/views/` — source `.temp.php` view templates.
- `resources/css/` — frontend CSS source files.
- `resources/js/` — frontend JavaScript source files.
- `storage/views/` — generated PHP view files.

 The view transpiler also maintains:

```
storage/views.json
```

 as the mapping between source templates and their compiled files.

 The source of truth for views is `resources/views/`; files under `storage/views/` are generated output.

 ## Development Commands at a Glance

| Task | Command |
| --- | --- |
| Start PHP development server | `composer run dev` |
| Start PHP server directly | `php worker serve` |
| Start server on another port | `php worker serve 8080` |
| Watch and transpile views | `php worker transpile` |
| Build/transpile views once | `php worker transpile build` |
| Clear compiled views | `php worker clear:viewcache` |
| Install frontend dependencies | `npm install` |
| Start integrated frontend workflow | `npm run dev` |
| Watch Tailwind CSS | `npm run tw:dev` |
| Build Tailwind CSS | `npm run tw:build` |
| Run JavaScript development process | `npm run js:run` |
| Build JavaScript | `npm run js:build` |

> `fundamentals/architecture.md`
