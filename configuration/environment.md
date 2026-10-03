# Environment Configuration

OwnWork reads application configuration from environment variables.

Environment values are accessed through the framework's environment helper:

```php
env("KEY")
````

 ## `.env`

 Create a `.env` file in the project root:

```
APP_ENV=development
APP_DEBUG=true
APP_URL=http://localhost:8000
```

 Environment variables can then be accessed from application code.

```
$environment = env("APP_ENV");
```

 ## Reading Environment Values

 Use:

```
env("KEY")
```

 For example:

```
$appName = env("APP_NAME");
```

 If the environment contains:

```
APP_NAME=OwnWork
```

 then:

```
env("APP_NAME")
```

 returns the configured value.

 ## Common Environment Values

 A typical application may define values such as:

```
APP_NAME=OwnWork
APP_ENV=development
APP_DEBUG=true
APP_URL=http://localhost:8000
```

 Database or external-service configuration can also be stored in environment variables:

```
DB_HOST=localhost
DB_PORT=3306
DB_DATABASE=ownwork
DB_USERNAME=root
DB_PASSWORD=
```

 The names used by an application are determined by the application's configuration and code.

 ## Environment Variables in Configuration

 Environment values can be used when defining application configuration:

```
$config = [
    "app" => [
        "name" => env("APP_NAME"),
        "environment" => env("APP_ENV"),
        "debug" => env("APP_DEBUG")
    ]
];
```

 This keeps environment-specific values outside the application source code.

 ## Development and Production

 Use different environment values for different environments.

 Development:

```
APP_ENV=development
APP_DEBUG=true
```

 Production:

```
APP_ENV=production
APP_DEBUG=false
```

 Do not place production secrets directly in source-controlled PHP files.

 ## Secrets

 Sensitive values such as passwords, API keys, and credentials should be supplied through environment configuration.

 Example:

```
API_KEY=your-secret-key
DB_PASSWORD=your-database-password
```

 Access them with:

```
$apiKey = env("API_KEY");
$dbPassword = env("DB_PASSWORD");
```

 Do not commit secrets to the application's source repository.

 ## Application Root

 OwnWork uses:

```
approot()
```

 to resolve paths relative to the application root.

 For example:

```
$path = approot() . "/resources/views/";
```

 Environment configuration and application paths should be kept separate: use environment variables for configurable values and `approot()` for project-relative paths.

 ## Example

 `.env`:

```
APP_NAME=My Application
APP_ENV=development
APP_DEBUG=true
APP_URL=http://localhost:8000
```

 Application code:

```
$name = env("APP_NAME");
$environment = env("APP_ENV");
$debug = env("APP_DEBUG");
$url = env("APP_URL");
```

 These values can then be used by the application without changing the source code between environments.

> next: `configuration/helpers.md`
