# View Transpilation

 OwnWork transpiles `.temp.php` view templates into PHP files that can be rendered by the application.

 ## Source Views

 Application templates are stored under:

```
resources/views/
```

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

 Only files using the `.temp.php` extension are treated as OwnWork templates.

 ## Compiled Views

 Transpiled views are stored under:

```
storage/views/
```

 Compiled files use the `.c.php` extension.

 For example:

```
storage/views/
└── <generated-name>.c.php
```

 These files are generated artifacts. Application source should remain in `resources/views/`; generated files under `storage/views/` should not normally be edited manually.

 ## Running the Transpiler

 Run the transpiler with:

```
php worker transpile
```

 The Composer equivalent is:

```
composer run transpile
```

 The transpiler scans the application's view directory and processes the `.temp.php` templates it finds.

 ## Recursive View Discovery

 View directories are scanned recursively.

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

 All `.temp.php` files under the view tree can be discovered by the transpiler.

 ## Template Transformation

 The transpiler converts OwnWork template syntax into PHP.

 For example:

```
<h1>{{ $title }}</h1>

@if($users):
    @foreach($users as $user):
        <p>{{ $user["name"] }}</p>
    @endforeach;
@endif;
```

 is transformed into PHP source suitable for execution.

 Escaped output such as:

```
{{ $title }}
```

 is compiled using HTML escaping through `htmlspecialchars()`.

 Template control structures such as `@if` and `@foreach` are converted into their corresponding PHP constructs.

 The transpiler therefore acts as the translation layer between OwnWork's template syntax and executable PHP.

 ## Compiled Filename Generation

 Compiled view filenames incorporate information derived from the source template.

 For example, a source such as:

```
home.temp.php
```

 can produce a compiled file resembling:

```
<hash>_home.<modified-time>.c.php
```

 When the source template changes, its modification time changes, allowing a different compiled filename to be generated.

 ## Template Changes

 During development, after modifying a `.temp.php` file, transpile the views again when automatic transpilation is not being used:

```
php worker transpile
```

 The generated files remain under:

```
storage/views/
```

 ## Clearing Compiled Views

 Compiled view files can be cleared with:

```
php worker clear:viewcache
```

 After clearing the generated views, regenerate them with:

```
php worker transpile
```

 ## Development Workflow

 A typical development workflow is:

```
resources/views/
        │
        │ edit .temp.php
        ▼
    Transpiler
        │
        ▼
storage/views/
        │
        ▼
     view()
        │
        ▼
   HTTP response
```

 The development server can be started with:

```
php worker serve
```

 Composer also provides:

```
composer run dev
```

 and:

```
composer run transpile
```

 ## Source vs Generated Files

 Keep the distinction clear:

 | Purpose | Location |
| --- | --- |
| Source templates | `resources/views/` |
| Template extension | `.temp.php` |
| Generated views | `storage/views/` |
| Compiled extension | `.c.php` |
| Transpile command | `php worker transpile` |
| Clear compiled views | `php worker clear:viewcache` |

Application code should reference the source view name:

```
return view("home.temp.php");
```

 It should not directly reference generated `.c.php` files.

 ## View Transpilation Flow

```
resources/views/*.temp.php
            │
            ▼
       Transpiler
            │
            ▼
       PHP source
            │
            ▼
storage/views/*.c.php
            │
            ▼
          View
            │
            ▼
      HTTP response
```

 ## Important Rules

 - Put application templates in `resources/views/`.
- Use the `.temp.php` extension for OwnWork templates.
- Run `php worker transpile` after template changes when automatic transpilation is not active.
- Treat `storage/views/` as generated output.
- Do not manually edit compiled `.c.php` files.
- Clear generated views with `php worker clear:viewcache` when necessary.
- Render views through the normal `view()` helper rather than referencing compiled files directly.

 > `views/views.md`
