## &#10162; URL Generation:

Laravel provides several helpers. Helpers for generating URLs for an application are primarily helpful when building links in templates and API response.

### &#9780; Overview:

1. [Generating URLs](#-generating-urls)
2. [Accessing The Current URL](#-accessing-the-current-url)
3. [URLs For Named Routes](#-urls-for-named-routes)
4. [Eloquent Models](#-eloquent-models)
5. [Signed URLs](#-signed-urls)
6. [Validating Signed Route Requests](#-validating-signed-route-requests)
7. [URLs For Controller Actions](#-urls-for-controller-actions)
8. [Default Values](#-default-values)

### &#10022; Generating URLs:

- The `url` helper can create any URL for an application.
- The made URL uses the scheme (HTTP or HTTPS) and host from the current request.

*Example:*

```php
$post = App\Models\Post::find(1);
echo url("/posts/{$post->id}");
```

*Output:*

```
http://example.host/posts/1
```

### &#10022; Accessing The Current URL:

- The `url` helper gives an `Illuminate\Routing\UrlGenerator` class object when no path is given.
- This object lets a user find details about the current URL.

**Current URL For Empty Query String:**

- `url()->current()` shows the current URL without the query string.

```php
echo url()->current();
```

**Current URL With Query String:**

- `url()->full()` shows the current URL with the query string.

```php
echo url()->full();
```

**Previous Request URL:**

- `url()->previous()` shows the full URL of the last request.

```php
echo url()->previous();
```

**Using the URL Facade:**

- A user can also use the `URL` facade to do these actions.

```php
use Illuminate\Support\Facades\URL;

echo URL::current();
```

### &#10022; URLs For Named Routes:

- The `route` helper creates URLs for routes with names.
- Named routes let a user make URLs without depending on the route's real URL.
- If the route's URL changes, a user does not need to change the `route` function calls.
- Consider, an application has a route as below:

```php
Route::get('/post/{post}', function (Post $post) {
    //
})->name('post.show');
```

- A user can make a URL to this route using the `route` helper function:

```php
echo route('post.show', ['post' => 1]);
// http://example.host/post/1
```

- The `route` helper can also make URLs for routes with many parameters:

```php
Route::get('/post/{post}/comment/{comment}', function (Post $post, Comment $comment) {
    //
})->name('comment.show');

echo route('comment.show', ['post' => 1, 'comment' => 3]);

// http://example.host/post/1/comment/3
```

- Extra array items that do not match the route's parameters go into the URL's query string:

```php
echo route('post.show', ['post' => 1, 'search' => 'rocket']);

// http://example.host/post/1?search=rocket
```

### &#10022; Eloquent Models:

- URLs are often made using the route key usually the main primary key of Eloquent models.
- A user can give Eloquent models as parameter values.
- The `route` helper will get the model's route key automatically:

```php
echo route('post.show', ['post' => $post]);
```

### &#10022; Signed URLs:

- Laravel allows making `signed` URLs for named routes.
- These URLs have a `signature` hash in the query string.
- Laravel uses this hash to check if the URL was changed after it was made.
- Signed URLs are good for public routes that need protection from URL changes.
- For example, a user might use signed URLs for public `unsubscribe` links in emails.
- To make a signed URL for a named route, use the `signedRoute` method of the `URL` facade:

```php
use Illuminate\Support\Facades\URL;

return URL::signedRoute('unsubscribe', ['user' => 1]);
```

- To make a temporary signed route URL that expires, use `temporarySignedRoute`.
- Laravel checks the expiration timestamp in the signed URL.

```php
use Illuminate\Support\Facades\URL;

return URL::temporarySignedRoute(
    'unsubscribe', now()->addMinutes(30), ['user' => 1]
);
```

### &#10022; Validating Signed Route Requests:

- To check if a request has a valid signature, call `hasValidSignature` on the `Request`:

```php
use Illuminate\Http\Request;

Route::get('/unsubscribe/{user}', function (Request $request) {
    if (! $request->hasValidSignature()) {
        abort(401);
    }

    // ...
})->name('unsubscribe');
```

- Or, add the `Illuminate\Routing\Middleware\ValidateSignature` middleware to the route.
- Add this middleware a key in the HTTP kernel's `routeMiddleware` array:

```php
/**
 * The application's route middleware.
 *
 * These middleware may be assigned to groups or used individually.
 *
 * @var array
 */
protected $routeMiddleware = [
    'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
];
```

- After registering the middleware, add it to a route.
- If the request has an invalid signature, the middleware returns a 403 HTTP response:

```php
Route::post('/unsubscribe/{user}', function (Request $request) {
    // ...
})->name('unsubscribe')->middleware('signed');
```

**Responding To Invalid Signed Routes:**

- When a user visits an expired signed URL, they get a 403 error page.
- A user can change this by making a custom `renderable` closure for `InvalidSignatureException` in the exception handler.
- This closure should return an HTTP response:

```php
use Illuminate\Routing\Exceptions\InvalidSignatureException;

/**
 * Register the exception handling callbacks for the application.
 *
 * @return void
 */
public function register()
{
    $this->renderable(function (InvalidSignatureException $e) {
        return response()->view('error.link-expired', [], 403);
    });
}
```

### &#10022; URLs For Controller Actions:

- The `action` function makes a URL for a controller action.

```php
use App\Http\Controllers\HomeController;

$url = action([HomeController::class, 'index']);
```

- If the controller method takes route parameters, a user can give an array of route parameters as the second argument.

```php
$url = action([UserController::class, 'profile'], ['id' => 1]);
```

**Default Values:**

- Some applications may need request-wide default values for certain URL parameters.
- Imagine many routes have a `{locale}` parameter:

```php
Route::get('/{locale}/posts', function () {
    //
})->name('post.index');
```

- It is annoying to always pass the locale when calling the `route` helper.
- A user can use the `URL::defaults` method to set a default value for this parameter during the current request.
- A user can call this method from a route middleware to access the current request:

```php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Support\Facades\URL;

class SetDefaultLocaleForUrls
{
    /**
     * Handle the incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return \Illuminate\Http\Response
     */
    public function handle($request, Closure $next)
    {
        URL::defaults(['locale' => $request->user()->locale]);

        return $next($request);
    }
}
```

- After setting the default value for the locale parameter, a user does not need to pass its value when making URLs with the `route` helper.

### &#10022; Default Values:

- Setting URL default values can cause problems with Laravel's implicit model bindings.
- A user should make sure the middleware that sets URL defaults runs before Laravel's `SubstituteBindings` middleware.
- A user can do this by placing the middleware before `SubstituteBindings` in the `$middlewarePriority` property of the application's HTTP kernel.
- The `$middlewarePriority` property is in the `Illuminate\Foundation\Http\Kernel` class.
- A user can copy and change this definition in the application's HTTP kernel:

```php
/**
 * The priority-sorted list of middleware.
 *
 * This forces non-global middleware to always be in the given order.
 *
 * @var array
 */
protected $middlewarePriority = [
    // ...
    \App\Http\Middleware\SetDefaultLocaleForUrls::class,
    \Illuminate\Routing\Middleware\SubstituteBindings::class,
    // ...
];
```

---
[&#8682; To Top](#-url-generation)

[&#10094; Previous Topic](./csrf-protection.md) &emsp; [Next Topic &#10095;](./sessions.md)

[&#8962; Goto Home Page](../README.md)