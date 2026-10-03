# Worker CLI

OwnWork provides a `worker` command-line script for common application development tasks.

Run commands from the project root.

## Start the Development Server

```bash
php worker serve
````

 This starts the OwnWork development server on port `8000`.

 Example output:

```
Starting OwnWork server at port:8000...
```

 ## Transpile Views

 Compile `.temp.php` views:

```
php worker transpile
```

 This processes views from:

```
resources/views/
```

 and generates compiled views under:

```
storage/views/
```

 ## Generate a Controller

 Create a controller using:

```
php worker make controller User
```

 Generated controllers are placed in the application's controller directory.

 ## Generate a Model

 Create a model using:

```
php worker make model User
```

 Generated models are placed in the application's model directory.

 ## Generate a View

 Create a view using:

```
php worker make view users/index
```

 The view is created under:

```
resources/views/
```

 and uses the `.temp.php` extension.

 ## `make` Commands

 The `make` command is used to generate application files.

 General syntax:

```
php worker make <type> <name>
```

 Examples:

```
php worker make controller User
php worker make model User
php worker make view users/index
```

 ## Composer Commands

 The same development tasks can also be run through Composer scripts.

 Start development:

```
composer run dev
```

 Transpile views:

```
composer run transpile
```

 ## Typical Development Workflow

 Start the server:

```
php worker serve
```

 In another terminal, transpile the views:

```
php worker transpile
```

 Then edit application files under:

```
app/
resources/views/
bundle/
```

 and transpile views again after changing templates when automatic transpilation is not running.

 ## Command Reference

| Command | Purpose |
| --- | --- |
| `php worker serve` | Start the development server |
| `php worker transpile` | Transpile application views |
| `php worker make controller <name>` | Generate a controller |
| `php worker make model <name>` | Generate a model |
| `php worker make view <name>` | Generate a view |
| `composer run dev` | Start the development workflow |
| `composer run transpile` | Transpile views |

> next: `configuration/environment.md`
