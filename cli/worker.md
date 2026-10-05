# Worker CLI

 OwnWork provides the `worker` script as its command-line manager for application components, development serving, view transpilation, and view-cache management.

 Run commands from the project root.

 ## Command Help

 Running:

```
php worker
```

 prints the available commands and their options.

 The currently registered commands are:

```
make
serve
transpile
clear:viewcache
```

 ## `make`

 The `make` command generates application components from the templates under:

```
resources/template/
```

 General syntax:

```
php worker make <type> <name>
```

 Supported component types are:

```
controller
middleware
service
model
view
```

 For example:

```
php worker make controller UserController
php worker make middleware AuthMiddleware
php worker make service UserService
php worker make model UserModel
php worker make view users/index
```

 The worker creates the corresponding application file using the component's template.

 ## Generate a Controller

```
php worker make controller UserController
```

 The controller is created under:

```
app/Controller/
```

 using:

```
resources/template/Controller.php
```

 ## Generate Middleware

```
php worker make middleware AuthMiddleware
```

 The middleware is created under:

```
app/Middleware/
```

 using:

```
resources/template/Middleware.php
```

 ## Generate a Service

```
php worker make service UserService
```

 The service is created under:

```
app/Service/
```

 using:

```
resources/template/Service.php
```

 ## Generate a Model

```
php worker make model UserModel
```

 The model is created under:

```
app/Model/
```

 using:

```
resources/template/Model.php
```

 ## Generate a View

```
php worker make view users/index
```

 Views are created under:

```
resources/views/
```

 using:

```
resources/template/View.php
```

 The worker creates the requested name as a PHP view file. View templates intended for OwnWork's templating system use the `.temp.php` convention.

 ## Component Names

 The worker takes the component name from the third argument:

```
php worker make controller UserController
```

 Internally, the worker uses that name when creating the file and replaces:

```
DEFAULT_NAME
```

 in the corresponding template.

 If a component name is not supplied, the worker prompts for it interactively.

 ## Existing Components

 The worker checks whether the target file already exists.

 If it does, the worker does not overwrite it and reports that the component already exists.

 ## Start the Development Server

 Run:

```
php worker serve
```

 The default port is:

```
8000
```

 The worker starts PHP's built-in development server with:

```
--server=localhost:8000
--docroot=public
```

 A different port can be supplied:

```
php worker serve 8080
```

 The resulting server uses:

```
http://localhost:8080
```

 The default command therefore corresponds to:

```
http://localhost:8000
```

 ## Transpile Views

 Run:

```
php worker transpile
```

 The worker uses Coretex's templater to scan and compile OwnWork view templates.

 Source templates are stored in:

```
resources/views/
```

 Compiled views are stored in:

```
storage/views/
```

 The worker also maintains:

```
storage/views.json
```

 which contains the mapping used by the view system.

 ## Continuous Transpilation

 Without a numeric argument or `build`, the transpiler runs as a watcher.

```
php worker transpile
```

 It waits for view changes and retranspiles when changes are detected.

 The worker checks the view state periodically and clears the view cache before recompiling changed templates.

 ## Production Transpilation

 The transpiler accepts a numeric argument specifying how many transpilation operations should occur:

```
php worker transpile 1
```

 A special `build` argument performs a production-style build:

```
php worker transpile build
```

 The build operation:

```
scan storage
    ↓
scan resources
    ↓
compile views
    ↓
write storage/views.json
```

 Unlike the normal watcher mode, the build operation exits after compilation.

 ## Clear View Cache

 Compiled view files can be removed with:

```
php worker clear:viewcache
```

 The command clears files inside:

```
storage/views/
```

 It does not remove the source templates under:

```
resources/views/
```

 After clearing the cache, views can be regenerated with:

```
php worker transpile
```

 ## Composer Development Commands

 The project also defines Composer scripts.

 Development server:

```
composer run dev
```

 View transpilation:

```
composer run transpile
```

 The `dev` Composer script starts PHP's development server on port `8000`.

 The `transpile` script invokes:

```
php worker transpile
```

 ## npm Development Workflow

 When the optional Node.js tooling is installed, the project also provides an npm development workflow.

 Install dependencies:

```
npm i
```

 Then:

```
npm run dev
```

 The npm development workflow can run the project's development tooling together rather than manually starting each process.

 ## Command Reference

 | Command | Purpose |
| --- | --- |
| `php worker` | Display worker help |
| `php worker make controller <name>` | Generate a controller |
| `php worker make middleware <name>` | Generate middleware |
| `php worker make service <name>` | Generate a service |
| `php worker make model <name>` | Generate a model |
| `php worker make view <name>` | Generate a view |
| `php worker serve` | Start the development server on port `8000` |
| `php worker serve <port>` | Start the development server on a custom port |
| `php worker transpile` | Watch and transpile views |
| `php worker transpile <number>` | Run a limited number of transpilation operations |
| `php worker transpile build` | Build compiled views and exit |
| `php worker clear:viewcache` | Clear compiled view files |
| `composer run dev` | Start the Composer development server |
| `composer run transpile` | Run the view transpiler |
| `npm run dev` | Run the npm development workflow |

## Worker Flow

 The worker provides the main development operations around the application:

```
                    worker
                      │
       ┌──────────────┼──────────────┐
       ↓              ↓              ↓
     make           serve        transpile
       │              │              │
       ↓              ↓              ↓
 app components   PHP server     resources/views
                                  │
                                  ↓
                            storage/views
```

 The worker itself is a lightweight command manager; application behavior remains in the generated application classes and the underlying OwnWork/Coretex components.

 > `configuration/environment.md`
