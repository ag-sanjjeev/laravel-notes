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
    - [Naming Resource Routes](#-naming-resource-routes)
    - [Naming Resource Route Parameters](#-naming-resource-route-parameters)
    - [Scoping Resource Routes](#-scoping-resource-routes)
    - [Localizing Resource URIs](#-localizing-resource-uris)
    - [Supplementing Resource Controllers](#-supplementing-resource-controllers)
5. [Dependency Injection](#-dependency-injection)
6. [API Resource Controllers](#-api-resource-controllers)


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

### &#10022; Missing Model Binding in Resource Routes:

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
|GET | /post/{post}/comment | index | post.comment.index | List Page |
|GET | /post/{post}/comment/create | create | post.comment.create | Show Create Form |
|POST | /post/{post}/comment | store | post.comment.store | Store Form Data |
|GET | /comment/{comment} | show | comment.show | Show Data |
|GET | /comment/{comment}/edit | edit | comment.edit | Show Edit Form |
|PUT/PATCH | /comment/{comment} | update | comment.update | Update Form Data |
|DELETE | /comment/{comment} | destroy | comment.destroy | Delete Data |

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

Route::resource will create the route parameters as per the table above mentioned. This route parameters can be override with `parameters` method by mention alternative names for parameters as an array.

In `web.php` file:

Resource routes generate URI for the resource as `/user/{user}` for show route. To override `{user}` with `{admin_user}` then final URI is `/users/{admin_user}` and it is possible with below approach:

```php
use App\Http\Controllers\AdminUserController;
 
Route::resource('user', AdminUserController::class)->parameters([
    'user' => 'admin_user'
]);
```

### &#10022; Scoping Resource Routes:

Laravel resolves nested model binding and confirms that child model is belongs to the parent model. It can be scoped by using `scoped` method when define nested resource. It is possible to enable automatic scope as well as required field of the child resource should be retrieved.

In `web.php` file:

By default, URI assigned as `/post/{post}/comment/{comment}` and to register route as URI `/post/{post}/comment/{comment:comment_id}`.

```php
use App\Http\Controllers\PostCommentController;
 
Route::resource('post.comment', PhotoCommentController::class)->scoped([
    'comment' => 'comment_id',
]);
```

Refer [Custom Column Key in Routing](./routing.md#-custom-column-key) for more.

### &#10022; Localizing Resource URIs:

Laravel uses English verbs to describe the route action methods. If it need to localize those verbs then it can be modified according to the localization. It is possible with `resourceVerbs` method of `Route` class at the beginning of the `boot` method within `App\Providers\RouteServiceProvider` class.

For an example, To customize those verbs corresponding in Tamil Language. `uruvakku` is same to `create` and `thiruthu` is same to `edit`. Once those verbs defined, then resource route registration need to be done.

In `web.php` file:

```php
Route::resource('porul', ProductController::class);
```

The above route definition will produce URI as below:

For create - `/porul/uruvakku`
For edit - `/porul/{porul}/thiruthu`

In `App\Providers\RouteServiceProvider` class:

```php
/**
 * Define your route model bindings, pattern filters, etc.
 *
 * @return void
 */
public function boot()
{
    Route::resourceVerbs([
        'create' => 'uruvakku', // Tamil language corresponding 
        'edit' => 'thiruthu' // Tamil language corresponding
    ]);
 
    // ...
}
```

### &#10022; Supplementing Resource Controllers:

If it requires to handle additional set of routes to the same resource controller other than default set of resource action methods. It is possible to define those supplement routes before any resource route definition. Otherwise, it will proceed with resource route action methods and might through error.

**Note:** 

- If it need to additional methods other than the resource controller action methods, then it is recommend to split the controller. And keep the controller logic focused. 

In `web.php` file:

```php
use App\Http\Controller\PostController;
 
Route::get('/post/popular', [PostController::class, 'popular']);
Route::resource('post', PostController::class);
```

### &#10022; Dependency Injection:

**Constructor Injection:**

Laravel service container will resolve all Laravel controllers dependencies when it is type-hinted. Those declared dependencies will be resolved and injected into the controller instance.

```php
namespace App\Http\Controllers;
 
use App\Models\Product;
 
class ProductController extends Controller
{
    /**
     * The product model instance.
     * 
     * @var Product|null
     */
    protected ?Product $product = null;
 
    /**
     * Create a new controller instance.
     *
     * @param  \App\Models\Product $product
     * @return void
     */
    public function __construct(Product $product): void
    {
        $this->product = $product;
    }
}
```

**Method Injection:**

Similarly, Dependencies can be resolved and injected to the controller's method arguments. Here, `Request` and `Response` dependencies will be injected into the method when invoked.

```php
namespace App\Http\Controllers;
 
use Illuminate\Http\Request;
use Illuminate\Http\Response;
 
class ProductController extends Controller
{
    /**
     * Store a new product.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request): Response
    {
        $name = $request->name;
        // ... 
    }
}
```

**Inject Route Parameter:**

In `web.php` file:

```php
use App\Http\Controllers\ProductController;
 
Route::get('/product/{id}', [ProductController::class, 'show'])->where('id', '[0-9]+');
```

It is still possible by type-hint `Illuminate\Http\Request` class instance and access product id parameter.

```php
namespace App\Http\Controllers;
 
use Illuminate\Http\Request;
 
class ProductController extends Controller
{
    /**
     * Show the given product.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  int $id
     * @return \Illuminate\Http\Response
     */
    public function show(Request $request, int $id): Response
    {
        $id = $request->id; // can be accessed the route parameter
        $id = $id; // can be accessed the route parameter
        // ...
    }
}
```

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

---
[&#8682; To Top](#-controllers)

[&#10094; Previous Topic](./middleware.md) &emsp; [Next Topic &#10095;](./requests.md)

[&#8962; Goto Home Page](../README.md)