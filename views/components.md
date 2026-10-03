# Components

OwnWork provides a component directive for reusable view elements.

Components are rendered using:

```php
@comp(...)
````

 This directive maps to the application's `comp()` helper.

 ## Basic Usage

 A component can be called with:

```
@comp("button")
```

 The first argument identifies the component.

 For example:

```
@comp("navbar")
```

 or:

```
@comp("card")
```

 ## Passing Data

 Additional arguments can be passed to a component:

```
@comp("button", [
    "text" => "Save"
])
```

 A component can therefore receive configuration or data from the view.

 For example:

```
@comp("user-card", [
    "user" => $user
])
```

 ## Components in Templates

 Components can be used alongside normal template syntax:

```
<h1>{{ $title }}</h1>

@foreach($users as $user):
    @comp("user-card", [
        "user" => $user
    ])
@endforeach;
```

 This allows repeated UI structures to be extracted from larger templates.

 ## Component Data

 Pass any required values explicitly:

```
@comp("alert", [
    "type" => "error",
    "message" => $message
])
```

 The component receives the supplied values through the `comp()` helper.

 ## Components and Views

 Components are part of the view layer.

 A typical structure can be:

```
resources/views/
├── pages/
│   └── users.temp.php
└── components/
    ├── alert.php
    ├── button.php
    └── user-card.php
```

 The exact component organization depends on how the application's `comp()` helper is configured.

 ## Reusable UI

 Components are useful for UI that appears in multiple places.

 Instead of repeating the same markup:

```
<button class="button">
    Save
</button>
```

 an application can use:

```
@comp("button", [
    "text" => "Save"
])
```

 The component can then be reused wherever the same UI is required.

 ## Components with Dynamic Data

 Components can receive variables from the current template:

```
@foreach($users as $user):
    @comp("user-card", [
        "user" => $user,
        "showEmail" => true
    ])
@endforeach;
```

 This keeps the page template responsible for supplying data while the component handles its presentation.

 ## Component Directive Syntax

 The templater recognizes both forms:

```
@comp(...)
```

 and:

```
@comp (...)
```

 The arguments are passed to the application's `comp()` function.

 ## Components and Application Logic

 Components should primarily contain presentation logic.

 Prepare application data before calling the component:

```
$user = $userService->find($id);

return view("users/show.temp.php", [
    "user" => $user
]);
```

 Then pass the prepared data:

```
@comp("user-card", [
    "user" => $user
])
```

 Keep database operations and business rules in services or models rather than inside UI components.

 ## Example

 Controller:

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

 View:

```
<h1>Users</h1>

@foreach($users as $user):
    @comp("user-card", [
        "user" => $user
    ])
@endforeach;
```

 The component receives the individual user and is responsible for rendering its UI.

 ## Component vs Include

 Both components and includes can be used to reuse view code, but they serve different purposes.

 Use an include when you want to include another view:

```
@include("header.php")
```

 Use a component when you want to invoke reusable UI through the component system:

```
@comp("button", [
    "text" => "Save"
])
```

 ## Quick Reference

 | Task | Syntax |
| --- | --- |
| Render component | `@comp("name")` |
| Pass component data | `@comp("name", [...])` |
| Component helper | `comp(...)` |
| Include a view | `@include(...)` |

Components provide a reusable presentation layer for OwnWork applications.

> Note: Components don't support templating as of current ownwork and coretex version.

> next: `views/transpilation.md`
