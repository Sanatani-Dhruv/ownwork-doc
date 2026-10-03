# View Transpilation

OwnWork transpiles `.temp.php` view files into PHP files that can be rendered by the application.

## Source Views

Application views are stored in:

```text
resources/views/
````

 For example:

```
resources/views/
├── home.temp.php
├── users/
│   ├── index.temp.php
│   └── show.temp.php
└── layouts/
    └── app.temp.php
```

 Only files using the `.temp.php` extension are processed as templates.

 ## Compiled Views

 Compiled views are stored in:

```
storage/views/
```

 The compiled files use the:

```
.c.php
```

 extension.

 For example:

```
storage/views/
└── <generated-name>.c.php
```

 Compiled files are generated automatically. Application code should normally work with the source files in `resources/views/` rather than editing files under `storage/views/`.

 ## Running the Transpiler

 Run:

```
php worker transpile
```

 You can also use:

```
composer run transpile
```

 The transpiler scans the application's views and compiles the `.temp.php` files.

 ## Development

 During development, use:

```
php worker serve
```

 and:

```
php worker transpile
```

 The project also provides:

```
composer run dev
```

 and:

```
composer run transpile
```

 ## How Transpilation Works

 A template such as:

```
<h1>{{ $title }}</h1>

@if($users):
    @foreach($users as $user):
        <p>{{ $user["name"] }}</p>
    @endforeach;
@endif;
```

 is converted into PHP before it is rendered.

 The template syntax is transformed into normal PHP syntax.

 For example:

```
{{ $title }}
```

 becomes an escaped PHP expression using:

```
htmlspecialchars()
```

 and:

```
@if($users):
```

 becomes a PHP `if` block.

 ## Template Changes

 When a `.temp.php` file is changed, its modification time is used when generating the compiled filename.

 For example:

```
home.temp.php
```

 can produce a compiled file similar to:

```
<hash>_home.<modified-time>.c.php
```

 After the source file changes, a new modification time results in a different compiled filename.

 ## Nested Views

 Transpilation scans view directories recursively.

 For example:

```
resources/views/
├── home.temp.php
├── users/
│   ├── index.temp.php
│   └── show.temp.php
└── admin/
    └── dashboard.temp.php
```

 All `.temp.php` files are discovered during the scan.

 ## Generated Files

 Do not place application source templates directly inside:

```
storage/views/
```

 Use:

```
resources/views/
```

 as the source location.

 The `storage/views/` directory contains generated files.

 ## Clearing Compiled Views

 Compiled view files can be cleared using the framework's view cache functionality.

```
php worker clear:viewcache
```

 After clearing the compiled views, run:

```
php worker transpile
```

 to generate them again.

 ## Typical Workflow

 Create or edit:

```
resources/views/home.temp.php
```

 Then transpile:

```
php worker transpile
```

 The resulting compiled view is placed under:

```
storage/views/
```

 The application can then render the view normally:

```
return view("home.temp.php");
```

 ## View Transpilation Flow

```
resources/views/*.temp.php
            ↓
        Transpiler
            ↓
       PHP source
            ↓
     storage/views/*.c.php
            ↓
          View
            ↓
       HTTP response
```

 ## Important Rules

 - Put source templates in `resources/views/`.
- Use the `.temp.php` extension.
- Run the transpiler after template changes when automatic transpilation is not running.
- Do not manually edit generated files in `storage/views/`.
- Keep generated view files out of application source templates.

 ## Quick Reference

 | Task | Command / Path |
| --- | --- |
| Source views | `resources/views/` |
| Template extension | `.temp.php` |
| Compiled views | `storage/views/` |
| Compiled extension | `.c.php` |
| Transpile views | `php worker transpile` |
| Composer transpile | `composer run transpile` |
| Start development server | `php worker serve` |
| Composer development | `composer run dev` |
| Render view | `view("home.temp.php")` |

```

```
