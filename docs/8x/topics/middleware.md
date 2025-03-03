## &#10162; Middleware:

Middleware concept is used for inspecting and filtering incoming HTTP requests in the application. In Laravel, middleware can be defined and utilized built-in middleware based on the requirements. Such as, auth middleware can be used for check whether user authenticated or not. Beyond authentication, middleware can used for many important task before a incoming request reaching the application.

Also, it acts as a filter and security layer of the application. 

Series of middleware is a best practice instead of creating all in one. The incoming request passes through each layer of middleware. Each layer examine and inspect the request before reach the application.

All middleware are resolved via the service container.

### &#9780; Overview:
1. [Defining Middleware](#-defining-middleware)
2. [Middleware Response](#-middleware-response)
3. [Registering Middleware](#-registering-middleware)
4. [Excluding Middleware](#-excluding-middleware)
5. [Middleware Groups](#-middleware-groups)
6. [Sorting And Priorities Middlewares](#-sorting-and-priorities-middlewares)
7. [Middleware Parameters](#-middleware-parameters)
8. [Terminable Middleware](#-terminable-middleware)

### &#10022; Defining Middleware:

Laravel provides several middlewares for authentication, CSRF Protection, throttle and so on. These all middleware are located in the `app/Http/Middleware` directory. 

**Create Middleware by Artisan Command:**

*Syntax:*

```bash
php artisan make:middleware <name>
```

*Example:*

```bash
php artisan make:middleware TokenValidation
```

The above command will create `TokenValidation` class under `app\Http\Middleware` directory. 

**Middleware Definitions:**

```php
namespace App\Http\Middleware;
 
use Closure;
 
class TokenValidation
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if ($request->input('token') !== 'secret-token') {
            return redirect('home');
        }
 
        return $next($request);
    }
}
```

### &#10022; Middleware Response:

Laravel middleware can handle and perform tasks before and after the request.

**Middleware Before Request End Point:**

```php

namespace App\Http\Middleware;
 
use Closure;
 
class BeforeMiddleware
{
    public function handle($request, Closure $next)
    {
        // Perform action
 
        return $next($request); // pass it to next closure such as another middleware or callback
    }
}
```

**Middleware After Request End Point:**

```php
namespace App\Http\Middleware;
 
use Closure;
 
class AfterMiddleware
{
    public function handle($request, Closure $next)
    {
        $response = $next($request); // get response after request end point
 
        // Perform action
 
        return $response; // return response to user
    }
}
```

### &#10022; Registering Middleware:

Middlewares can take into the action by register either global or assign to specific group.

**Global Middleware Registration:**

If it requires to utilize middleware at every HTTP request in the application, then it possible by list those middlewares in the `$middleware` property of `app/Http/Kernel.php` class.

**Assign Middleware to Specific Groups:**

Before assign middleware to specific routes, it should be added as the middleware a key in `app/Http/Kernel.php` file. Laravel has defined default entires for built-in middleware. It offers to  add own middleware to this list and assign it a key for choosing. If it is not added, then it requires to mention fully qualified class name with namespace.

In `App\Http\Kernel` class:

```php
protected $routeMiddleware = [
    'auth' => \App\Http\Middleware\Authenticate::class,
    // ...
    'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
    // ...
    'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
];
```

Once middleware defined in the HTTP kernel, It can be used with `middleware` method to assign middleware to a route.

In `web.php` file:

```php
Route::get('/user', function () {
  // ... 
})->middleware('auth');
```

It can be assigned with multiple middlewares to the specific route.

In `web.php` file:

```php
Route::get('/user', function () {
    //
})->middleware(['auth', 'throttle']);
```

If it is not added key for own middleware in middleware entries or it allows to specify middleware directly.

```php
use App\Http\Middleware\IsVIPUser;
 
Route::get('/explore', function () {
    //
})->middleware(IsVIPUser::class);
```

### &#10022; Excluding Middleware:

When a middleware is assigned to a group of routes, where some of the route does not required to use that or need to ignore using that middleware. This can be done by `withoutMiddleware` method.

In `web.php` file:

```php
use App\Http\Middleware\IsVIPUser;
 
Route::middleware([IsVIPUser::class])->group(function () {
    Route::get('/', function () {
        //
    })->withoutMiddleware([IsVIPUser::class]);
 
    Route::get('/explore', function () {
        //
    });
});
```

**Note:**

- `withoutMiddleware` method can only prevent being utilized route middleware and does not applicable to global middleware.

### &#10022; Middleware Groups:

To assign multiple middlewares to routes is possible by specifying the group key which is defined in `App\Providers\RouteServiceProvider` class as `$middlewareGroups` property. This property has associative array of middleware groups. By default, Laravel built-in middlewares were added as individual group for `web` and `api`.

In `App\Providers\RouteServiceProvider` class:

```php
protected $middlewareGroups = [
    'web' => [
        // ...
        \App\Http\Middleware\VerifyCsrfToken::class,
        // ...
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],
 
    'api' => [
        'throttle:api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],
];
``` 

These group of middleware can be assigned to routes using the `middleware` method by specifying the middleware group key. 

By default, `web` and `api` middleware groups are applied to `web.php` and `api.php` by `App\Providers\RouteServiceProvider` class. 

In `web.php` file:

```php
Route::get('/', function () {
    //
})->middleware('web'); // by default applied no need to specify
 
Route::middleware(['web'])->group(function () {
    //
}); // by default applied no need to specify

Route::middleware(['custom'])->group(function() {
    //
});
```

### &#10022; Sorting And Priorities Middlewares:

By rare in case, the certain middlewares need to be evaluated in the order or need to assign priority for each middleware. It is possible by define the order or priority of middlewares inside `app/Http/Kernel.php` with the property `$middlewarePriority`.

By default, it is not exist in the `Kernal.php`, but it can create as below:

In `app/Http/Kernal.php` class:

```php
/**
 * middleware priority property
 *
 * @var array
 */
protected $middlewarePriority = [
    \Illuminate\Session\Middleware\StartSession::class,
    \Illuminate\Routing\Middleware\ThrottleRequests::class,
    \Illuminate\Contracts\Auth\Middleware\AuthenticatesRequests::class,
    \Illuminate\Session\Middleware\AuthenticateSession::class,
    \Illuminate\Routing\Middleware\SubstituteBindings::class,
    \Illuminate\Auth\Middleware\Authorize::class,
];
```

### &#10022; Middleware Parameters:

Middleware may receive or need of parameters in order to perform certain task. Additional parameters for middlewares will be passed to the middleware after the `$next` argument:

```php
namespace App\Http\Middleware;
 
use Closure;
 
class HasValidUserRole
{
    /**
     * Handle the incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @param  string  $role
     *
     * @return mixed
     */
    public function handle($request, Closure $next, string $role): mixed
    {
        if (! $request->user()->hasRole($role)) {
            return redirect('dashboard');
        }
 
        return $next($request);
    }
}
```

In `web.php` file:

Middleware parameters can be specified when define the route. It mention as middleware key name followed by `:` colon and parameter values. If more than one parameters then it will be separated by comma.

```php
Route::put('/products/{id}', function ($id) {
    //
})->middleware('role:staff');
```

### &#10022; Terminable Middleware:

Sometimes, it requires to do some task after send response to the user. It is possible by define `terminate` method with `$request` and `$response` instance as arguments.

The terminate method should receive both the request and the response object. Once middleware with terminate method defined then it should add to the list of routes or global middleware in the `app/Http/Kernel.php` file.

```php
namespace App\Http\Middleware;
 
use Closure;
 
class VerifyMiddleware
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     *
     * @return mixed
     */
    public function handle(Request $request, Closure $next): mixed
    {
        // send authentication mail
        return $next($request);
    }
 
    /**
     * Handle tasks after the response has been sent to the browser.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Illuminate\Http\Response  $response
     *
     * @return void
     */
    public function terminate(Request $request, Response $response): void
    {
        // send success mail
    }
}
```

When terminate method invoked from the middleware, then Laravel will resolve with a fresh instance of the middleware from the service container. But if it requires to use same instance of middleware instead of fresh, then it requires to register middleware in container's singleton method inside `AppServiceProvider` class.

```php
use App\Http\Middleware\VerifyMiddleware;
 
/**
 * Register any application services.
 *
 * @return void
 */
public function register()
{
    $this->app->singleton(VerifyMiddleware::class);
}
```

---
[&#8682; To Top](#-middleware)

[&#10094; Previous Topic](./routing.md) &emsp; [Next Topic &#10095;](./csrf-protection.md)

[&#8962; Goto Home Page](../README.md)