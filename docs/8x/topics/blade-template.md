## &#10162; Blade Template:

Blade is a simple, powerful templating engine in Laravel. All Blade templates are compiled into plain PHP code and cached until changed. Blade template files uses `.blade.php` extension and are stored in the `resources/views` directory.

Blade views are returned from routes or controllers using the global `view` helper. Data is passed to the Blade view using the `view` helper's second argument:

### &#9780; Overview:

1. [Displaying Data](#-displaying-data)
    - [HTML Entity Encoding](#-html-entity-encoding)
    - [Display Unescaped Data](#-display-unescaped-data)
    - [Blade and JavaScript Frameworks](#-blade-and-javascript-frameworks)
    - [Render JSON in Blade](#-render-json-in-blade)
    - [Verbatim Directive](#-verbatim-directive)
2. [Blade Directives](#-blade-directives)
    - [If Statements](#-if-statements)
    - [Authentication Directives](#-authentication-directives)
    - [Environment Directives](#-environment-directives)
    - [Section Directives As If Statements](#-section-directives-as-if-statements)
    - [Switch Statements](#-switch-statements)
    - [Loops](#-loops)
    - [The Loop Variable](#-the-loop-variable)
    - [Conditional Classes](#-conditional-classes)
    - [Including Subviews](#-including-subviews)
    - [Rendering Views For Collections](#-rendering-views-for-collections)
    - [The @once Directive](#-the-once-directive)
    - [Raw PHP](#-raw-php)
    - [Comments](#-comments)
3. [Components](#-components)
    - [Manually Registering Package Components](#-manually-registering-package-components)
    - [Rendering Components](#-rendering-components)
    - [Passing Data To Components](#-passing-data-to-components)
    - [Casing](#-casing)
    - [Escaping Attribute Rendering](#-escaping-attribute-rendering)
    - [Component Methods](#-component-methods)
    - [Accessing Attributes and Slots Within Component Classes](#-accessing-attributes-and-slots-within-component-classes)
    - [Additional Dependencies](#-additional-dependencies)
    - [Hiding Attributes or Methods](#-hiding-attributes-or-methods)
    - [Component Attributes](#-component-attributes)
    - [Default or Merged Attributes](#-default-or-merged-attributes)
    - [Conditionally Merge Classes](#-conditionally-merge-classes)
    - [Non-Class Attribute Merging](#-non-class-attribute-merging)
    - [Retrieving and Filtering Attributes](#-retrieving-and-filtering-attributes)
    - [Reserved Keywords](#-reserved-keywords)
    - [Slots](#-slots)
        - [Scoped Slots](#-scoped-slots)
        - [Slot Attributes](#-slot-attributes)
    - [Inline Component Views](#-inline-component-views)
    - [Generating Inline View Components](#-generating-inline-view-components)
    - [Anonymous Components](#-anonymous-components)
        - [Anonymous Index Components](#-anonymous-index-components)
    - [Data Properties or Attributes](#-data-properties-or-attributes)
    - [Accessing Parent Data](#-accessing-parent-data)
    - [Dynamic Components](#-dynamic-components)
    - [Manually Registering Components](#-manually-registering-components)
    - [Autoloading Package Components](#-autoloading-package-components)
4. [Building Layouts](#-building-layouts)
    - [Defining Layout Component](#-defining-layout-component)
    - [Applying The Layout Component](#-applying-the-layout-component)
    - [Layouts Using Template Inheritance](#-layouts-using-template-inheritance)
    - [Extending A Layout](#-extending-a-layout)
5. [Forms](#-forms)
    - [CSRF Field](#-csrf-field)
    - [Method Field](#-method-field)
    - [Validation Errors](#-validation-errors)
6. [Stacks](#-stacks)
7. [Service Injection](#-service-injection)
8. [Extending Blade](#-extending-blade)
    - [Custom Echo Handlers](#-custom-echo-handlers)
    - [Custom If Statements](#-custom-if-statements)

### &#10022; Displaying Data:

Display data passed to Blade views by wrapping the variable in curly braces. For example, given this route:

```php
Route::get('/', function () {
    return view('welcome', ['name' => 'Kumar']);
});
```

Display the contents of the name variable like this:

```php
Welcome, {{ $name }}.
```

This {{ }} curly braces are echo statements with PHP's `htmlspecialchars` to prevent XSS attacks.

It is possible to `echo` the results of any PHP function. Put any PHP code inside a Blade echo statement:

```php
The current UNIX timestamp {{ time() }}.
```

### &#10022; HTML Entity Encoding:

HTML entities are special character to display symbols in HTML.

By default, Blade will double encode HTML entities. To disable double encoding, call the `Blade::withoutDoubleEncoding` method from the boot method of the `AppServiceProvider`.

```php


namespace App\Providers;

use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::withoutDoubleEncoding();
    }
}
```

### &#10022; Display Unescaped Data:

By default, Blade {{ }} statements use PHP's `htmlspecialchars` to prevent XSS attacks. To display data without escaping, use this syntax:

```php
Hello, {!! $name !!}.
```

Be careful when echoing user-supplied content. Use the escaped, double curly brace syntax to prevent XSS attacks when displaying user data.

### &#10022; Blade and JavaScript Frameworks:

Since many JavaScript frameworks use "curly" braces to show an expression, use the @ symbol to tell the Blade rendering engine to leave an expression alone. For example:

```php
<h1>Laravel</h1>

Hello, @{{ name }}.
```

In this example, Blade removes the @ symbol, but leaves the {{ name }} expression, So the JavaScript framework can render it.

The @ symbol can also escape Blade directives:

```php
{{-- Blade template --}}
@@if()
```

Result:

```
@if()
```

### &#10022; Render JSON in Blade:

Sometimes, It required to pass an array to the view to render it as JSON for initializing a JavaScript variable. 

*Example:*

```php
<script>
    var app =  echo json_encode($array); ?>;
</script>
```

Instead of manually calling `json_encode`, use the `Illuminate\Support\Js::from` method directive. The `from` method takes the same arguments as PHP's `json_encode` function, but ensures the resulting JSON is properly escaped for HTML quotes. The `from` method returns a string `JSON.parse` JavaScript statement that converts the given object or array into a valid JavaScript object:

```php
<script>
    var app = {{ Illuminate\Support\Js::from($array) }};
</script>
```

The latest Laravel application skeleton versions include a `Js` facade, providing easy access to this functionality in Blade templates:

```php
<script>
    var app = {{ Js::from($array) }};
</script>
```

Only use the `Js::from` method to render existing variables as JSON. Blade templating is based on regular expressions, and passing a complex expression to the directive may cause unexpected failures.

### &#10022; Verbatim Directive:

If you're displaying JavaScript variables in a large part of the template, wrap the HTML in the `@verbatim` directive to avoid prefixing each Blade echo statement with an @ symbol:

```php
@verbatim
    <div class="container">
        Hello, {{ name }}.
    </div>
@endverbatim
```

### &#10022; Blade Directives:

Blade provides shortcuts for common PHP control structures, like conditional statements and loops, in addition to template inheritance and data display. These shortcuts offer a clean, concise way to work with PHP control structures while staying familiar to their PHP equivalents.

### &#10022; If Statements:

Use `@if`, `@elseif`, `@else`, and `@endif` directives to make if statements. These directives work like their PHP counterparts:

```php
@if (count($records) === 1)
    I have one record!
@elseif (count($records) > 1)
    I have multiple records!
@else
    I don't have any records!
@endif
```

Blade also provides an `@unless` directive:

```php
@unless (Auth::check())
   Unauthorized access, Please login
@endunless
```

Use the `@isset` and `@empty` directives as shortcuts for their PHP functions:

```php
@isset($records)
    // $records is defined and may not null...
@endisset

@empty($records)
    // $records is "empty"...
@endempty
```

### &#10022; Authentication Directives:

The `@auth` and `@guest` directives quickly check if the current user is authenticated or a guest:

```php
@auth
    // The user is authenticated...
@endauth

@guest
    // The user is not authenticated...
@endguest
```

Specify the authentication guard to check with `@auth` and `@guest` directives:

```php
@auth('admin')
    // The user is authenticated...
@endauth

@guest('admin')
    // The user is not authenticated...
@endguest
```

### &#10022; Environment Directives:

Check if the application is running in the production environment using the `@production` directive:

```php
@production
    // Production specific content...
@endproduction
```

Or, check if the application is running in a specific environment using the `@env` directive:

```php
@env('staging')
    // The application is running in "staging"...
@endenv

@env(['staging', 'production'])
    // The application is running in either "staging" or "production"
@endenv
```

### &#10022; Section Directives As If Statements:

Check if a template inheritance section has content using the `@hasSection` directive:

```php
@hasSection('navigation')
    <div class="pull-right">
        @yield('navigation')
    </div>

    <div class="clearfix"></div>
@endif
```

Use the `@sectionMissing` directive to check if a section does not have content:

```php
@sectionMissing('navigation')
    <div class="pull-right">
        @include('default-navigation')
    </div>
@endif
```

### &#10022; Switch Statements:

Switch statements are made using the `@switch`, `@case`, `@break`, `@default`, and `@endswitch` directives:

```php
@switch($i)
    @case(1)
        First case...
        @break

    @case(2)
        Second case...
        @break

    @default
        Default case...
        @break
@endswitch
```

### &#10022; Loops:

Blade offers simple directives for PHP's loop structures, working just like their PHP counterparts:

```php
@for ($i = 0; $i < 10; $i++)
    The current value is {{ $i }}
@endfor

@foreach ($users as $user)
    <p>This is user {{ $user->id }}</p>
@endforeach

@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>No users</p>
@endforelse

@while (true)
    <p>I'm looping forever.</p>
@endwhile
```

While iterating through a `foreach` loop, use the `loop` variable for info about the loop, like the `first` or `last` iteration.

Use the `@continue` and `@break` directives to end the loop or skip the current iteration:

```php
@foreach ($products as $product)
    @if ($product->type == 1)
        @continue
    @endif

    <li>{{ $product->name }}</li>

    @if ($product->number == 5)
        @break
    @endif
@endforeach
```

Include the continuation or break condition within the directive declaration:

```php
@foreach ($products as $product)
    @continue($product->type == 1)

    <li>{{ $product->name }}</li>

    @break($product->number == 5)
@endforeach
```

### &#10022; The Loop Variable:

While iterating through a `foreach` loop, a `$loop` variable is available inside the loop. This variable gives access to useful info like the current loop `index` and whether it's the `first` or `last` iteration:

```php
@foreach ($products as $product)
    @if ($loop->first)
        This is the first iteration.
    @endif

    @if ($loop->last)
        This is the last iteration.
    @endif

    <p>This is product {{ $product->id }}</p>
@endforeach
```

In a nested loop, access the parent loop's `$loop` variable via the parent property:

```php
@foreach ($products as $product)
    @foreach ($product->offers as $offer)
        @if ($loop->parent->first)
            This is the first iteration of the parent loop.
        @endif
    @endforeach
@endforeach
```

The `$loop` variable also has other useful properties:

Property | Description
---|---
$loop->index | The index of the current loop iteration (starts at 0).
$loop->iteration | The current loop iteration (starts at 1).
$loop->remaining | The iterations remaining in the loop.
$loop->count | The total number of items in the array being iterated.
$loop->first | Whether this is the first iteration through the loop.
$loop->last | Whether this is the last iteration through the loop.
$loop->even | Whether this is an even iteration through the loop.
$loop->odd | Whether this is an odd iteration through the loop.
$loop->depth | The nesting level of the current loop.
$loop->parent | When in a nested loop, the parent's loop variable.

### &#10022; Conditional Classes:

The `@class` directive conditionally compiles a CSS class string. It takes an array of classes where the array key is the class or classes to add, and the value is a boolean expression. If the array element has a numeric key, it's always included in the rendered class list:

```php
@php
    $isActive = false;
    $hasError = true;
@endphp

<span @class([
    'p-4',
    'text-primary' => $isActive,
    'text-light' => ! $isActive,
    'bg-red' => $hasError,
])></span>
```

*Result:*

```
<span class="p-4 text-light bg-red"></span>
```

### &#10022; Including Subviews:

While using the `@include` directive, Blade components offer similar functionality with benefits like data and attribute binding.

Blade's `@include` directive includes a Blade view within another view. All parent view variables are available to the included view:

```php
<div>
    @include('shared.errors')

    <form>
        </form>
</div>
```

Pass an array of additional data to the included view:

```php
@include('bootstrap.alert', ['status' => 'complete'])
```

If the `@include` view doesn't exist, Laravel throws an error. Use `@includeIf` for views that may not exist:

```php
@includeIf('bootstrap.alert', ['status' => 'complete'])
```

Use `@includeWhen` and `@includeUnless` for conditional includes:

```php
@includeWhen($boolean, 'bootstrap.alert', ['status' => 'complete'])

@includeUnless($boolean, 'bootstrap.alert', ['status' => 'complete'])
```

Use `@includeFirst` to include the first existing view from an array:

```php
@includeFirst(['custom.admin', 'admin'], ['status' => 'complete'])
```

**Note:**

Avoid using `__DIR__` and `__FILE__` constants in Blade views, as they refer to the cached, compiled view.

### &#10022; Rendering Views For Collections:

Combine loops and includes with Blade's `@each` directive:

```php
@each('bootstrap.alert', $jobs, 'job')
```

The `@each` directive's first argument is the view to render for each array or collection element. The second is the array or collection, and the third is the variable name for the current iteration within the view. The array key for the current iteration is available as the key variable.

Pass a fourth argument for the view to render if the array is empty:

```php
@each('bootstrap.alert', $jobs, 'job', 'alert.warning')
```

Views rendered via `@each` don't inherit parent view variables. Use `@foreach` and `@include` instead if the child view needs them.

### &#10022; The @once Directive:

The `@once` directive defines a template portion that's evaluated only once per rendering cycle. This is useful for pushing JavaScript into the page's header using stacks. If rendering a component in a loop, push the JavaScript to the header only the first time the component renders:

```php
@once
    @push('scripts')
        <script>
            // Custom JavaScript...
        </script>
    @endpush
@endonce
```

### &#10022; Raw PHP:

To embed PHP code in the views, use the Blade `@php` directive to execute a block of plain PHP within the template:

```php
@php
    $counter = 1;
@endphp
```

### &#10022; Comments:

Blade define comments in the views. Unlike HTML comments, Blade comments aren't included in the HTML returned by the application:

```php
{{-- This comment exist and not displayed in the rendered HTML --}}
```

### &#10022; Components:

Components and slots offer benefits similar to sections, layouts, and includes, with a potentially simpler mental model. There are two ways to create components: 

1. Class-based components
2. Anonymous components

### &#10022; Class Based Components:

To create a class-based component, use the `make:component` Artisan command. This command places the component in the `app/View/Components` directory.

```bash
php artisan make:component Alert
```

This command also creates a view template for the component, placed in the `resources/views/components` directory. Components within the application are automatically discovered in the `app/View/Components` and `resources/views/components` directories, so no registration is needed.

It is possible to create components in subdirectories as below:

```bash
php artisan make:component Form/Button
```

This command creates an Button component in the `app/View/Components/Form` directory, and the view is placed in the `resources/views/components/form` directory.

### &#10022; Manually Registering Package Components:

When creating components for the application, It is automatically discovered from the `app/View/Components` and `resources/views/components` directories.

But, when building a package with Blade components, Then it must manually register the component class and its HTML tag alias. Register components in the `boot` method of package service provider class.

```php
use Illuminate\Support\Facades\Blade;

/**
 * Bootstrap your package's services.
 */
public function boot()
{
    Blade::component('package-alert', Alert::class);
}
```

Once registered, render the component using its tag alias:

```php
<x-package-alert/>
```

Alternatively, use the `componentNamespace` method to autoload component classes by convention. For instance, a `Nightshade` package with `Calendar` and `ColorPicker` components in the `Package\Views\Components` namespace as below:

```php
use Illuminate\Support\Facades\Blade;

/**
 * Bootstrap your package's services.
 *
 * @return void
 */
public function boot()
{
    Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
}
```

This allows package component usage by their vendor namespace using the `package-name::component` syntax:

```php
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Blade automatically detects the class linked to the component by pascal-casing the component name. Subdirectories are supported using `.` dot notation.

### &#10022; Rendering Components:

To display a component, use a Blade component tag in a Blade template. Blade component tags start with `x-` followed by the `kebab-case` name of the component class:

```php
<x-alert/>

<x-user-profile/>
```

If the component class is nested deeper in the `app/View/Components` directory, use the `.` dot character to indicate directory nesting. For example, if a component is located at `app/View/Components/Form/Button.php`, render it like this:

```php
<x-form.button/>
```

### &#10022; Passing Data To Components:

Pass data to Blade components using HTML attributes. Hard-coded, primitive values are passed using simple HTML attribute strings. PHP expressions and variables are passed via attributes using the `:attribute` prefix:

```php
<x-alert type="error" :message="$message"/>
```

Define the component's required data in its class constructor. All public properties on a component are automatically available to the component's view. No need to pass data to the view from the component's render method:

```php
namespace App\View\Components;

use Illuminate\View\Component;

class Alert extends Component
{
    /**
     * The alert type.
     *
     * @var string
     */
    public $type;

    /**
     * The alert message.
     *
     * @var string
     */
    public $message;

    /**
     * Create the component instance.
     *
     * @param  string  $type
     * @param  string  $message
     * @return void
     */
    public function __construct($type, $message)
    {
        $this->type = $type;
        $this->message = $message;
    }

    /**
     * Get the view / contents that represent the component.
     *
     * @return \Illuminate\View\View|\Closure|string
     */
    public function render()
    {
        return view('components.alert');
    }
}
```

When the component is rendered, display the contents of public variables by echoing the variables by name:

```php
<div class="alert alert-{{ $type }}">
    {{ $message }}
</div>
```

### &#10022; Casing:

Component constructor arguments should use `camelCase`, while `kebab-case` should be used when referencing the argument names in HTML attributes. For example, given this component constructor:

```php
/**
 * Create the component instance.
 *
 * @param  string  $alertType
 * @return void
 */
public function __construct($alertType)
{
    $this->alertType = $alertType;
}
```

The `$alertType` argument can be provided to the component like this:

```php
<x-alert alert-type="danger" />
```

### &#10022; Escaping Attribute Rendering:

Since some JavaScript frameworks like `Alpine.js` also use colon-prefixed attributes, use a double colon `::` prefix to tell Blade the attribute isn't a PHP expression. For example, given this component:

```php
<x-button ::class="{ danger: isDeleting }">
    Submit
</x-button>
```

Blade will render the following HTML:

```php
<button :class="{ danger: isDeleting }">
    Submit
</button>
```

### &#10022; Component Methods:

Besides public variables being available to the component template, any public methods on the component can be invoked. For instance, consider a component with an `isSelected` method:

```php
/**
 * Determine if the given option is the currently selected option.
 *
 * @param  string  $option
 * @return bool
 */
public function isSelected($option)
{
    return $option === $this->selected;
}
```

It is possible to execute this method from the component template by invoking the variable matching the method's name:

```php
<option {{ $isSelected($value) ? 'selected="selected"' : '' }} value="{{ $value }}">
    {{ $label }}
</option>
```

### &#10022; Accessing Attributes and Slots Within Component Classes:

Blade components allows to access the `component name`, `attributes`, and `slot` inside the component class `render` method. To access this data, return a closure from the component `render` method. The closure receives a `$data` array as its only argument, containing information about the component.

```php
/**
 * Get the view / contents that represent the component.
 *
 * @return \Illuminate\View\View|\Closure|string
 */
public function render()
{
    return function (array $data) {
        // $data['componentName'];
        // $data['attributes'];
        // $data['slot'];

        return '<div>Components content</div>';
    };
}
```

The `componentName` is the name used in the HTML tag after the `x-` prefix. So `<x-alert />` is a component and name of `componentName` is `alert`. The attributes element contains all attributes present on the HTML tag. The slot element is an `Illuminate\Support\HtmlString` instance with the component's slot contents.

The closure should return a string. If the string matches an existing view, that view is rendered. Otherwise, the string is evaluated as an inline Blade view.

### &#10022; Additional Dependencies:

If the component needs dependencies from Laravel service container, list them before the component data attributes. It will be automatically injected by the container.

```php
use App\Services\AlertCreator;

/**
 * Create the component instance.
 *
 * @param  \App\Services\AlertCreator  $creator
 * @param  string  $type
 * @param  string  $message
 * @return void
 */
public function __construct(AlertCreator $creator, $type, $message)
{
    $this->creator = $creator;
    $this->type = $type;
    $this->message = $message;
}
```

### &#10022; Hiding Attributes or Methods:

To prevent some public methods or properties from being exposed as variables to the component template, add them to an `$except` array property on the component.

```php
namespace App\View\Components;

use Illuminate\View\Component;

class Alert extends Component
{
    /**
     * The alert type.
     *
     * @var string
     */
    public $type;

    /**
     * The properties / methods that should not be exposed to the component template.
     *
     * @var array
     */
    protected $except = ['type'];
}
```

### &#10022; Component Attributes:

Sometimes, It is required to specify additional HTML attributes like class, id, data-attributes and so on. That aren't part of the component required data. If it required to pass these attributes to the component template root element. For example, rendering an alert component as below.

```php
<x-alert type="error" :message="$message" class="mt-4"/>
```

Attributes are not in the component constructor are added to the component `attribute bag`. This bag is available to the component via the `$attributes` variable. Render all attributes within the component by echoing this variable.

```php
<div {{ $attributes }}>
    <!-- Component content -->
</div>
```

Using directives like `@env` within component tags isn't supported. 

For example, below definition won't compile and work.

```php
<x-alert :live="@env('production')"/>
```

### &#10022; Default or Merged Attributes:

Sometimes, If required to specify default attribute values or merge additional values into component attributes. Use the attribute bag's `merge` method. This method is helpful for defining default CSS classes that should always apply to a component.

```php
<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

Above definition will be used like below:

```php
<x-alert type="error" :message="$message" class="mb-4"/>
```

The final rendered HTML of the component will be:

```php
<div class="alert alert-error mb-4">
    <!-- Message information -->
</div>
```

### &#10022; Conditionally Merge Classes:

Sometimes, If required to merge classes based on a condition. Use the `class` method, which takes an array of classes. The array key is the `class` or `classes` to add, and the value is a boolean expression. If the array element has a numeric key, it is always included in the rendered class list.

```php
<div {{ $attributes->class(['p-4', 'bg-red' => $hasError]) }}>
    {{ $message }}
</div>
```

To merge other attributes onto the component, chain the merge method onto the class method.

```php
<button {{ $attributes->class(['p-4'])->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
```

For conditionally compiling classes on other HTML elements that shouldn't receive merged attributes, use the `@class` directive.

### &#10022; Non-Class Attribute Merging:

When merging attributes that aren't class attributes, the values provided to the merge method are considered the attribute's `default` values. Unlike class attributes, these attributes won't merge with injected attribute values. It will be overwritten. For example, a button component's implementation as below:

```php
<button {{ $attributes->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
```

To render the button component with a custom type, specify it when consuming the component. If nothing is specified to `type` attribute, then `type` will use default value of `button`.

```php
<x-button type="submit">
    Submit
</x-button>
```

The rendered HTML of the button component:

```php
<button type="submit">
    Submit
</button>
```

If it required an attribute other than class to have its default value and injected values joined, use the `prepends` method. In this example, the `data-controller` attribute will always start with `profile-controller`, and any additional injected `data-controller` values will be placed after this default value.

```php
<div {{ $attributes->merge(['data-controller' => $attributes->prepends('profile-controller')]) }}>
    {{ $slot }}
</div>
```

### &#10022; Retrieving and Filtering Attributes:

To filter attributes using the `filter` method, which takes a closure that returns `true` to keep the attribute in the attribute bag.

```php
{{ $attributes->filter(fn ($value, $key) => $key == 'type') }}
```

Use the `whereStartsWith` method to get attributes whose keys start with a given string.

```php
{{ $attributes->whereStartsWith('wire:model') }}
```

Use `whereDoesntStartWith` to exclude attributes whose keys start with a given string.

```php
{{ $attributes->whereDoesntStartWith('wire:model') }}
```

Use the `first` method to render the `first` attribute in an attribute bag.

```php
{{ $attributes->whereStartsWith('wire:model')->first() }}
```

Use the `has` method to check if an attribute is present, returning a boolean:

```php
@if ($attributes->has('class'))
    <div>message content</div>
@endif
```

Use the `get` method to retrieve a specific attribute value.

```php
{{ $attributes->get('class') }}
```

### &#10022; Reserved Keywords:

By default, some keywords are reserved for Blade internal use when rendering components. These keywords cannot be defined as public `properties` or `method` names within the components as below:

- data
- render
- resolveView
- shouldRender
- view
- withAttributes
- withName

### &#10022; Slots:

The additional content to be passed to the component through `slots`. Component `slots` are rendered by echoing the `$slot` variable. Imagine an alert component with as below:

```php
<div class="alert alert-danger">
    {{ $slot }}
</div>
```

Pass content to the slot to injecting content in the component.

```php
<x-alert>
    <strong>Danger!</strong> Something went wrong!
</x-alert>
```

Sometimes a component needs to render multiple slots in different locations.

```php
<span class="alert-title">{{ $title }}</span> <!-- title slot -->

<div class="alert alert-danger">
    {{ $slot }} <!-- Default content slot -->
</div>
```

Define the content of a named slot using the `x-slot` tag as below. Content outside `x-slot` tags is passed to the `$slot` variable.

```php
<x-alert>
    <x-slot name="title">
        404
    </x-slot>

    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

### &#10022; Scoped Slots:

JavaScript frameworks like `Vue` that might uses `scoped slots`, which allow to access `data` or `methods` from the component within slot. It is achieve similar behavior in Laravel by defining public `methods` or `properties` on the component and accessing the component within the slot via the `$component` variable. In this example, assume the `x-alert` component has a public `formatAlert` method defined on its component class.

```php
<x-alert>
    <x-slot name="title">
        {{ $component->formatAlert('Server Error') }}
    </x-slot>

    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

### &#10022; Slot Attributes:

Like Blade components, to assign additional attributes to slots, such as CSS class names.

```php
<x-card class="shadow-sm">
    <x-slot name="heading" class="font-bold">
        Heading
    </x-slot>

    Content

    <x-slot name="footer" class="text-sm">
        Footer
    </x-slot>
</x-card>
```

To interact with `slot` attributes, access the `attributes` property of the slot variable. For more information on attribute interaction, see the component attributes documentation.

```php
@props([
    'heading',
    'footer',
])

<div {{ $attributes->class(['border']) }}>
    <h1 {{ $heading->attributes->class(['text-lg']) }}>
        {{ $heading }}
    </h1>

    {{ $slot }}

    <footer {{ $footer->attributes->class(['text-gray-700']) }}>
        {{ $footer }}
    </footer>
</div>
```

### &#10022; Inline Component Views:

For very small components to manage both the component class and view template by return the component markup directly from the `render` method.

```php
/**
 * Get the view / contents that represent the component.
 *
 * @return \Illuminate\View\View|\Closure|string
 */
public function render()
{
    return <<<'blade'
        <div class="alert alert-danger">
            {{ $slot }}
        </div>
    blade;
}
```

### &#10022; Generating Inline View Components:

To create a component that renders an inline view, use the `--inline` option when executing the `make:component` command.

```bash
php artisan make:component Alert --inline
```

### &#10022; Anonymous Components:

Like inline components, The anonymous components will manage a component with a single file. But, anonymous components uses a single view file and have no associated class. To define an anonymous component, place a Blade template inside `resources/views/components` directory. For example, if there is a component at `resources/views/components/alert.blade.php`, render it like anonymous component as below:

```php
<x-alert/>
```

Use the `.` dot character to indicate, if a component is nested multi-level or deeper in the components directory. For example, if the component is defined at `resources/views/components/form/button.blade.php`, then it can be rendered as below:

```php
<x-inputs.button/>
```

### &#10022; Anonymous Index Components:

When a component consists of many Blade templates, Then it need to be grouped them within a single directory. For example, an `accordion` component with this directory structure as below:

```
/resources/views/components/accordion.blade.php
/resources/views/components/accordion/item.blade.php
```

This structure allows to render the accordion component and its items as below:

```php
<x-accordion>
    <x-accordion.item>
        ...
    </x-accordion.item>
</x-accordion>
```

But, It can render the accordion component via `x-accordion`, the `index` accordion component template must be placed in the `resources/views/components` directory, not nested within the accordion directory with the other accordion-related templates.

Blade allows to place an `index.blade.php` file within a component template directory. When an `index.blade.php` template exists, it rendered as the `root` node of the component. So, use the same Blade syntax as above, but adjust the directory structure as below:

```
/resources/views/components/accordion/index.blade.php
/resources/views/components/accordion/item.blade.php
```

### &#10022; Data Properties or Attributes:

Since anonymous components don't have an associated class, So, it important to differentiate which data should be passed as variables and which attributes should be placed in the component's attribute bag.

Use the `@props` directive at the top of the component's Blade template to specify which attributes should be considered data variables. All other attributes are available via the component's `attribute` bag. To give a data variable a default value, specify the variable's `name` as the array `key` and the default `value` as the array value as below:

```php
@props(['type' => 'info', 'message'])

<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

Given the component definition above, render the component as below:

```php
<x-alert type="error" :message="$message" class="mb-4"/>
```

### &#10022; Accessing Parent Data:

Sometimes, It need to access data from a parent component within a child component. In these cases, use the `@aware` directive. 

For example, consider building a navigation menu component with a parent `<x-menu>` and child `<x-menu.item>` as below:

```php
<x-menu color="primary">
    <x-menu.item>...</x-menu.item>
    <x-menu.item>...</x-menu.item>
</x-menu>
```

The `<x-menu>` component might have this implementation:

```php
@props(['color' => 'light'])

<nav {{ $attributes->merge(['class' => 'bg-'.$color.'-200']) }}>
    {{ $slot }}
</nav>
```

Because the color `prop` was passed only to the parent `<x-menu>`, it won't be available inside `<x-menu.item>`. But it is possible by using the `@aware` directive, to makes it available inside `<x-menu.item>` as well:

```php
@aware(['color' => 'gray'])

<a {{ $attributes->merge(['class' => 'text-'.$color.'-800']) }}>
    {{ $slot }}
</a>
```

### &#10022; Dynamic Components:

Sometimes, It requires to render a component, but it not know which one until runtime. Use Laravel built-in `dynamic-component` component to render based on a runtime value or variable.

```php
<x-dynamic-component :component="$componentName" class="mt-4" />
```

### &#10022; Manually Registering Components:

If building a package that utilizes Blade components or placing components in non-conventional directories, Then it needs to manually register those component class and its HTML tag alias. So that, Laravel knows where to find these component. Register the components in the `boot` method of the package's service provider.

```php
use Illuminate\Support\Facades\Blade;
use VendorPackage\View\Components\AlertComponent;

/**
 * Bootstrap your package's services.
 *
 * @return void
 */
public function boot()
{
    Blade::component('package-alert', AlertComponent::class);
}
```

Once the component has been registered, it may be rendered using its tag alias.

```php
<x-package-alert/>
```

### &#10022; Autoloading Package Components:

Alternatively, use the `componentNamespace` method to autoload component classes by convention. For instance, a `Nightshade` package might have `Calendar` and `ColorPicker` components in the `Package\Views\Components` namespace.

```php
use Illuminate\Support\Facades\Blade;

/**
 * Bootstrap your package's services.
 *
 * @return void
 */
public function boot()
{
    Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
}
```

This enables package component usage by their vendor namespace using the `package-name::` syntax:

```php
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Blade automatically detects the class linked to the component by `pascal-casing` the component name. Subdirectories are also supported using `.` dot notation.

### &#10022; Building Layouts:

Most of the web applications use the same general layout across different pages.

Blade Template gives possible to create reusable layouts. That can minimizes the various declaration used in the view.

### &#10022; Defining Layout Component:

For example, consider a a simple web application. That might use similar layouts across multiple pages.

```php
<html>
    <head>
        <title>{{ $title ?? 'App Title' }}</title>
    </head>
    <body>
        <h1>Welcome</h1>
        <hr/>
        {{ $slot }}
    </body>
</html>
```

### &#10022; Applying The Layout Component:

Once the layout component is defined, then create a Blade view that uses the layout component. In this example, we'll define a simple view that displays products list:

```php
<x-layout>
    @foreach ($products as $product)
        {{ $product }}
    @endforeach
</x-layout>
```

Content injected into a component is supplied to the default `$slot` variable in the layout component. The layout also respects a `$title` slot if provided. Otherwise, a default title is shown. Inject a custom title from products list into view using standard slot syntax.

```php
<x-layout>
    <x-slot name="title">
        Custom Title
    </x-slot>

    @foreach ($products as $product)
        {{ $product }}
    @endforeach
</x-layout>
```

Render to user by define in the route:

```php
use App\Models\Product;

Route::get('/products', function () {
    return view('products', ['products' => Product::all()]);
});
```

### &#10022; Layouts Using Template Inheritance:

**Defining A Layout:**

Layouts can also be created using template inheritance. This was the primary method before components were introduced.

*Example:* 

```php
<html>
    <head>
        <title>@yield('title')</title>
    </head>
    <body>
        <aside>
            @section('sidebar')
                Some sidebar content
            @show
        </aside>

        <main class="container">
            @yield('content')
        </main>
    </body>
</html>
```

As above, layout has the `@section` and `@yield` directives. The `@section` directive defines a content section, while `@yield` displays the contents of a given section.

To utilize the layout then define child layout.

### &#10022; Extending A Layout:

To extend layout in child layouts, use the `@extends` Blade directive to specify which layout the child view should to `inherit`. Views that extend a Blade layout can inject content into the layout sections using `@section` directives. 

```php
@extends('layouts.app')

@section('title', 'Page Title')

@section('sidebar')
    @@parent <!-- Parent layout content -->

    <p>Another content to be appended in the sidebar.</p>
@endsection

@section('content')
    Main content
@endsection
```

In this example, the sidebar section uses the `@@parent` directive to append rather than overwrite content to the layout sidebar. The `@@parent` directive will be replaced by the layout content when the view is rendered.

This sidebar section ends with `@endsection` instead of `@show` in previous layout definition. The `@endsection` directive defines a section, while `@show` defines and immediately yields the section.

The `@yield` directive also accepts a default value as its second parameter. This value will be rendered if the section being yielded is undefined.

```php
@yield('title', 'Default Title')
```

### &#10022; Forms:

Blade template provides possibilities to creating forms and form elements.

### &#10022; CSRF Field:

Whenever an HTML form in the application, that should include a hidden `CSRF` `_token` field to allow the CSRF protection middleware to validate the request. Use the `@csrf` Blade directive to generate the token field.

```php
<form method="POST" action="/product/create">
    @csrf
    <!-- reset of the form definitions -->
</form>
```

### &#10022; Method Field:

Since HTML forms does not support PUT, PATCH, or DELETE requests methods. So, it need to add a hidden `_method` field to simulate these HTTP verbs. Use the `@method` Blade directive can create this field.

```php
<form action="/product/2" method="POST">
    @method('PUT')
    <!-- Rest of the form definitions -->
</form>
```

### &#10022; Validation Errors:

The `@error` directive quickly checks if validation error messages exist for a given attribute. Within an `@error` directive, echo the `$message` variable to display the form validation error messages.

```php
<label for="product-name">Product Name</label>

<input id="product-name" type="text" class="@error('product_name') is-invalid @enderror">

@error('product_name')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

Since the `@error` directive compiles to an `if` statement, use the `@else` directive to render content when there is no error for given attribute.

```php
<label for="product-name">Product Name</label>

<input id="product-name" type="text" class="@error('email') is-invalid @else is-valid @enderror">
```

If the view has multiple forms then to retrieve errors in specific form, then pass the name of a specific error bag as the second parameter to the `@error` directive to retrieve validation error messages on pages.

```php
<form name="main" action="/product/create" method="post">

<label for="product-name">Product Name</label>

<input id="product-name" type="text" class="@error('product_name') is-invalid @else is-valid @enderror">

@error('product_name', 'main')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

### &#10022; Stacks:

Blade allows to push named stacks, which can be rendered elsewhere in another view or layout. This is useful for specifying JavaScript libraries required by child views.

```php
@push('scripts')
    <script src="/example.js"></script>
@endpush
```

Push to a stack as many times as needed. To render the complete stack contents, pass the stack name to the `@stack` directive:

```php
<html>
<body>
    <!-- rest of the view definitions -->
    @stack('scripts')
</body>
</html>
```

To prepend content to the beginning of a stack, use the `@prepend` directive.

```php
@push('scripts')
    <!-- primary scripts-->
@endpush

// Later...

@prepend('scripts')
    <!-- This will be prepend to top of all scripts -->
@endprepend
```

### &#10022; Service Injection:

The `@inject` directive retrieves a service from the Laravel service container. The first argument is the variable name for the service, and the second is the class or interface name of the service.

```php
@inject('metrics', 'App\Services\MetricsService')

<div>
    Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
</div>
```

### &#10022; Extending Blade:

Blade allows to define custom directives using the `directive` method. When the Blade compiler encounters a custom directive, it calls the provided callback with the directive expression.

For an example, It creates a `@datetime($var)` directive that formats a given `$var` (an instance of DateTime).

```php
namespace App\Providers;

use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        //
    }

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::directive('datetime', function ($expression) {
            return " echo ($expression)->format('m/d/Y H:i'); ?>";
        });
    }
}
```

As shown, the `format` method is chained onto the expression passed into the directive. The generated PHP as below:

```php
 echo ($var)->format('m/d/Y H:i'); ?>
```

**Note:**

- After updating a Blade directive logic, delete all cached Blade views using the `view:clear` Artisan command.

### &#10022; Custom Echo Handlers:

When try to `echo` an object using Blade, the object `__toString` method is invoked. `__toString` is one of PHP's built-in `magic` methods. But it is not always control a class `__toString` method, especially with third-party libraries.

In these cases, Blade allow to register a custom `echo` handler for the object types. Use Blade `stringable` method to implement custom echo handler. The `stringable` method accepts a closure that type-hints the object type it renders. Typically, invoke `stringable` in the application `AppServiceProvider` class `boot` method.

```php
use Illuminate\Support\Facades\Blade;
use Money\Money;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Blade::stringable(function (Money $money) {
        return $money->formatTo('en_IN');
    });
}
```

Once custom echo handler is defined, simply `echo` the object in the Blade template.

```php
Cost: {{ $money }}
```

### &#10022; Custom If Statements:

Blade provides a `if` method to quickly define custom conditional directives using closures. 

For example, To check the configured default "disk" for the application. It is possible by define in the `boot` method of `AppServiceProvider` class.

```php
use Illuminate\Support\Facades\Blade;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Blade::if('disk', function ($value) {
        return config('filesystems.default') === $value;
    });
}
```

Once the custom conditional is defined, The it can be used in the template.

```php
@disk('local')
    <!-- operation with local disk -->
    @elsedisk('s3')
    <!-- operation with s3 disk -->
    @else
    <!-- operation for no other disk match -->
@enddisk

@unlessdisk('local')
    <!-- Executes this block of expression when disk does not matches local -->
@enddisk
```

---
[&#8682; To Top](#-blade-template)

[&#10094; Previous Topic](./views.md) &emsp; [Next Topic &#10095;](./csrf-protection.md)

[&#8962; Goto Home Page](../README.md)