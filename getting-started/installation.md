# Installation

OwnWork is distributed as a Composer project.

## Requirements

OwnWork currently requires:

- PHP `^8.0`
- Composer

The framework declares `dhruv125/coretex` `^1.0` as a runtime dependency.

For the optional frontend development workflow, Node.js and npm can also be used.

## Create a New Project

Create an OwnWork application with Composer:

```bash
composer create-project dhruv125/ownwork my-app
````

 Then enter the project directory:

```
cd my-app
```

 ## Run Setup

 Run the setup script:

```
composer run setup
```

 The setup script performs two operations:

 1. Installs the Composer dependencies.
2. Creates `.env` from `.env.example` if `.env` does not already exist.

 The setup command is defined by the project's Composer configuration:

```
"setup": [
    "composer install",
    "@php -r \"file_exists('.env') || copy('.env.example', '.env');\""
]
```

 ## Verify the Installation

 After setup, the project should contain the Composer autoloader and an environment file:

```
my-app/
├── .env
├── vendor/
│   └── autoload.php
└── ...
```

 OwnWork's application bootstrap checks for both `.env` and `vendor/autoload.php` before starting the application.

 If either is missing, OwnWork stops execution and displays a setup error instructing you to run:

```
composer run setup
```

 ## Start the Development Server

 Start the application with:

```
composer run dev
```

 The default Composer development script starts PHP's built-in development server with:

 - Host: `localhost`
- Port: `8000`
- Document root: `public/`

 The application is therefore available at:

```
http://localhost:8000
```

 You can also start the server directly through the OwnWork worker:

```
php worker serve
```

 The worker uses port `8000` by default.

 To use another port:

```
php worker serve 8080
```

 The server will then use:

```
http://localhost:8080
```

 ## Optional Node.js Setup

 Node.js and npm are optional dependencies.

 They are useful when working with the project's frontend tooling, including the JavaScript and CSS development workflow.

 Install the Node dependencies with:

```
npm install
```

 The exact frontend workflow is described in the development documentation.

 ## Installation Flow

 A typical new OwnWork project can therefore be initialized with:

```
composer create-project dhruv125/ownwork my-app
cd my-app
composer run setup
composer run dev
```

 At this point, the OwnWork application is running through `public/index.php`.

> next: `getting-started/first-app.md`
