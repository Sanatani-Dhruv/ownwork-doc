# Development

OwnWork includes a development workflow for running the PHP application, transpiling template views, and rebuilding frontend assets while files change.

## Development Server

The simplest way to start the application is:

```bash
composer run dev
````

 The development server uses PHP's built-in web server with `public/` as the document root.

 The default server address is:

```
http://localhost:8000
```

 You can also start the server directly:

```
php worker serve
```

 To use a different port:

```
php worker serve 8080
```

 The worker passes the selected port to PHP's built-in server and uses:

```
public/
```

 as the document root.

 ## Why `public/` Is the Document Root

 OwnWork uses:

```
public/index.php
```

 as its front controller.

 Application source code, templates, configuration, and other project files remain outside the web document root.

 The request flow therefore begins with:

```
Browser
   ↓
public/index.php
   ↓
Bundler
   ↓
Kernel
   ↓
Router
   ↓
Middleware
   ↓
Route handler
   ↓
Response
```

 ## View Development

 OwnWork's `.temp.php` files are template files rather than ordinary PHP views.

 During development, the framework transpiles these templates into PHP files under:

```
storage/views/
```

 Run the view transpiler with:

```
php worker transpile
```

 The development transpiler watches the view source directory and processes template changes.

 This means that when you modify a `.temp.php` file, the compiled representation can be regenerated without manually compiling every template.

 ## Build View Templates

 For a build-oriented transpilation pass:

```
php worker transpile build
```

 Unlike the normal development mode, the build mode performs the compilation operation and exits rather than continuously watching for changes.

 ## Clear Compiled Views

 If compiled templates become stale, clear the generated view cache:

```
php worker clear:viewcache
```

 Compiled views are stored under:

```
storage/views/
```

 The template mapping is stored in:

```
storage/views.json
```

 After clearing the cache, the templates can be transpiled again.

 ## Node.js Development Workflow

 OwnWork also defines an npm development command.

 Install frontend dependencies:

```
npm install
```

 Then run:

```
npm run dev
```

 The npm development workflow starts multiple development processes for the application.

 These include:

 - the PHP development server
- the Tailwind CSS watcher
- the OwnWork template transpiler
- the JavaScript bundler/watch process

 The exact commands are defined in the project's `package.json`.

 ## Development Environment

 The `.env` file controls environment-specific behavior.

 A typical development environment contains:

```
DEV_ENV=true
```

 When development mode is enabled, Coretex configures PHP's error display for development.

 OwnWork can also enable its global error handler through:

```
OWNWORK_ERROR_HANDLER=true
```

 Keep development-oriented settings separate from production configuration.

 ## Recommended Development Loop

 A typical development workflow is:

```
1. Start the development environment
2. Edit routes/controllers/views
3. Let the relevant watcher process changes
4. Refresh the browser
5. Inspect errors and responses
6. Clear compiled views if necessary
```

 For PHP-only work:

```
composer run dev
```

 For template development:

```
php worker transpile
```

 For the integrated frontend workflow:

```
npm install
npm run dev
```

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

 Generated files are based on the templates in:

```
resources/template/
```

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
├── css/
├── js/
├── template/
└── views/

storage/
└── views/
```

 The application code belongs primarily under `app/`, routes and bootstrap configuration under `bundle/`, browser-accessible files under `public/`, source templates/assets under `resources/`, and generated view output under `storage/`.

> next: `fundamentals/architecture.md`
