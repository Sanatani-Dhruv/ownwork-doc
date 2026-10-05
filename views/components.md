## Components

 OwnWork provides a component directive for reusable view elements.

 Components are invoked through:

```
@comp(...)
```

 The directive maps to the application's `comp()` helper.

 ## Basic Usage

```
@comp("button")
```

 The argument identifies the component.

```
@comp("navbar")

@comp("card")
```

 ## Passing Data

 Components can receive data as a second argument:

```
@comp("button", [
    "text" => "Save"
])
```

 For example:

```
@comp("user-card", [
    "user" => $user
])
```

 The supplied values are passed to the component through the `comp()` helper.

 ## Using Components in Templates

 Components can be used alongside normal template syntax:

```
<h1>{{ $title }}</h1>

@foreach($users as $user):
    @comp("user-card", [
        "user" => $user
    ])
@endforeach;
```

 This is useful for repeated presentation elements.

 ## Component Data

 Pass the values required by the component explicitly:

```
@comp("alert", [
    "type" => "error",
    "message" => $message
])
```

 The component receives the supplied data according to the application's component implementation.

 ## Component Organization

 Components belong to the view layer. A project may organize component files separately from page templates, for example:

```
resources/views/
├── pages/
│   └── users.temp.php
└── components/
    ├── alert.php
    ├── button.php
    └── user-card.php
```

 The exact component location and loading behavior depend on how the application's `comp()` helper is configured.

 ## Reusable UI

 Components are intended for reusable presentation elements.

 Instead of repeatedly writing:

```
<button class="button">
    Save
</button>
```

 a template can invoke:

```
@comp("button", [
    "text" => "Save"
])
```

 ## Dynamic Data

 Components can receive values from the current template:

```
@foreach($users as $user):
    @comp("user-card", [
        "user" => $user,
        "showEmail" => true
    ])
@endforeach;
```

 The page supplies the data while the component handles its presentation.

 ## Component Directive Syntax

 The templater recognizes:

```
@comp(...)
```

 and:

```
@comp (...)
```

 The arguments are passed to `comp()`.

 ## Components and Application Logic

 Components should primarily handle presentation.

 Application data should be prepared before rendering the view:

```
$user = $userService->find($id);

return view("users/show.temp.php", [
    "user" => $user
]);
```

 The view can then pass the data to the component:

```
@comp("user-card", [
    "user" => $user
])
```

 Database operations and business logic should remain in the application's service or model layer.

 ## Component vs Include

 An include includes another view file:

```
@include("header.php")
```

 A component invokes reusable UI through the component system:

```
@comp("button", [
    "text" => "Save"
])
```

 They are therefore different mechanisms within the view layer.

 ## Current Limitation

 **Components do not support OwnWork/Coretex templating as of the current version.**

 Components should therefore not be treated as `.temp.php` templates or expected to use OwnWork template directives internally.

 ## Quick Reference

 | Task | Syntax |
| --- | --- |
| Render component | `@comp("name")` |
| Pass component data | `@comp("name", [...])` |
| Component helper | `comp(...)` |
| Include a view | `@include(...)` |

Components provide a reusable presentation mechanism, while the application remains responsible for defining how components are located, loaded, and rendered.

 > `views/transpilation.md`
