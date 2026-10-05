# Installation

 OwnWork is distributed as a Composer project.

 ## Requirements

 OwnWork currently requires:

 - PHP `^8.0`
- Composer

 The framework declares `dhruv125/coretex` `^1.0` as a runtime dependency.

 Node.js and npm are optional. They are required only when using OwnWork's frontend development/build tooling for Tailwind CSS and JavaScript.

 ## Create a New Project

 Create an OwnWork application with Composer:

```
composer create-project dhruv125/ownwork my-app
```

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

 1. Runs `composer install` to install the project's Composer dependencies.
2. Creates `.env` from `.env.example` if `.env` does not already exist.

 The setup command is defined in the project's Composer configuration:

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

 OwnWork's `Bundler` checks for both `.env` and `vendor/autoload.php` before starting the application.

 If either is missing, OwnWork stops execution and displays a setup error instructing you to run:

```
composer run setup
```

 ## Start the Development Server

 The Composer development command starts PHP's built-in development server:

```
composer run dev
```

 It uses:

 - Host: `localhost`
- Port: `8000`
- Document root: `public/`

 The application is therefore available at:

```
http://localhost:8000
```

 The same server can be started directly through the OwnWork worker:

```
php worker serve
```

 The worker uses port `8000` by default.

 To use another port:

```
php worker serve 8080
```

 The application will then be available at:

```
http://localhost:8080
```

 The `serve` command invokes PHP's built-in server with `public/` as the document root.

 ## Optional Node.js Setup

 Node.js and npm are optional for the PHP framework itself.

 They are used by the project's frontend tooling, including:

 - Tailwind CSS development and production builds
- JavaScript bundling through esbuild
- Running the frontend development workflow alongside the PHP server and OwnWork view transpiler

 Install the Node dependencies with:

```
npm install
```

 The project defines the following frontend commands:

```
npm run tw:dev
npm run tw:build
npm run js:run
npm run js:build
```

 The combined frontend/build workflow is described in the development documentation.

 ## Installation Flow

 A basic OwnWork application can therefore be initialized and started with:

```
composer create-project dhruv125/ownwork my-app
cd my-app
composer run setup
composer run dev
```

 For projects using the frontend tooling, install the Node dependencies separately:

```
npm install
```

 At runtime, the application starts from `public/index.php`, which bootstraps OwnWork through its `Bundler`.

 > `getting-started/first-app.md`
