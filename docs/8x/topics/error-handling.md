## &#10162; Error Handling:

- Laravel projects come with pre-configured error and exception handling.
- The `App\Exceptions\Handler` class handles logging and rendering exceptions.
- This class is the central point for exception management in the application.


### &#9780; Overview:

1. [Configuration](#-configuration)
2. [Reporting Exceptions](#-reporting-exceptions)
    - [Custom Exception Reporting](#-custom-exception-reporting)
    - [Global Log Context](#-global-log-context)
    - [Exception Log Context](#-exception-log-context)
    - [Report Helper](#-report-helper)
    - [Ignoring Exceptions By Type](#-ignoring-exceptions-by-type)
    - [Rendering Exceptions](#-rendering-exceptions)
    - [Reportable and Renderable Exceptions](#-reportable-and-renderable-exceptions)
    - [Mapping Exceptions By Type](#-mapping-exceptions-by-type)
3. [HTTP Exceptions](#-http-exceptions)
    - [Custom HTTP Error Pages](#-custom-http-error-pages)

### &#10022; Configuration:

- The `debug` option in `config/app.php` controls error information displayed to users.
- It defaults to the `APP_DEBUG` environment variable from `.env`.
- Set `APP_DEBUG` to `true` during local development.
- Set `APP_DEBUG` to `false` in production to avoid exposing sensitive information.


### &#10022; Reporting Exceptions:

- Exceptions are handled by `App\Exceptions\Handler`.
- The `register` method registers custom exception reporting and rendering callbacks.
- Exception reporting logs exceptions or sends them to external services.
- By default, exceptions are logged based on logging configuration.

### &#10022; Custom Exception Reporting:

- Use the `reportable` method to register closures for specific exception types.
- Laravel deduces the exception type from the closure's type-hint.
- Example:

```php
use App\Exceptions\InvalidOrderException;

public function register()
{
  $this->reportable(function (InvalidOrderException $e) {
    // ...
  });
}
```

- Use `stop` or return `false` to prevent default logging.

```php
$this->reportable(function (InvalidOrderException $e) {
  // ...
})->stop();

$this->reportable(function (InvalidOrderException $e) {
  return false;
});
```

- Use reportable exceptions to customize exception reporting.

### &#10022; Global Log Context:

- Laravel adds the current user's ID to exception log messages.
- Override `context` in `App\Exceptions\Handler` to define global contextual data.
- Example:

```php
protected function context()
{
  return array_merge(parent::context(), [
    'key' => 'value',
  ]);
}
```

### &#10022; Exception Log Context:

- Define a `context` method on custom exceptions for exception-specific contextual data.
- Example:

```php
namespace App\Exceptions;

use Exception;

class InvalidOrderException extends Exception
{
  // ...

  public function context()
  {
    return ['order_id' => $this->orderId];
  }
}
```

### &#10022; Report Helper:

- Use the `report` helper to report exceptions without rendering an error page.
- Example:

```php
public function isValid($value)
{
  try {
    // Validate the value...
  } catch (Throwable $e) {
    report($e);

    return false;
  }
}
```

### &#10022; Ignoring Exceptions By Type:

- Some exceptions should be ignored and not reported.
- The `$dontReport` property in the exception handler (`App\Exceptions\Handler`) is used for this.
- Add exception classes to `$dontReport` to prevent reporting.
- Example:

```php
use App\Exceptions\InvalidOrderException;

/**
 * A list of the exception types that should not be reported.
 *
 * @var array
 */
protected $dontReport = [
  InvalidOrderException::class,
];
```

- Laravel already ignores certain errors (e.g., 404 and 419 exceptions).

### &#10022; Rendering Exceptions:

- Laravel's exception handler converts exceptions to HTTP responses by default.
- Use the `renderable` method to register custom rendering closures for specific exception types.
- The closure must return an `Illuminate\Http\Response` instance (use the `response` helper).
- Laravel determines the exception type from the closure's type-hint.
- Example:

```php
use App\Exceptions\InvalidOrderException;

public function register()
{
  $this->renderable(function (InvalidOrderException $e, $request) {
    return response()->view('errors.invalid-order', [], 500);
  });
}
```

- `renderable` can override rendering for built-in Laravel or Symfony exceptions (e.g., `NotFoundHttpException`).
- If the closure doesn't return a value, Laravel's default rendering is used.
- Example for API requests:

```php
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

public function register()
{
  $this->renderable(function (NotFoundHttpException $e, $request) {
    if ($request->is('api/*')) {
      return response()->json([
        'message' => 'Record not found.'
      ], 404);
    }
  });
}
```

### &#10022; Reportable and Renderable Exceptions:

- Define `report` and `render` methods directly on custom exceptions.
- Framework automatically calls these methods.
- Example:

```php
<?php

namespace App\Exceptions;

use Exception;

class InvalidOrderException extends Exception
{
  /**
   * Report the exception.
   *
   * @return bool|null
   */
  public function report()
  {
    // ...
  }

  /**
   * Render the exception into an HTTP response.
   *
   * @param \Illuminate\Http\Request $request
   * @return \Illuminate\Http\Response
   */
  public function render($request)
  {
    return response(...);
  }
}
```

- Return `false` from `render` to use the exception's default HTTP response.
- Example:

```php
/**
 * Render the exception into an HTTP response.
 *
 * @param \Illuminate\Http\Request $request
 * @return \Illuminate\Http\Response
 */
public function render($request)
{
  // Determine if the exception needs custom rendering...

  return false;
}
```

- Return `false` from `report` to use default exception handling.
- Example:

```php
/**
 * Report the exception.
 *
 * @return bool|null
 */
public function report()
{
  // Determine if the exception needs custom reporting...

  return false;
}
```

- Type-hint dependencies in `report` for automatic injection by the service container.

### &#10022; Mapping Exceptions By Type:

- Third-party libraries may throw exceptions that want to render, but it is not possible to modify them.
- Laravel allows mapping these exceptions to own exception types.
- Use the `map` method in the exception handler's `register` method.
- Example:

```php
use League\Flysystem\Exception;
use App\Exceptions\FilesystemException;

public function register()
{
  $this->map(Exception::class, FilesystemException::class);
}
```

- Pass a closure to `map` for more control over the target exception's creation.
- Example:

```php
use League\Flysystem\Exception;
use App\Exceptions\FilesystemException;

$this->map(fn (Exception $e) => new FilesystemException($e));
```

### &#10022; HTTP Exceptions:

- Some exceptions represent HTTP error codes (e.g., 404, 401, 500).
- Use the `abort` helper to generate such responses.
- Example: `abort(404);`

### &#10022; Custom HTTP Error Pages:

- Laravel simplifies displaying custom error pages for HTTP status codes.
- Create a view template in `resources/views/errors/{status_code}.blade.php` (e.g., `404.blade.php`).
- This view is rendered for all corresponding HTTP errors.
- The `HttpException` instance from `abort` is passed as `$exception`.
- Example:

```blade
<h2>{{ $exception->getMessage() }}</h2>
```

- Publish Laravel's default error page templates using `vendor:publish`.
- Customize published templates.
- Command: `php artisan vendor:publish --tag=laravel-errors`

---
[&#8682; To Top](#-error-handling)

[&#10094; Previous Topic](./validation.md) &emsp; [Next Topic &#10095;](./logging.md)

[&#8962; Goto Home Page](../README.md)