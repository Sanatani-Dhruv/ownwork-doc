# Templating

 OwnWork provides a PHP-based templating system for application views. Templates use the `.temp.php` extension and are stored in `resources/views/`.

 ## Template Files

 Create templates inside:

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

 ## Rendering a View

 Render a template using the `view()` helper:

```
return view("home.temp.php");
```

 A nested view can be rendered using its relative path:

```
return view("users/index.temp.php");
```

 ## Passing Data

 Pass data to a view as the second argument:

```
return view("users/index.temp.php", [
    "title" => "Users",
    "users" => $users
]);
```

 The values are available inside the template:

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

 Example:

```
<h1>{{ $title }}</h1>
<p>{{ $message }}</p>
```

 Values using `{{ }}` are passed through `htmlspecialchars()`.

 ## Raw Output

 Use:

```
{!{ $value }!}
```

 for raw output.

 Example:

```
{!{ $html }!}
```

 Raw output should only be used when the value is already trusted or intentionally contains HTML.

 ## PHP Blocks

 Use `@php` to start a PHP block:

```
@php
$name = "OwnWork";
@endphp;
```

 The variables can then be used in the template:

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

 Use:

```
@while($condition):
    <p>Processing...</p>
@endwhile;
```

 ## Do-While Loops

 Use:

```
@dowhile:
    <p>Processing...</p>
@enddowhile($condition);
```

 ## Switch Statements

 Use `@switch` with `@case` and `@default`:

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

 For a specifying first case of switch statement, use `@fcase`:
 Switch uses unusual end sequence of characters `):>`, because php doesn't allow any html before first case of switch, so we use `):>` to not end the sequence and `$fcase` to handle first case of switch statement:

 Another Example:
```
@switch($value):>
    @fcase("one"):
        <p>One</p>
    @default:
        <p>Other</p>
@endswitch;
```

 ## Includes

 Include another view from `resources/views/`:

```
@include("header.php")
```

 Include a view only once:

```
@include_once("header.php")
```

 Require a view:

```
@require("header.php")
```

 Require a view only once:

```
@require_once("header.php")
```

 ## Root Includes

 Use `@includeRoot` when the file is relative to the application root:

```
@includeRoot("config/example.php")
```

 Similarly:

```
@requireRoot("config/example.php")
```

 ## Transpiled Template Includes

 Use `@@includeTemp()` to include another transpiled template:

```
@@includeTemp("header.temp.php")
```

 This is intended for templates that participate in OwnWork's view compilation system.

 ## Components

 Use `@comp()` to render a component:

```
@comp("button")
```

 Arguments can be passed to the component:

```
@comp("button", [
    "text" => "Save"
])
```

 See Components.

 ## Template Comments

 Use:

```
{{--
    This will not be rendered.
--}}
```

 for template comments.

 ## Template Compilation

 OwnWork compiles `.temp.php` templates before they are rendered.

 Source templates:

```
resources/views/
```

 Compiled templates:

```
storage/views/
```

 Run the transpiler with:

```
php worker transpile
```

 or:

```
composer run transpile
```

 ## Development

 When developing templates, run the development server:

```
php worker serve
```

 and the template transpiler:

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

 Keep application logic outside templates.

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

 For example:

```
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

 Then the template focuses on displaying the data:

```
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
| Include | `@include(...)@` |
| Include once | `@include_once(...)@` |
| Root include | `@includeRoot(...)@` |
| Require | `@require(...)@` |
| Root require | `@requireRoot(...)@` |
| Transpiled include | `@@includeTemp(...)@@` |
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
| Normal Cases | `@case(...)` |
| Default | `@default:` |
| End switch | `@endswitch;` |

> next: `views/components.md`
