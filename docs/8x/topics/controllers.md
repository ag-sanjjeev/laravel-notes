## &#10162; Controllers:

Controllers are classes, which used to implement application logics asper incoming requests. Controllers are stored in the `app/Http/Controllers` directory in the application. 

After creating and implementing application logic to the controller, then it need to be assigned for appropriate routes requests. Otherwise, it will not be used.

### &#9780; Overview:
1. [Creating Controllers](#-creating-controllers)
2. [Single Action Controllers](#-single-action-controllers)
3. [Controller Middleware](#-controller-middleware)
4. [Resource Controllers](#-resource-controllers)
    - [Specifying Resource Model](#-specifying-resource-model)
    - [Generating Controller With Form Requests](#-generating-controller-with-form-requests)
    - [Missing Model Binding in Resource Routes](#-missing-model-binding-in-resource-routes)
    - [Partial Resource Routes](#-partial-resource-routes)
    - [Nested Resources](#-nested-resources)
    - [Scope Nested Resources](#-scope-nested-resources)
    - [Shallow Nested Resources](#-shallow-nested-resources)
    . [Naming Resource Routes](#-)
    . [Naming Resource Route Parameters](#-)
    . [Scoping Resource Routes](#-)
    . [Localizing Resource URIs](#-)
    . [Supplementing Resource Controllers](#-)
. [API Resource Controllers](#-api-resource-controllers)
. [Dependency Injection & Controllers](#-)


### &#10022; Creating Controllers:

Laravel controller created by extending the base controller class `App\Http\Controllers\Controller`. It is not required to extend a base controller class. But, it will not have access to the features like middleware and authorize methods.

```php
namespace App\Http\Controllers;
 
use App\Http\Controllers\Controller;
use App\Models\User;
 
class UserController extends Controller
{
    /**
     * Show the profile for a given user.
     *
     * @param  int  $id
     * @return \Illuminate\View\View
     */
    public function show($id)
    {
        return view('user.profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

Define route for the created controller.

In `web.php` file:

The below route definition will invoke `show` method of `UserController` class when request URI matches `/user/{id}` like `/user/123`.

```php
use App\Http\Controllers\UserController;
 
Route::get('/user/{id}', [UserController::class, 'show'])->where('id', '[0-9]+');
```

### &#10022; Single Action Controllers:

Single Action Controllers are also known as invokable controller.

If specific application logic is complex to handle by a single action method, then it is possible to dedicate an entire controller for it.

**Artisan Command:**

To generate using Artisan command with --invokable option as below:

```bash
php artisan make:controller ProvisionServer --invokable
```

**Manual Procedure:**

This can be accomplished by define a single `__invoke` method within the controller.

```php
namespace App\Http\Controllers;
 
use App\Http\Controllers\Controller;
use App\Models\User;
 
class ApplicationSetup extends Controller
{
    /**
     * Setting up this application based on the requests
     *
     * @return \Illuminate\Http\Response
     */
    public function __invoke()
    {
        // ...
    }
}
```

In `web.php` file:

When registering routes for single action controllers or invokable controller, then it does not required to mention the method name next to the controller. 

```php
use App\Http\Controllers\ApplicationSetup;
 
Route::post('/setup/finalize', ApplicationSetup::class);
```

### &#10022; Controller Middleware:

Middleware can be assigned to the routes before controller method get into action.

In `web.php` file:

```php
Route::get('/post', [PostController::class, 'index'])->middleware('auth');
```

Middleware can be assigned to the controller with in `construct` method. It can be controlled and assigned for all action controller method, only specific action controller method and all action controller method except specific method is possible. 

```php
use Illuminate\Http\Request;
use Illuminate\Http\Response;
use App\Models\Post;

class PostController extends Controller
{
    /**
     * Instantiate a new controller instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware('auth'); // applicable to all action methods
        $this->middleware('log')->only('index'); // applicable to index method
        $this->middleware('subscribed')->except('store'); // applicable to all method except store method
    }

    // ...

    public function index(Request $request, Response $response): mixed
    {
        // ...
        return view('post.article', ['title' => $post->title, 'content' => $post->content]);
    }
}
```

**Register Middleware Inside Controller:**

Controllers can provide possibilities to register middlewares using a closure. This allows to define inline middleware for a single controller without define new middleware class.

```php
$this->middleware(function ($request, $next) {
    // ...
    // middleware logic
    // ...
    return $next($request); // deliver request to next closure
});
```

### &#10022; Resource Controllers:

Consider, when an Eloquent model in application as a "resource", then it need to perform same set of actions such as create, read, update or delete (CRUD).

Laravel can create and handle controller with methods for handling resource actions (CRUD).

**Artisan Command:**

To generate resource controller with resource methods using --resource option to the Artisan command as below:

```bash
php artisan make:controller ProductController --resource
```

This command will generate a controller at `app/Http/Controllers/ProductController.php` and contain methods for each of the available resource operations.

**Register Route For Resource Controller:**

After creating and implementing logic to the resource action methods then it need to assign routes to it.

This single route declaration can handle all resource requests to appropriate controller methods. 

In `web.php` file:

```php
use App\Http\Controllers\ProductController;
 
Route::resource('product', ProductController::class);
```


**Register Routes For Multiple Resource Controllers:**

In `web.php` file:

```php
Route::resources([
    'product' => PhotoController::class,
    'post' => PostController::class,
]);
```

**Actions Handled By Resource Controller:**

| Request Method | URI | Controller Method |  Route Name | Remark |
|---|---|---|---|---|
|GET | /product | index | product.index | List Page |
| GET | /product/create | create | product.create | Show Create Form |
| POST | /product | store | product.store | Store Form Data |
| GET | /product/{id} | show | product.show | Show Data |
| GET | /product/{id}/edit | edit | product.edit | Show Edit Form |
| PUT/PATCH | /product/{id} | update | product.update | Update Form Data |
| DELETE | /product/{id} | destroy | product.destroy | Delete Data |

### &#10022; Specifying Resource Model:

When it is required to use route model binding with the resource controller's methods and to type-hint a model instance. Then controller can be generated with `--model` option.

```bash
php artisan make:controller PostController --model=Post --resource
```

### &#10022; Generating Controller With Form Requests:

To generate form request classes for the controller storage and update methods with `--requests` option. 

```bash
php artisan make:controller PostController --model=Post --resource --requests
```

**Missing Model Binding in Resource Routes:**

In `web.php` file:

```php
use App\Http\Controllers\PostController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Redirect;
 
Route::resource('post', PostController::class)
        ->missing(function (Request $request) {
            return Redirect::route('post.index');
        });
```

Sometimes, the specified route parameter does not match with model's table, then it throws 404 page not found error. To avoid this error then `missing` method can be used to handle.

To handle missing model data when model binding in resource route definition refer [Missing Model Behavior](./routing.md#-missing-model-behavior)

### &#10022; Partial Resource Routes:

Sometimes, it required to use few of the resource method. It can be defined in the route definition for resource routes with methods `only` or `except` as below:

In `web.php` file:

```php
use App\Http\Controllers\PostController;
use App\Http\Controllers\ProductController;
 
Route::resource('report/post', PostController::class)->only([
    'index', 'show'
]); // only allows index and show request instead of all 
 
Route::resource('report/product', PhotoController::class)->except([
    'create', 'store', 'update', 'destroy'
]); // allows all request except create, store, update and destroy 
```

### &#10022; Nested Resources:

Nested resources can be possible for example scenarios such as a post resource may have multiple comments to it. To nest these resource controllers, then use `.` dot notation in the route definition.

When define nested resources then will be treats as below for all resource methods:

`/post/{post}/comment/{comment}` 

In `web.php` file:

```php
use App\Http\Controllers\PostCommentController;
 
Route::resource('post.comment', PostCommentController::class);
```

### &#10022; Scope Nested Resources:

Laravel automatically scope nested resource. But it can be possible to scope custom nested resource with `scopeBindings` method in Route class when implicit model binding. Refer [Key Scope in Routings](./routing.md#-key-scoping).

### &#10022; Shallow Nested Resources:

When it is not necessary to child id when parent id is auto increment and primary key. Then it is not necessary to mention child's id in the request URI. This can be defined with `shallow` method as below:

In `web.php` file:

```php
use App\Http\Controllers\CommentController;
 
Route::resource('post.comment', CommentController::class)->shallow();
```

This above route definition will have the following routes:

| Request Method | URI | Controller Method | Route Name | Remark |
|---|---|---|---|---|
|GET | /post/{post}/comments | index | post.comments.index | List Page |
|GET | /post/{post}/comments/create | create | post.comments.create | Show Create Form |
|POST | /post/{post}/comments | store | post.comments.store | Store Form Data |
|GET | /comments/{comment} | show | comments.show | Show Data |
|GET | /comments/{comment}/edit | edit | comments.edit | Show Edit Form |
|PUT/PATCH | /comments/{comment} | update | comments.update | Update Form Data |
|DELETE | /comments/{comment} | destroy | comments.destroy | Delete Data |

### &#10022; Naming Resource Routes:

In Laravel, all resource controller actions have corresponding route name. But it is possible to override existing names by passing a names array in `names` method as below:

In `web.php` file:

Hence, as per below example, route name `post.create` will be changed into `post.draft` for the resource routes.

```php
use App\Http\Controllers\PostController;
 
Route::resource('post', PostController::class)->names([
    'create' => 'post.draft'
]);
```

### &#10022; Naming Resource Route Parameters:

### &#10022; Scoping Resource Routes:

### &#10022; Localizing Resource URIs:

### &#10022; Supplementing Resource Controllers:

### &#10022; API Resource Controllers:

When declaring resource routes for API then it does not required routes that present HTML templates such as create and edit.

**Method 1:**

It can be done by using `apiResource` method to exclude `create` and `edit` methods.

In `web.php` file:

```php
use App\Http\Controllers\PostController;
 
Route::apiResource('post', PostController::class);
```

**Register multiple API resource for different routes:**

```php
use App\Http\Controllers\ProductController;
use App\Http\Controllers\PostController;
 
Route::apiResources([
    'product' => ProductController::class,
    'post' => PostController::class,
]);
```

**Method 2:**

To generate controller for API resource by using `--api` option in Artisan command, which does not included `create` and `edit` methods with in it.

```bash
php artisan make:controller PostController --api
```

### &#10022; Dependency Injection & Controllers:


---
[&#8682; To Top](#-controllers)

[&#10094; Previous Topic](./middleware.md) &emsp; [Next Topic &#10095;](./requests.md)

[&#8962; Goto Home Page](../README.md)