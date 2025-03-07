## &#10162; Sessions:

HTTP on applications might not save user data between requests.
Sessions allow storing user information and state across many requests.
User data is stored in a persistent store or backend.
This data can be accessed in later requests.
Laravel provides many session backends.
These backends are accessed through a simple, unified API.
Support for common backends like `Memcached`, `Redis`, and `databases` is included.

### &#9780; Overview:
1. [Configuration](#-configuration)
    - [Driver Prerequisites](#-driver-prerequisites)
2. [Retrieving Data](#-retrieving-data)
    - [Retrieving All Session Data](#-retrieving-all-session-data)
3. [Storing Data](#-storing-data)
    - [Pushing To Array Session Values](#-pushing-to-array-session-values)
    - [Retrieve and Delete An Item](#-retrieve-and-delete-an-item)
    - [Increment and Decrement Session Values](#-increment-and-decrement-session-values)
4. [Flash Data](#-flash-data)
    - [Reflash Data](#-reflash-data)
    - [Keep Data](#-keep-data)
5. [Deleting Data](#-deleting-data)
6. [Regenerating The Session ID](#-regenerating-the-session-id)
7. [Session Blocking](#-session-blocking)
8. [Adding Custom Session Drivers](#-adding-custom-session-drivers)
    - [Registering The Driver](#-registering-the-driver)

### &#10022; Configuration:

- The session configuration file is in `config/session.php`.
- Review the options in this file.
- Laravel uses the `file` session driver by default.
- This works well for many applications.
- For applications with multiple web servers, use a centralized session store with `Redis` or a `database`.
- The `session driver` option sets, where session data is stored.
- Laravel includes these drivers:
    - `file`: sessions are stored in `storage/framework/sessions`.
    - `cookie`: sessions are stored in secure, encrypted cookies.
    - `database`: sessions are stored in a relational database.
    - `memcached` / `redis`: sessions are stored in fast, cache-based stores.
    - `dynamodb`: sessions are stored in AWS DynamoDB.
    - `array`: sessions are stored in a PHP array and are not saved.
- The `array` driver is mainly for testing.
- It stops session data from being saved.

### &#10022; Driver Prerequisites:

**Database:**

- When using the `database` session driver, a user needs to create a table for session records.
- Here is an example schema for the table:

```php
Schema::create('sessions', function ($table) {
    $table->string('id')->primary();
    $table->foreignId('user_id')->nullable()->index();
    $table->string('ip_address', 45)->nullable();
    $table->text('user_agent')->nullable();
    $table->text('payload');
    $table->integer('last_activity')->index();
});
```

- A user can use the `session:table` Artisan command to create this migration.
- To learn about database migrations, review the [migrations](./migrations.md) notes.

```bash
php artisan session:table

php artisan migrate
```

**Redis:**

- Before using `Redis` sessions, a user needs to install the `PhpRedis` PHP extension via `PECL` or the `predis/predis` package (~1.0) via Composer.
- For `Redis` configuration, review Laravel Redis documentation.
- In the session configuration file, the `connection` option specifies the `Redis` connection for sessions.

### &#10022; Retrieving Data:

- Laravel has two main ways to use session data: the global `session` helper and a `Request` instance.
- A user can access the session via a `Request` instance in a route closure or controller method.
- Controller method dependencies are automatically injected by the Laravel service container.

```php
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     *
     * @param  Request  $request
     * @param  int  $id
     * @return Response
     */
    public function show(Request $request, $id)
    {
        $value = $request->session()->get('key');
        // ...
    }
}
```

- A user can provide a default value as the second argument to the `get` method.
- This default value is returned if the key does not exist.
- If a user provides a closure as the default value to `get` method, and if the key is missing, the closure is executed and returned a result if specified.

```php
$value = $request->session()->get('key', 'default');

$value = $request->session()->get('key', function () {
    // ...
    return $default;
});
```

**The Global Session Helper:**

- A user can use the global Laravel `session` function to get and store session data.
- When `session` is called with a string, it returns the value of that session key.
- When `session` is called with an array, those key/value pairs are stored in the session.

```php
Route::get('/home', function () {
    // Retrieve a piece of data from the session...
    $value = session('key');

    // Specifying a default value...
    $value = session('key', 'default');

    // Store a piece of data in the session...
    session(['key' => 'value']);
});
```

- Using `session` via a `Request` instance or the global `session` helper has little difference.
- Both methods are testable via the `assertSessionHas` method in test cases.

### &#10022; Retrieving All Session Data:

- A user can get all session data using the `all` method:

```php
$data = $request->session()->all();
```

**Determining If An Item Exists In The Session:**

- A user can check if an item exists using the `has` method.
- `has` returns true if the item exists and not null otherwise, returns false:

```php
if ($request->session()->has('users')) {
    //
}
```

- To check if an item exists, even if it's null, use the `exists` method:

```php
if ($request->session()->exists('users')) {
    //
}
```

- To check if an item is missing, use the `missing` method.
- `missing` returns true if the item is null or does not exist:

```php
if ($request->session()->missing('users')) {
    //
}
```

### &#10022; Storing Data:

- To store data in the session, use the request instance's `put` method or the global `session` helper function.

```php
// Via a request instance...
$request->session()->put('key', 'value');

// Via the global "session" helper...
session(['key' => 'value']);
```

### &#10022; Pushing To Array Session Values:

- The `push` method adds a new value to a session value that is an array.
- For example, if `user.teams` has an array of team names, a user can add a new value:

```php
$request->session()->push('user.teams', 'developers');
```

### &#10022; Retrieve and Delete An Item:

- The `pull` method gets and deletes an item from the session at once:

```php
$value = $request->session()->pull('key', 'default');
```

### &#10022; Increment and Decrement Session Values:

- If session data is an integer, a user can increment or decrement it using the `increment` and `decrement` methods.

```php
$request->session()->increment('count');

$request->session()->increment('count', $incrementBy = 2);

$request->session()->decrement('count');

$request->session()->decrement('count', $decrementBy = 2);
```

### &#10022; Flash Data:

- Sometimes, a user may want to store items in the session for the next request.
- A user can use the `flash` method.
- Data stored with `flash` is available immediately and in the next HTTP request.
- After the next request, the flashed data is deleted.
- Flash data is useful for short status messages:

```php
$request->session()->flash('status', 'Task was successful!');
```

### &#10022; Reflash Data:

- To keep flash data for more requests, use the `reflash` method.

```php
$request->session()->reflash();
```

### &#10022; Keep Data:

- To keep only specific flash data, use the `keep` method.

```php
$request->session()->keep(['username', 'email']);
```

- To keep flash data only for the current request, use the `now` method:

```php
$request->session()->now('status', 'Task was successful!');
```

### &#10022; Deleting Data:

- The `forget` method removes data from the session.
- To remove all session data, use the `flush` method.

```php
// Forget a single key...
$request->session()->forget('name');

// Forget multiple keys...
$request->session()->forget(['name', 'status']);

$request->session()->flush();
```

### &#10022; Regenerating The Session ID:

- Regenerating the session ID helps prevent session fixation attacks.
- Laravel automatically regenerates the session ID during authentication with starter kits or Fortify.
- To manually regenerate the session ID, use the `regenerate` method.

```php
$request->session()->regenerate();
```

- To regenerate the session ID and remove all session data, use the `invalidate` method.

```php
$request->session()->invalidate();
```

### &#10022; Session Blocking:

- To use session blocking, an application needs a cache driver with atomic locks.
- These drivers include `memcached`, `dynamodb`, `redis`, and `database`.
- The `cookie` session driver cannot be used.
- By default, Laravel allows concurrent requests with the same session.
- If JavaScript makes two HTTP requests, they run at the same time.
- This can cause session data loss in some applications with concurrent requests to different endpoints that write to the session.
- Laravel lets a user limit concurrent requests for a session.
- A user can chain the `block` method onto a route definition.
- In this example, a request to `/profile` gets a session lock.
- While locked, requests to `/profile` or `/order` with the same session ID wait for the first request to finish.

```php
Route::post('/profile', function () {
    //
})->block($lockSeconds = 10, $waitSeconds = 10)

Route::post('/order', function () {
    //
})->block($lockSeconds = 10, $waitSeconds = 10)
```

- The `block` method takes two optional arguments.
- The first argument is the maximum seconds to hold the session lock.
- If the request finishes earlier, the lock is released.
- The second argument is the seconds a request waits to get a session lock.
- An `Illuminate\Contracts\Cache\LockTimeoutException` is thrown if the request cannot get a lock in time.
- If no arguments are given, the lock is held for 10 seconds, and requests wait for 10 seconds:

```php
Route::post('/profile', function () {
    //
})->block()
```

### &#10022; Adding Custom Session Drivers:

- If existing session drivers do not fit to the application needs, Laravel allows writing a custom session handler.
- The custom session driver should implement PHP `SessionHandlerInterface`.
- This interface has a few methods.
- Consider, `MongoDB` implementation with PHP `SessionHandlerInterface`, which consists of open, close, read, write, destroy, gc methods to be implemented.

### &#10022; Registering The Driver:

- After implementing the driver, register it with Laravel.
- Use the `extend` method of the `Session` facade to add drivers.
- Call `extend` from the `boot` method of a service provider.
- Use `App\Providers\AppServiceProvider` or create a new provider:

```php
<?php

namespace App\Providers;

use App\Extensions\MongoSessionHandler;
use Illuminate\Support\Facades\Session;
use Illuminate\Support\ServiceProvider;

class SessionServiceProvider extends ServiceProvider
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
        Session::extend('mongo', function ($app) {
            // Return an implementation of SessionHandlerInterface...
            return new MongoSessionHandler;
        });
    }
}
```

- After registering the driver, use the `mongo` driver in `config/session.php`.

---
[&#8682; To Top](#-sessions)

[&#10094; Previous Topic](./url-generation.md) &emsp; [Next Topic &#10095;](./validation.md)

[&#8962; Goto Home Page](../README.md)