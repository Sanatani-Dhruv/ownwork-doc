# Environment Configuration

 OwnWork provides environment configuration through environment variables and the `env()` helper.

 ## .env

 Create a `.env` file in the project root:

```
APP_ENV=development
APP_DEBUG=true
APP_URL=http://localhost:8000
```

 Environment values can then be accessed from application code:

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

 ## Environment Values

 Applications can define their own environment variables.

 For example:

```
APP_NAME=OwnWork
APP_ENV=development
APP_DEBUG=true
APP_URL=http://localhost:8000

DB_HOST=localhost
DB_PORT=3306
DB_DATABASE=ownwork
DB_USERNAME=root
DB_PASSWORD=
```

 The variable names are application-defined.

 OwnWork does not require a particular database configuration format.

 ## Configuration by Environment

 Environment variables can be used while building application configuration:

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

 Different environments can use different `.env` values.

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

 Application secrets should not be committed to source control.

 ## Secrets

 Credentials and other sensitive configuration can be supplied through environment variables:

```
API_KEY=your-secret-key
DB_PASSWORD=your-database-password
```

 Access them with:

```
$apiKey = env("API_KEY");
$dbPassword = env("DB_PASSWORD");
```

 Keep sensitive environment files outside version control where appropriate.

 ## Environment and Application Paths

 Environment configuration should not be confused with application-relative paths.

 Use:

```
env("APP_URL");
```

 for configurable environment values.

 Use:

```
approot();
```

 for resolving paths relative to the application root.

 For example:

```
$views = approot() . "/resources/views/";
```

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

 This allows environment-specific configuration without changing application source code.

 > next: `configuration/helpers.md`
