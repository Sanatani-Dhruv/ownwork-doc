# Templating

 OwnWork provides a PHP-based templating system for application views. Templates use the `.temp.php` extension and are stored in:

```
resources/views/
```

 ## Template Files

 Templates are application source files under `resources/views/`.

 A project can organize templates into directories:

```
resources/views/
├── home.temp.php
├── users/
│   ├── index.temp.php
│   └── show.temp.php
└── layouts/
    └── app.temp.php
```

 ## Rendering a View

 Use the `view()` helper:

```
return view("home.temp.php");
```

 Nested templates use their path relative to `resources/views/`:

```
return view("users/index.temp.php");
```

 ## Passing Data

 Pass view data as the second argument:

```
return view("users/index.temp.php", [
    "title" => "Users",
    "users" => $users
]);
```

 The supplied values are available to the template:

```
<h1>{{ $title }}</h1>

@foreach($users as $user):
    <p>{{ $user["name"] }}</p>
@endforeach;
```

 ## Escaped Output

 Use:

```
{{ $value }}
```

 for escaped output.

 Values passed through `{{ }}` are processed with `htmlspecialchars()`.

```
<h1>{{ $title }}</h1>
<p>{{ $message }}</p>
```

 ## Raw Output

 Use:

```
{!{ $value }!}
```

 for raw output:

```
{!{ $html }!}
```

 Raw output should only be used for trusted or intentionally HTML-formatted content.

 ## PHP Blocks

 Templates can contain PHP blocks using `@php`:

```
@php
$name = "OwnWork";
@endphp;
```

 The resulting variable can be used by the template:

```
<h1>{{ $name }}</h1>
```

 ## Conditionals

 ### `@if`

```
@if($user):
    <h1>{{ $user["name"] }}</h1>
@endif;
```

 ### `@else`

```
@if($user):
    <p>User found.</p>
@else:
    <p>User not found.</p>
@endif;
```

 ### `@elseif`

```
@if($role === "admin"):
    <p>Administrator</p>
@elseif($role === "editor"):
    <p>Editor</p>
@else:
    <p>User</p>
@endif;
```

 ## For Loops

 Use `@for` for standard PHP `for` loops:

```
@for($i = 0; $i < 5; $i++):
    <p>{{ $i }}</p>
@endfor;
```

 ## Foreach Loops

 Use `@foreach` for collections:

```
@foreach($users as $user):
    <p>{{ $user["name"] }}</p>
@endforeach;
```

 ## While Loops

```
@while($condition):
    <p>Processing...</p>
@endwhile;
```

 ## Do-While Loops

```
@dowhile:
    <p>Processing...</p>
@enddowhile($condition);
```

 ## Switch Statements

 The template syntax supports `@switch`, `@fcase`, `@case`, and `@default`:

```
@switch($status):>
    @fcase("active"):
        <p>Active</p>
    @case("inactive"):
        <p>Inactive</p>
    @default:
        <p>Unknown</p>
@endswitch;
```

 `@fcase` is used for the first case of a switch.

 The `):>` syntax on `@switch` is intentional. It allows the transpiler to preserve the PHP switch structure before the first case is encountered.

 For example:

```
@switch($value):>
    @fcase("one"):
        <p>One</p>
    @default:
        <p>Other</p>
@endswitch;
```

 ## Includes

 Include a view relative to `resources/views/`:

```
@include("header.php")@
```

 Include only once:

```
@include_once("header.php")@
```

 Require a view:

```
@require("header.php")@
```

 Require only once:

```
@require_once("header.php")@
```

 ## Root Includes

 Use `@includeRoot` for files relative to the application root:

```
@includeRoot("config/example.php")@
```

 The corresponding require form is:

```
@requireRoot("config/example.php")@
```

 These differ from normal view includes because they resolve from the application root rather than the application's view directory.

 ## Transpiled Template Includes

 Use `@@includeTemp()@@` to include another transpiled template:

```
@@includeTemp("header.temp.php")@@
```

 This is intended for `.temp.php` files participating in the OwnWork template compilation process.

 ## Components

 Templates can render components using `@comp()`:

```
@comp("button")@
```

 Component data can be supplied as the second argument:

```
@comp("button", [
    "text" => "Save"
])@
```

 Components are documented separately in:

```
views/components.md
```

 ## Template Comments

 Template comments use:

```
{{--
    This will not be rendered.
--}}
```

 They are removed from the rendered template output.

 ## Template Compilation

 `.temp.php` files are transpiled before execution.

 The source templates are located under:

```
resources/views/
```

 Compiled representations are stored under:

```
storage/views/
```

 The transpiler can be run with:

```
php worker transpile
```

 or:

```
composer run transpile
```

 ## Development

 During template development, the application can be served with:

```
php worker serve
```

 Templates can be transpiled with:

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

 ## Recommended Structure

 Keep application and persistence logic outside templates.

 A typical flow is:

```
Controller
    ↓
Service / Model
    ↓
Data
    ↓
View
    ↓
HTML Response
```

 For example, the controller prepares the data:

```php
<?php
public function index(
    Request $request,
    Response $response
) {
    $users = $this->userService->listUsers();

    return view("users/index.temp.php", [
        "users" => $users
    ]);
}
```

 The template is then concerned primarily with presentation:

```blade - temp.php
<h1>Users</h1>

@foreach($users as $user):
    <article>
        <h2>{{ $user["name"] }}</h2>
    </article>
@endforeach;
```

 ## Quick Reference

| Feature | Syntax |
| --- | --- |
| Escaped output | `{{ $value }}` |
| Raw output | `{!{ $value }!}` |
| Comment | `{{-- ... --}}` |
| PHP block | `@php ... @endphp;` |
| Include | `@include(...)` |
| Include once | `@include_once(...)` |
| Root include | `@includeRoot(...)` |
| Require | `@require(...)` |
| Require once | `@require_once(...)` |
| Root require | `@requireRoot(...)` |
| Transpiled include | `@@includeTemp(...)` |
| Component | `@comp(...)` |
| If | `@if(...)` |
| Else | `@else:` |
| Else-if | `@elseif(...)` |
| End if | `@endif;` |
| For | `@for(...)` |
| End for | `@endfor;` |
| Foreach | `@foreach(...)` |
| End foreach | `@endforeach;` |
| While | `@while(...)` |
| End while | `@endwhile;` |
| Do-while | `@dowhile:` / `@enddowhile(...);` |
| Switch | `@switch(...):>` |
| First case | `@fcase(...)` |
| Case | `@case(...)` |
| Default | `@default:` |
| End switch | `@endswitch;` |

> `views/components.md`
