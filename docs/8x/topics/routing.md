## &#10162; Routing:

In Laravel, routing is important to defines the mapping between URLs (Uniform Resource Locators) and the code that should be executed when those URLs are accessed. Where the code means controller method or closure should handle a specific request.

### &#9780; Overview:
1. [Route Definition](#-route-definition),
2. [Dependency Injection](#-dependency-injection)
3. [CSRF Protection](#-csrf-protection)
4. [Redirect Routes](#-redirect-routes)
5. [View Routes](#-view-routes)
6. [Route Parameters](#-route-parameters)
7. [Optional Route Parameters](#-optional-route-parameters)
8. [RegEx Constraints](#-regex-constraints)
. [Named Routes](#-)
. [Route Groups](#-)
	- [Controllers](#-)
	- [Middleware](#-)
	- [Route Prefixes](#-)
	- [Route Name Prefixes](#-)
	- [Subdomain Routing](#-)
. [Model Binding](#-)
. [Fallback Routes](#-)
. [Request Rate Limiting](#-)
. [Form Method Spoofing](#-)
. [Current Route Information](#-)
. [CORS](#-)
. [Route Caching](#-)

### &#10022; Route Definition:

Routes are located at routes directory. Where it has web.php, api.php and so on. That defines routes for different interfaces such as web and api correspondingly. 

- routes/web.php file defines routes for web interface. These routes are assigned with web middleware group, which provides features like session state management and CSRF protection. 
- routes/api.php file defines routes for api interface. These routes are stateless and assigned the api middleware group.

These route files automatically loaded by App\Providers\RouteServiceProvider.

Laravel namespace standard for web.php Route file:

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

- When defining routes for same URI with different HTTP method, then first definition will be taken with corresponding HTTP method. Avoid defining any, match and redirect methods before to it.

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

### &#10022; Named Routes:

### &#10022; Route Groups:

### &#10022; Controllers:

### &#10022; Middleware:

### &#10022; Route Prefixes:

### &#10022; Route Name Prefixes:

### &#10022; Subdomain Routing:

### &#10022; Model Binding:

### &#10022; Fallback Routes:

### &#10022; Request Rate Limiting:

### &#10022; Form Method Spoofing:

### &#10022; Current Route Information:

### &#10022; CORS:

### &#10022; Route Caching:

---
[&#8682; To Top](#-routing)

[&#10094; Previous Topic](./artisan-commands.md) &emsp; [Next Topic &#10095;](./middleware.md)

[&#8962; Goto Home Page](../README.md)