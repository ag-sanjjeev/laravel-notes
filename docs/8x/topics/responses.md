## &#10162; Responses:

### &#9780; Overview:

1. [Creating Responses](#-creating-responses)
    - [Response Object](#-response-object)
    - [Response Eloquent Model](#-response-eloquent-model)
    - [Cache Control](#-cache-control)
    - [Attaching Headers To Responses](#-attaching-headers-to-responses)
    - [Attaching Cookies To Responses](#-attaching-cookies-to-responses)
    - [Generating Cookie Instances](#-generating-cookie-instances)
    - [Cookies Queue](#-cookies-queue)
    - [Cookies Expiry](#-cookies-expiry)
    - [Cookies and Encryption](#-cookies-and-encryption)
2. [Redirects](#-redirects)
    - [Redirect Back](#-redirect-back)
    - [Redirect To Named Routes](#-redirect-to-named-routes)
    - [Redirect To Controller Actions](#-redirect-to-controller-actions)
    - [Redirect To External Domains](#-redirect-to-external-domains)
    - [Redirect With Flashed Session Data](#-redirect-with-flashed-session-data)
    - [Redirect With Input](#-redirect-with-input)
3. [View Responses](#-view-responses)
4. [JSON Responses](#-json-responses)
5. [File Downloads](#-file-downloads)
6. [File Responses](#-file-responses)
7. [Response Macros](#-response-macros)

### &#10022; Creating Responses:

Creating response in Laravel is possible from the route definition itself. All routes and controllers should return a response which is sent back to the user. Whether the route or controller, Laravel converts the string or array into a fully qualified HTTP response.

**HTTP Response From String:**

```php
Route::get('/', function () {
    return 'Hello World';
});
```

**HTTP Response From Array:**

Laravel will automatically convert given array into a JSON response.

```php
Route::get('/', function () {
    return [1, 2, 3];
});
```

### &#10022; Response Object:

Generally, route actions return strings and arrays. And it also return `Illuminate\Http\Response` class instances or views.

`Response` class instance allows to specify the HTTP response status code and headers. `Response` class is extends from the `Symfony\Component\HttpFoundation\Response` class, which has many methods for making HTTP responses.

```php
Route::get('/home', function () {
    return response('Hello World', 200)
           ->header('Content-Type', 'text/plain');
});
```

### &#10022; Response Eloquent Model:

Eloquent ORM models and collections can be returned directly from routes and controllers. When this happens, Laravel will automatically change the models and collections into JSON responses, and it will respect the models hidden attributes by hide them.

In `web.php` file:

```php
use App\Models\Post;
use App\Models\User;

Route::get('/post/{post}', function (Post $post) {
    return $post;
});

Route::get('/user/{user}', function (User $user) {
    return $user;
});
```

### &#10022; Cache Control:

Cache Control Middleware

`cache.headers` middleware in Laravel, is used to set the Cache-Control header for route groups from the route definition. Directives are given as `snake case` and split by a semicolon. If `etag` is in the directive list, then MD5 hash of the response content is set as the `ETag` identifier.

```php
Route::middleware('cache.headers:public;max_age=2628000;etag')->group(function () {
    Route::get('/privacy', function () {
        // ...
    });

    Route::get('/terms', function () {
        // ...
    });

    Route::get('/contact', function () {
        // ...
    });
});
```

### &#10022; Attaching Headers To Responses:

With `Response` instance `header` method, It is possible to add multiple headers to the response before it is sent to the user.

```php
return response($content)
                ->header('Content-Type', $type)
                ->header('X-Header-One', 'Header Value')
                ->header('X-Header-Two', 'Header Value');
```

Or, the `withHeaders` method can be used to specify multiple headers as an array of headers to the response.

```php
return response($content)
                ->withHeaders([
                        'Content-Type' => $type,
                        'X-Header-One' => 'Header Value',
                        'X-Header-Two' => 'Header Value',
                ]);
```

### &#10022; Attaching Cookies To Responses:

Attach a cookie with outgoing response instance by using the `cookie` method. Specify the `name`, `value`, and valid `minutes` of the cookie to this method.

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes
);
```

The cookie method also accepts more arguments used less often. It possible to set `path`, `domain`, `secure` and `httpOnly`.

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
);
```

### &#10022; Generating Cookie Instances:

To generate a `Symfony\Component\HttpFoundation\Cookie` instance that can be attached to a response instance later, use the global cookie helper in Laravel, which will not sent to the client unless attached to a response instance.

```php
$cookie = cookie('name', 'value', $minutes);

return response('Hello World')->cookie($cookie);
```

### &#10022; Cookies Queue:

To ensure a cookie is sent with the outgoing response when response instance is not yet created, use `queue` method from `Illuminate\Support\Facades\Cookie` class for attachment to the response when sent. The queue method takes the arguments to create a cookie instance. These cookies will be attached to the outgoing response before it is sent to the browser.

```php
use Illuminate\Support\Facades\Cookie;

Cookie::queue('name', 'value', $minutes);
```

### &#10022; Cookies Expiry:

To remove a cookie by expire it with the `withoutCookie` method to an outgoing response.

```php
return response('Hello World')->withoutCookie('name');
```

If you don't have an outgoing response instance yet, use the `expire` method to expire a cookie.

```php
Cookie::expire('name');
```

### &#10022; Cookies and Encryption:

Laravel generates cookies with encrypted and signed to preventing client modification or reading. To disable encryption for some application cookies, then use the `$except` property of the `App\Http\Middleware\EncryptCookies` middleware class found in the `app/Http/Middleware` directory.

```php
/**
 * The names of the cookies that should not be encrypted.
 *
 * @var array
 */
protected $except = [
    'cookie_name',
];
```

### &#10022; Redirects:

Redirect responses are `Illuminate\Http\RedirectResponse` class instances, and it has headers to redirect the user to another URL. There are many alternate ways to redirect with `RedirectResponse` instance. The easiest way is use with global redirect helper.

```php
Route::get('/dashboard', function () {
    return redirect('home/dashboard');
});
```

### &#10022; Redirect Back:

Sometimes, it required to redirect back to the user with their previous location when a submitted form is invalid or any other similar scenario. To achieve redirect with the global `back` helper function. This uses the session and ensure the route calling the back function uses the `web` middleware group.

```php
Route::post('/product/store', function () {
    // Validate the request...

    return back()->withInput();
});
```

### &#10022; Redirect To Named Routes:

When invoke redirect helper without parameters, `an Illuminate\Routing\Redirector` instance is used. To use `RedirectResponse` instance with redirect use `route` method. Which redirect to named routes.

```php
return redirect()->route('login');
```

**Redirect Route with Route Parameters:**

```php
// For a route with the following URI: /product/{id}

return redirect()->route('product', ['id' => 1]);
```

**Redirect Route with Eloquent Model:**

If redirect with named route having `id` route parameter from an Eloquent model. It is possible with pass the model, which extracts `id` automatically based on the specified model.

```php
// For a route with the following URI: /profile/{id}

return redirect()->route('profile', [$user]);
```

**Customize Column Key Bind for Route With Eloquent Model:**

It is possible to customize the route parameter in route definitions such as `/post/{post:post_id}` or override the `getRouteKey` method on used Eloquent model.

```php
/**
 * Get the value of the model's route key.
 *
 * @return mixed
 */
public function getRouteKey()
{
    return $this->post_id; // sets post_id column for route parameter
}
```

Refer [Custom Column Key](./routing.md#-custom-column-key) in Routings.

### &#10022; Redirect To Controller Actions:

To make redirects to controller actions. Pass the controller and action name to the action method.

```php
use App\Http\Controllers\PostController;

return redirect()->action([PostController::class, 'index']);
```

**Controller Action with Parameters:**

```php
return redirect()->action([PostController::class, 'show'], ['id' => 1]);
```

### &#10022; Redirect To External Domains:

Sometimes, it required to redirect to outside of the application. Use the `away` method, which makes a `RedirectResponse` without extra URL encoding, validation, or verification.

```php
return redirect()->away('https://github.com/ag-sanjjeev/');
```


### &#10022; Redirect With Flashed Session Data:

Redirecting with flashing data to the session after an action succeeds or error.

```php
Route::post('/post/store', function () {
    // ...
    return redirect('dashboard')->with('success', 'Post created!');
});
```

After redirected, It is possible to show the flashed message from the session. With `Blade` Template syntax.

```php
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```

### &#10022; Redirect With Input:

Use the `withInput` method of the `RedirectResponse` class instance to flash the current request input data to the session before redirecting the user. This is useful for repopulate form when the user has a validation error. Once the input is flashed to the session, then it can easily accessed during the next request.

```php
return back()->withInput();
```

### &#10022; View Responses:

To control the response status and headers while returning a view as the response content, use the `view` method.

```php
return response()
                ->view('dashboard', $data, 200)
                ->header('Content-Type', $type);
```

If it is not required to pass a custom HTTP status code or custom headers then use the global `view` helper function.


```php
return view('dashboard');
```

Refer More for [Views](./views.md)

### &#10022; JSON Responses:

The `json` method sets the `Content-Type` header to `application/json` and converts the given array to JSON using the `json_encode` PHP function.

```php
return response()->json([
    'name' => 'Kumar',
    'state' => 'TamilNadu',
]);
```

To create a JSONP response, use the `json` method with the `withCallback` method.

```php
return response()
                ->json(['name' => 'Kumar', 'state' => 'TamilNadu'])
                ->withCallback($request->input('callback'));
```

### &#10022; File Downloads:

The `download` method makes a response that forces the user browser to download the file from given path. The `download` method requires `filepath` parameter, second optional `filename` and third optional `headers` parameters.

```php
return response()->download($pathToFile);
```

```php
return response()->download($pathToFile, $name, $headers);
```

**Streamed Downloads:**

To turn the string response of an operation into a downloadable response without writing the operation contents to disk, use the `streamDownload` method. This method takes a `callback`, `filename`, and an optional array of headers as arguments.

```php
use App\Services\GitHub;

return response()->streamDownload(function () {
    echo GitHub::api('repo')
                ->contents()
                ->readme('ag-sanjjeev', 'laravel-notes')['Overview'];
}, 'readme.md');
```

### &#10022; File Responses:

The `file` method can display a file like an image or PDF directly in the user's browser instead of download. This method takes the file's path as its first argument and an array of headers as its second argument.

```php
return response()->file($pathToFile);
```

```php
return response()->file($pathToFile, $headers);
```

### &#10022; Response Macros:

To define a custom response for re-use in routes and controllers, use the macro method on the Response facade. Call this method from the boot method of a service provider, like the `App\Providers\AppServiceProvider` service provider.

```php
namespace App\Providers;

use Illuminate\Support\Facades\Response;
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
        Response::macro('capitalize', function ($value) {
            return Response::make(strtoupper($value));
        });
    }
}
```

The macro function takes a name as its first argument and a closure as its second argument. The macro's closure runs when calling the macro name from a ResponseFactory implementation or the response helper:

```php
return response()->capitalize('hello world');
```

---
[&#8682; To Top](#-responses)

[&#10094; Previous Topic](./requests.md) &emsp; [Next Topic &#10095;](./views.md)

[&#8962; Goto Home Page](../README.md)