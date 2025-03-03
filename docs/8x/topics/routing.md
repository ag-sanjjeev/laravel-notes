## &#10162; Routing:

In Laravel, routing is important to defines the mapping between URLs (Uniform Resource Locators) and the code that should be executed when those URLs are accessed. Where the code means controller method or closure should handle a specific request.

Laravel Routing will simplify the application logic almost within routes. It is possible by route parameters, regex constraints, route groups and model binding. 

By these features, it will eliminate usage of controller for retrieve data by the model.

### &#9780; Overview:
1. [Route Definition](#-route-definition),
2. [Dependency Injection](#-dependency-injection)
3. [CSRF Protection](#-csrf-protection)
4. [Redirect Routes](#-redirect-routes)
5. [View Routes](#-view-routes)
6. [Route Parameters](#-route-parameters)
7. [Optional Route Parameters](#-optional-route-parameters)
8. [RegEx Constraints](#-regex-constraints)
9. [Global RegEx Constraints](#-global-regex-constraints)
10. [Encoded Forward Slashes](#-encoded-forward-slashes)
11. [Named Routes](#-named-routes)
12. [Generate URL From Named Routes](#-generate-url-from-named-routes)
13. [Default URL Parameters](#-default-url-parameters)
14. [Inspecting Current Route](#-inspecting-current-route)
15. [Route Groups](#-route-groups)
	- [Controllers](#-controllers)
	- [Middleware](#-middleware)
	- [Route Prefixes](#-route-prefixes)
	- [Route Name Prefixes](#-route-name-prefixes)
	- [Subdomain Routing](#-subdomain-routing)
16. [Model Binding](#-model-binding)
17. [Implicit Model Binding](#-implicit-model-binding)
    - [Model Binding For Soft Deleted Records](#-model-binding-for-soft-deleted-records)
    - [Custom Column Key](#-custom-column-key)
    - [Key Scoping](#-key-scoping)
    - [Missing Model Behavior](#-missing-model-behavior)    
18. [Explicit Model Binding](#-explicit-model-binding)
19. [Custom Model Binding](#-custom-model-binding)
20. [Fallback Routes](#-fallback-routes)
21. [Request Rate Limiting](#-request-rate-limiting)
22. [Form Method Spoofing](#-form-method-spoofing)
23. [Current Route Information](#-current-route-information)
24. [CORS](#-cors)
25. [Route Caching](#-route-caching)

### &#10022; Route Definition:

Routes are located at routes directory. Where it has `web.php`, `api.php` and so on. That defines routes for different interfaces such as web and api correspondingly. 

- `routes/web.php` file defines routes for web interface. These routes are assigned with web middleware group, which provides features like session state management and CSRF protection. 
- `routes/api.php` file defines routes for api interface. These routes are stateless and assigned the api middleware group.

These route files automatically loaded by App\Providers\RouteServiceProvider.

Laravel namespace standard for `web.php` Route file:

```php
use Illuminate\Support\Facades\Route;

// Route definitions	
```

**Routes for Different HTTP Methods:**

```php
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
```

**Special Route Definition for multiple HTTP Methods:**

```php
Route::match(['get', 'post'], $uri, $callback); // matches get and post HTTP methods
Route::any($uri, $callback); // accepts any request methods
```

**Note:**

- When defining routes for same URI with different HTTP method, then first definition will be taken with corresponding HTTP method. Avoid defining any, match and redirect methods before any specific route method.

### &#10022; Dependency Injection:

When route definition has callback with dependencies, then Laravel resolve by inject to the callback closure. Here, Request class properties and method are injected as request object to the callback parameter. This will be resolved when a request.

```php
use Illuminate\Support\Facades\Route;
use Illuminate\Http\Request;
 
Route::get('/users', function (Request $request) {
    // ...
});
```

### &#10022; CSRF Protection:

If any of the HTML forms submitted with following methods of POST, PUT, PATCH or DELETE, then before form should include a CSRF token field. Otherwise, the request will be rejected due to Cross Site Request Forgery (CSRF).

```html
<form method="POST" action="/login">
    @csrf
    <!-- form fields -->
</form>
```

### &#10022; Redirect Routes:

Laravel Route class has methods for redirecting request.

**Redirect with default status code of 302:**

```php
Route::redirect('/here', '/there');
```

**Redirect with custom status code:**

```php
Route::redirect('/here', '/there', 301); // permanent redirect
```

**Redirect as permanent without specifying status code:**

```php
Route::permanentRedirect('/here', '/there');
```

**Note:**

Route redirection URLs should be defined in Routes definitions before use it.

### &#10022; View Routes:

Laravel directly provide view response with view method. The first argument takes URL, second argument takes view file and third optional parameter takes parameter to be injected in view for utilize when rendering.

**Note:**

- Third optional parameters should not use the names as `view, data, status and headers`. Because that are reserved by Laravel.

```php
Route::view('/welcome', 'welcome');
```

```php
Route::view('/dashboard', 'dashboard', ['name' => 'Kumar']);
```

### &#10022; Route Parameters:

Laravel will help to capture specific URL components like user id, post number and so on. 

**Note:**

- Route parameters are should be within `{}` curly braces and their name should consist of alphabetic characters with or without underscores `_`. 
- Route parameters are injected into route callbacks / controllers based on their order of definitions and the names of the route callback / controller arguments name does not matter.

```php
Route::get('/user/{id}', function ($id) {
    //
});
```

- If it required to capture multiple URL components, then should specify in the order to the callback / controller / closure arguments. 

```php
Route::get('/posts/{post_id}/comments/{comment_id}', function ($postId, $commentId) {
    // {post_id} will be captured and set to first occurrence of argument as specified as $postId
    // {comment_id} will be captured and set to second occurrence of argument as specified as $commentId
});
```

### &#10022; Optional Route Parameters:

Sometimes, route parameters may be or may not be found, that can be defined with `?` question mark. And set default value for callback / controller / closure parameter to avoid undefined variable error.

```php
Route::get('/post/{post_id?}', function ($postId = null) {
    //
});
```
 
```php
Route::get('/post/{post_id?}', function ($postId = 1) {
    //
});
```

### &#10022; RegEx Constraints:

Laravel can validate Route parameter with regular expression. Which does not match with constraints then fallback to 404 not found page.

```php
Route::get('/dashboard/{user_name}', function ($name) {
    //
})->where('user_name', '[A-Za-z]+');
```

```php
Route::get('/post/{id}', function ($id) {
    //
})->where('id', '[0-9]+');
```

```php
Route::get('/post/{post_id}/comments/{comment}', function ($postId, $commentId) {
    //
})->where(['post_id' => '[0-9]+', 'comment' => '[0-9]+']);
```

**Note:**

- In above, definitions Route parameter names might match regex constrains name and position to avoid conflicts.

**Simplified Constrain Methods:**

```php
Route::get('/user/{id}/{name}', function ($id, $name) {
    //
})->whereNumber('id')->whereAlpha('name');
```

```php
Route::get('/dashboard/{user_name}', function ($name) {
    //
})->whereAlphaNumeric('user_name');
```

```php
Route::get('/user/{id}', function ($id) {
    //
})->whereUuid('id');
```

### &#10022; Global RegEx Constraints:

Laravel give possibilities to define regular expression patterns inside `boot` method of `App\Providers\RouteServiceProvider` class. Once it is defined then it will be applied to all route parameters.

Boot method can define route model bindings, pattern filters, etc.

```php
public function boot()
{
    Route::pattern('id', '[0-9]+');
}
```

**Route Definitions:**

```php
Route::get('/user/{id}', function ($id) {
    // this will be invoked when $id is numeric based on boot method global constraints definitions.
});
```

### &#10022; Encoded Forward Slashes:

Laravel routes parameters will allow all characters except `/` slashes in the parameters' value. And it can allow when explicitly define with `where` method and RegEx.

But it supports only route parameter positioned at end of the URL component.

```php
Route::get('/search/{search}', function ($search) {
    // $search might contains slashed (/)
})->where('search', '.*');
```

### &#10022; Named Routes:

Laravel routes can allow to set name for route definitions. That gives convenient to generate URLs or to redirect with specified route. It is easy to define name for route using `name` method by method chaining. 

**Note:**

- Route name should be always unique to avoid conflict.

```php
Route::get('/dashboard', function () {
    //
})->name('dashboard');
```

### &#10022; Generate URL From Named Routes:

Generate URLs from previously defined routes using Laravel route helper functions.

```php
$url = route('dashboard');
```

Redirect to specific URL from previously defined routes using Laravel redirect helper functions.

```php
return redirect()->route('dashboard');
```

**Note:**

- If route has route parameters, then it requires to pass as a second argument when using route and redirect functions.

```php 
Route::get('/profile/{id}', function ($id) {
    //
})->name('profile');
```

```php
$url = route('profile', ['id' => 1]);
```

```php
return redirect()->route('profile', ['id' => 1]);
```

**Note:**

- If passing an another parameters additionally than definition then it will also injected as query parameters in URL's string.

```php
$url = route('profile', ['id' => 1, 'theme' => 'dark']);
```

```
URL: /profile/1?theme=dark
```

### &#10022; Default URL Parameters:

Sometimes, It is required to specify through out all request with default values for URL parameters such as the locale setting, theme setting an so on. 

To accomplish default URL parameters to all requests, then use the `URL::defaults` method.

### &#10022; Inspecting Current Route:

It is possible to inspect current request, which is defined with route name. Use `named` method on a Route instance. 

*Example: Check current route name inside route middleware*

```php
/**
 * Handles incoming request.
 *
 * @param  \Illuminate\Http\Request  $request
 * @param  \Closure  $next
 * @return mixed
 */
public function handle($request, Closure $next)
{
    if ($request->route()->named('profile')) {
        //
    }
 
    return $next($request);
}
```

### &#10022; Route Groups:

In Laravel, routes can be grouped under same Controllers, Middlewares, Route Prefixes, Route Name Prefixes and Subdomain Routing.

Which share same attributes across all the route member in the group definition.

Nested groups attempt to merge or append or add attributes with their parent group. For Middleware and where method conditions will be merged. Route Names and Route prefixes will be appended. Namespace delimiters and slashes in URI prefixes are automatically added.

### &#10022; Controllers:

To create route group with controller class using method chaining. When multiple routes access same controller then it would be grouped.

```php
use App\Http\Controllers\UserController;
 
Route::controller(UserController::class)->group(function () {
    Route::get('/user', 'index');
    Route::post('/user/setting', 'show');
});
```

### &#10022; Middleware:

To create route group with middleware using method chaining. middleware method accepts name of middlewares to be applied to all route and executed before route callback.  

```php
Route::middleware(['auth', 'role'])->group(function () {
    Route::get('/user', function () {
        // both auth and role middlewares executed before
    });
 
    Route::get('/dashboard', function () {
        // both auth and role middlewares executed before
    });
});
```

### &#10022; Route Prefixes:

When route URLs have same prefix in it, then it can be grouped with prefix method. 

```php
Route::prefix('blog')->group(function () {
    Route::get('/post/{id}', function ($id) {
        // Matches The "/blog/post/{id}" URL
    });

    Route::get('/post/{search}', function ($search) {
        // Matches The "/blog/post/{search}" URL
    });
});
```

### &#10022; Route Name Prefixes:

When it requires to set name that has common prefix for group of routes then those routes can be grouped.

Use trailing `.` dot as separator for route names.

```php
Route::name('post.')->group(function () {
    Route::get('/blog/post/{id}', function ($id) {
        // Route assigned name "post.show"...
    })->name('show');

    Route::delete('/blog/post/{id}', function ($id) {
        // Route assigned name "post.delete"...
    })->name('delete');
});
```

### &#10022; Subdomain Routing:

If there is sub-domain in the Laravel application, then it can be grouped. It might assign route parameters as route URIs. If dynamic sub-domain then use RegEx to capture that sub-domain name.

```php
Route::domain('{user}.example.host')->group(function () {
    Route::get('dashboard/{search}', function ($account, $search) {
        //
    });
});
```

**Note:**

Ensure, the sub-domain routes are reachable by defining before root domain routes definitions. Because, it will overwritten by root domain routes definitions due to same URI path.


### &#10022; Model Binding:

When it is required to bind model class instance, instead of using model class later in the callback. 

For example, Consider Post model. Instead of injecting post id, bind `Post` model instance that matches given post id.

It can be done by implicit and explicit.

### &#10022; Implicit Model Binding:

Laravel might try to resolves Eloquent models defined in routes or controller actions, whose type-hinted variable names that matches a route parameter name. 

If the post id is not matched with any of the record in the database, then it will fallback to 404 page not found error.

*Example:*

*Usage 1:*

```php
use App\Models\Post;
 
Route::get('/post/{post}', function (Post $post) {
    return view('post.article', ['title' => $post->title, 'content' => $post->content]);
});
```

*Usage 2:*

```php
use App\Http\Controllers\PostController;
use App\Models\Post;
 
// Route definition...
Route::get('/post/{post}', [PostController::class, 'show']);
 
// Controller method definition...
public function show(Post $post)
{
    return view('post.article', ['title' => $post->title, 'content' => $post->content]);
}
```

### &#10022; Model Binding For Soft Deleted Records:

When a model contains a soft deleted record, then Laravel treats as deleted by checking a column `deleted_at` is not null. 

When route parameter contains id for the model binding and their corresponding record was soft deleted, then it would through fallback 404 page not found error. To avoid 404, and consider those record as well to match when a model binding using method `withTrashed`. 

```php
use App\Models\User;
 
Route::get('/users/{user}', function (User $user) {
    // it will match and get soft deleted record as well
})->withTrashed();
```

### &#10022; Custom Column Key:

Sometimes to resolve Eloquent models using a different column other than id. To do so, by specifying the column in the route parameter definition.

*Example:*

```php
use App\Models\Post;
 
Route::get('/posts/{post:post_name}', function (Post $post) {
    // post/article%20name corresponding post/article name
    return view('post.article', ['title' => $post->title, 'content' => $post->content]);
});
```

**Default Route Key Definition:**

To change default route key for model binding, which is possible by override method in the corresponding model class with `getRouteKeyName` method. But it does not affect default primary key for Eloquent model methods. 

```php
/**
 * Get default route key name for the current model.
 *
 * @return string
 */
public function getRouteKeyName(): string
{
    return 'post_name';
}
```

```php
use App\Models\Post;
 
Route::get('/posts/{post}', function (Post $post) {
    // post/article%20name corresponding post/article name
    return view('post.article', ['title' => $post->title, 'content' => $post->content]);
});
```

**Multiple Model Binding:**

When first model is parent of second model, to scope these models by binding for corresponding route. For example, consider `Post` model is child of `User` model. To retrieve post by post id corresponding to the user id.

```php
use App\Models\User;
use App\Models\Post;
 
Route::get('/users/{user}/posts/{post:post_id}', function (User $user, Post $post) {
    return view('post.article', ['title' => $post->title, 'content' => $post->content]);
});
```

### &#10022; Key Scoping:

If the route definitions has multiple route parameters, that has corresponding model binding. By default, Laravel try to resolve by utilizing model relationship. If it is not then it can be enforced to treat as `parent` and `child` relationship between those models with method `scopeBinding`.

It adds extra layer of security that when a model is related with another. For example, `Author` model is parent of `Post` model. It is not possible to access any of the post eventhough the post id is known. But it allow to access post belongs to the author id.

If it is not found then it fallback 404 page not found error.

```php
use App\Models\Author;
use App\Models\Post;
 
Route::get('/author/{author}/posts/{post}', function (Author $author, Post $post) {
    return view('post.article', ['title' => $post->title, 'content' => $post->content]);
})->scopeBindings();
```

**Scope Binding as Route Group:**

To instruct and define the entire routes has to bind with scope binding.

```php
Route::scopeBindings()->group(function () {
    Route::get('/author/{author}/posts/{post}', function (Author $author, Post $post) {
        return view('post.article', ['title' => $post->title, 'content' => $post->content]);
    });
});
```

### &#10022; Missing Model Behavior:

If a model is not found when it is defined/bound implicitly, then a 404 HTTP response will be generated. But, it can be customized by using `missing` method when defining routes. The missing method accepts argument as a closure that will be invoked when model is not found which is implicitly bound.

```php
use App\Http\Controllers\PostController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Redirect;
 
Route::get('/post/{post:post_name}', [PostController::class, 'show'])
        ->name('post.article')
        ->missing(function (Request $request) {
            return Redirect::route('post.recent');
        });
```

### &#10022; Explicit Model Binding:

Laravel gives possibility to bind models explicitly through `boot` method inside `App\Providers\RouteServiceProvider`. It tells how route parameters correspond to the models.

If the correspond route parameter is not match with model then it would fallback 404 page not found error.

```php
use App\Models\User;
use Illuminate\Support\Facades\Route;
 
/**
 * Define your route model bindings, pattern filters, etc.
 *
 * @return void
 */
public function boot()
{
    Route::model('user', User::class);
    // ...
}
```

In `web.php`:

```php
use App\Models\User;
 
Route::get('/users/{user}', function (User $user) {
    //
});
```

### &#10022; Custom Model Binding:

In order to define custom model binding logic then it can be done by define inside `boot` method of `App\Providers\RouteServiceProvider` by using method `bind`.

```php
use App\Models\User;
use Illuminate\Support\Facades\Route;
 
/**
 * Define your route model bindings, pattern filters, etc.
 *
 * @return void
 */
public function boot()
{
    Route::bind('user', function ($value) {
        return User::where('name', $value)->firstOrFail();
    });
}
```

**Define Inside Model Class:**

```php
// User.php user model class

/**
 * Retrieve the model for a bound value.
 *
 * @param  mixed  $value
 * @param  string|null  $field
 * @return \Illuminate\Database\Eloquent\Model|null
 */
public function resolveRouteBinding($value, $field = null)
{
    return $this->where('name', $value)->firstOrFail();
}
```

In `web.php`:

```php
use App\Models\User;
 
Route::get('/users/{user}', function (User $user) {
    // route parameter will match with User model field `name` instead of `primary key`
});
```

**For child method scope binding:**

When route definitions has scoped model binding implicitly then it can resolved child model binding behavior of the parent model by override method `resolveChildRouteBinding` inside parent model.

```php
/**
 * Retrieve the child model for a bound value.
 *
 * @param  string  $childType
 * @param  mixed  $value
 * @param  string|null  $field
 * @return \Illuminate\Database\Eloquent\Model|null
 */
public function resolveChildRouteBinding($childType, $value, $field)
{
    return parent::resolveChildRouteBinding($childType, $value, $field);
}
``` 

### &#10022; Fallback Routes:

Laravel route `fallback` method will be executed when no other route matches for the incoming request. Any of the unmatched or unhandled requests will give a 404 page not found via Laravel application exception handler. 

For all middleware in the `web` middleware group will apply to the route.

```php
use Illuminate\Support\Facades\Route;

/**
 * All route definitions for web 
 */

// Starting of route definitions
// ...
// End of all route definitions

Route::fallback(function () {
    //
});
```

**Note:**

- Always, the fallback route definition should be the last route registered by the application.


### &#10022; Request Rate Limiting:

**Define Rate Limits:**

Laravel provides possibilities to add rate limiting services, that may restrict the amount of traffic based on certain conditions for a given route or group of routes. 

It can be defined within `App\Providers\RouteServiceProvider` class. It can be done within the `configureRateLimiting` method.

Rate limiters are defined using `Illuminate\Support\Facades\RateLimiter` class instance using the `for` method of it.

That `for` method accepts limiter `name` and `closure` callback or definition which returns limit configuration to it. 

Limit can be configured using `Illuminate\Cache\RateLimiting\Limit` class instance with their methods such as `none()`, `perMinute($value)` and `by()`.   

If request rate limits exceeds for incoming request then a response returned by Laravel with a 429 HTTP status code.

In `RouteServiceProvider` class:

Which defines and allow request of 1000 times from same user within a minutes and if exceeds then revoke the connection.

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;
 
/**
 * Configure the rate limiters for the application.
 *
 * @return void
 */
protected function configureRateLimiting()
{
    RateLimiter::for('global', function (Request $request) {
        return Limit::perMinute(1000);
    });
}
```

**Custom Rate Limit Response:**

In `RouteServiceProvider` class:

```php
RateLimiter::for('global', function (Request $request) {
    return Limit::perMinute(1000)->response(function () {
        return response('Exceeds allowed usage', 429);
    });
});
```

**Rate Limit For Different Users:**

It is possible to set rate limit for different users such as guest, authorized user and premium or VIP user.

In `RouteServiceProvider` class:

Which does not set any restriction for upload to VIP customers, but not for all users.

```php
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()->vipCustomer()
                ? Limit::none()
                : Limit::perMinute(100);
});
```

**Rate Limit by IP address:**

In `RouteServiceProvider` class:

Rate limit between VIP customer and all other users.

```php
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()->vipCustomer()
                ? Limit::none()
                : Limit::perMinute(100)->by($request->ip());
});
```

Rate limit between authorized user and all other users.

```php
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()
                ? Limit::perMinute(100)->by($request->user()->id)
                : Limit::perMinute(10)->by($request->ip());
});
```

**Multiple Rate Limits:**

In `RouteServiceProvider` class:

```php
RateLimiter::for('login', function (Request $request) {
    return [
        Limit::perMinute(50), // overall request limits per minute
        Limit::perMinute(3)->by($request->input('email')), // which allow until limit exceeds for same email address 
    ];
});
```

**Attach Route Limiter to Routes:**

It is possible to set rate limit with middleware group using `throttle` middleware, which accepts the name of the rate limiter defined in `RouteServiceProvider`.

In `web.php` file:

```php
Route::middleware(['throttle:uploads'])->group(function () {
    Route::post('/audio/upload', function () {
        //
    });
 
    Route::post('/video/upload', function () {
        //
    });
});
```

Refer official documentation for Throttling With Redis.

### &#10022; Form Method Spoofing:

This will set appropriate methods to the form data. Since, HTML forms do not support PUT, PATCH, or DELETE method actions. So, when defining a route with following methods PUT, PATCH, or DELETE, then it need to add a hidden `_method` field in the form. This will sent with form submission.

```php
<form action="/article" method="POST">
    <input type="hidden" name="_method" value="PUT">
    <input type="hidden" name="_token" value="{{ csrf_token() }}">
</form>
```

**Using blade template:**

```php
<form action="/example" method="POST">
    @method('PUT') <!-- it is equivalent to hidden _method field -->
    @csrf 
</form>
```

### &#10022; Current Route Information:

Facades Route class provides methods to access route definition details for incoming request. 

```php
use Illuminate\Support\Facades\Route;
 
$route = Route::current(); // Illuminate\Routing\Route
$name = Route::currentRouteName(); // string
$action = Route::currentRouteAction(); // string
```

Refer API documentation for both the underlying class of the Route facade and Route instance to know all possible methods available on the router and route classes.

### &#10022; CORS:

Laravel automatically respond to CORS OPTIONS HTTP requests. All CORS settings may be found and configured in `config/cors.php` CORS configuration file. This OPTIONS request will handled by `HandleCors` middleware. That middleware included by default in global middleware stack.

Global middleware stack is located in `App\Http\Kernel`.

### &#10022; Route Caching:

When deploying application to production, Take advantage of Laravel route cache. This route cache will decrease the amount of time to register all of the routes. It is possible to generate a route cache by using `Artisan` command.

```bash
php artisan route:cache
```

After route cached that will be utilized on every request. 

**Note:**

- If any of the new routes added in the routes, then it is important to generate a fresh route cache. 

**Clear Route Cache:**

```bash
php artisan route:clear
```

---
[&#8682; To Top](#-routing)

[&#10094; Previous Topic](./artisan-commands.md) &emsp; [Next Topic &#10095;](./middleware.md)

[&#8962; Goto Home Page](../README.md)