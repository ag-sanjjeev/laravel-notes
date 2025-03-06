## &#10162; Views:

Laravel views features provide convenient ways to place all of the HTML in separate files. Views can separate application controller / application logic from the presentation logic. Views are stored in the `resources/views` directory.

### &#9780; Overview:

1. [Creating and Rendering Views](#-creating-and-rendering-views)
    - [Nested View Directories](#-nested-view-directories)
    - [Creating The First Available View](#-creating-the-first-available-view)
    - [Determining If A View Exists](#-determining-if-a-view-exists)
2. [Passing Data To Views](#-passing-data-to-views)
    - [Sharing Data With All Views](#-sharing-data-with-all-views)
3. [View Composers](#-view-composers)
    - [Attach Composer To Multiple Views](#-attach-composer-to-multiple-views)
    - [View Creators](#-view-creators)
4. [Optimizing Views](#-optimizing-views)

### &#10022; Creating and Rendering Views:

simple view can be created manually under `resources/views` directory at any level.

**View with blade template:**

The view file extension should ends with `.blade.php` in order to process blade templates.

```php
<!-- View stored in resources/views/greeting.blade.php --> 
<html>
    <body>
        <h1>Welcome, {{ $name }}</h1>
    </body>
</html>
```

**View without blade template:**

```php
<!-- View stored in resources/views/greeting.php --> 
<html>
    <body>
        <h1>Welcome, <?= $name ?></h1>
    </body>
</html>
```

The view file extension ends with `.php` in order to process PHP script as view.

**Access view:**

Since this view is stored at `resources/views/greeting.blade.php`, we may return it using the global view helper function.

In `web.php` file:

```php
Route::get('/', function () {
    return view('greeting', ['name' => 'Kumar']);
});
```

Once a view is created, return it from routes or controllers using the global `view` helper:

```php
Route::get('/', function () {
    return view('greeting', ['name' => 'Kumar']);
});
```

Views can also be returned using the View facade with `make` method:

```php
use Illuminate\Support\Facades\View;

return View::make('greeting', ['name' => 'Kumar']);
```

The first argument passed to the view helper is the view file name in the `resources/views` directory. The second argument is an array of data for the view. 

Refer Blade template documentations for more.

### &#10022; Nested View Directories:

Views can be nested in subdirectories of the `resources/views` directory. That may be accessed with `.` dot notation to reference nested views. For example, if the view is at `resources/views/product/create.blade.php`, return it from `routes/controllers` as below:

```php
return view('product.create', $data);
```

**Note:**

View directory names should not have the `.` dot character.

### &#10022; Creating The First Available View:

Using the View facade's first method, you can create the first view that exists in an array of views. This is useful if your application or package lets views be customized or overwritten.

```php
use Illuminate\Support\Facades\View;

return View::first(['custom.admin', 'admin'], $data);
```

### &#10022; Determining If A View Exists:

To check if a view exists, use the View facade. The `exists` method returns true if the view exists.

```php
use Illuminate\Support\Facades\View;

if (View::exists('emails.checkout')) {
    //
}
```

### &#10022; Passing Data To Views:

As seen in previous examples, pass an array of data to views to make that data available:

```php
return view('greetings', ['name' => 'Kumar']);
```

When passing information this way, the data should be an array with key/value pairs. After giving data to a view, access each value within the view using the data's keys, like <?php echo $name; ?>.

Instead of passing a full array of data to the view helper function, use the with method to add individual data pieces to the view. The with method returns a view object instance, letting you chain methods before returning the view.

```php
return view('greeting')
                ->with('name', 'Kumar')
                ->with('state', 'TamilNadu');
```

### &#10022; Sharing Data With All Views:

Sometimes, need to share data with all views rendered by the application. Use the View facade's share method. Typically, place calls to the share method in a service provider's boot method. Add them to the `App\Providers\AppServiceProvider` class or make a separate service provider.

```php
namespace App\Providers;

use Illuminate\Support\Facades\View;
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
        View::share('key', 'value'); // this key will be available in all view files
    }
}
```

### &#10022; View Composers:

It is useful to track some informations such as page view count and so on. This get into action when view is returned to user.

View composers are callbacks or class methods called when a view is rendered. If data should be bound to a view each time it's rendered, a view composer organizes that logic. View composers are useful if the same view is returned by multiple routes or controllers and needs specific data.

View composers are registered in service providers. In this example, we'll use a new `App\Providers\ViewServiceProvider` class.

Use the View facade's composer method to register the view composer. Laravel doesn't have a default directory for class-based view composers, so organize them as needed. For example, create an `app/View/Composers` directory.

```php
namespace App\Providers;

use App\View\Composers\ProfileComposer;
use Illuminate\Support\Facades\View;
use Illuminate\Support\ServiceProvider;

class ViewServiceProvider extends ServiceProvider
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
        // Using class based composers...
        View::composer('profile', ProfileComposer::class);

        // Using closure based composers...
        View::composer('dashboard', function ($view) {
            //
        });
    }
}
```

If you create a new service provider for view composer registrations, add it to the providers array in `config/app.php` file.

Now, the compose method of `App\View\Composers\ProfileComposer` runs each time the profile view is rendered. Here's an example of the composer class:

```php

namespace App\View\Composers;

use App\Repositories\UserRepository;
use Illuminate\View\View;

class ProfileComposer
{
    /**
     * The user repository implementation.
     *
     * @var \App\Repositories\UserRepository
     */
    protected $users;

    /**
     * Create a new profile composer.
     *
     * @param  \App\Repositories\UserRepository  $users
     * @return void
     */
    public function __construct(UserRepository $users)
    {
        // Dependencies are automatically resolved by the service container...
        $this->users = $users;
    }

    /**
     * Bind data to the view.
     *
     * @param  \Illuminate\View\View  $view
     * @return void
     */
    public function compose(View $view)
    {
        $view->with('count', $this->users->count());
    }
}
```

All view composers are resolved via the service container, so type-hint any needed dependencies in the composer's constructor.

### &#10022; Attach Composer To Multiple Views:

Attach a view composer to multiple views by passing an array of views as the first argument to the composer method:

```php
use App\Views\Composers\MultiComposer;

View::composer(
    ['profile', 'dashboard'],
    MultiComposer::class
);
```

The composer method also accepts the `*` symbol as a wildcard, attaching a composer to all views:

```php
View::composer('*', function ($view) {
    //
});
```

### &#10022; View Creators:

View creators are like view composers, but they run right after the view is made, not when it's about to render. To register a view creator, use the creator method:

```php
use App\View\Creators\ProfileCreator;
use Illuminate\Support\Facades\View;

View::creator('profile', ProfileCreator::class);
```

### &#10022; Optimizing Views:

By default, Blade template views are compiled on demand. When a request renders a view, Laravel checks if a compiled version exists. If the file exists, Laravel checks if the uncompiled view was modified more recently than the compiled view. If the compiled view doesn't exist, or the uncompiled view was modified, Laravel recompiles the view.

Compiling views during a request can slightly slow performance, so Laravel provides the `view:cache` Artisan command to precompile all application views. For better performance, run this command during your deployment process.

```bash
php artisan view:cache
```

**Clear the view cache:**

```bash
php artisan view:clear
```

---
[&#8682; To Top](#-views)

[&#10094; Previous Topic](./responses.md) &emsp; [Next Topic &#10095;](./blade-template.md)

[&#8962; Goto Home Page](../README.md)