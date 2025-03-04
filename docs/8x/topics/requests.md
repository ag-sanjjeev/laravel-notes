## &#10162; Requests:

`Illuminate\Http\Request` is a class that used for access HTTP request in Laravel. Which is used to access the request input, files upload and cookies that were submitted with the request.

### &#9780; Overview:
1. [Access Request](#-access-request)
    - [Request Path](#-request-path)
    - [Request Method](#-request-method)
    - [Request URL](#-request-url)
    - [Request Headers](#-request-headers)
    - [Request IP Address](#-request-ip-address)
    - [Content Negotiation](#-content-negotiation)
    - [PSR-7 Requests](#-psr-7-requests)
2. [Retrieve Input](#-retrieve-input)
    - [Retrieve All Input](#-retrieve-all-input)
    - [Retrieve From Query String](#-retrieve-from-query-string)
    - [Retrieving JSON Input](#-retrieve-json-input)
    - [Retrieving Boolean Input](#-retrieve-boolean-input)
    - [Retrieving Date Input](#-retrieve-date-input)
    - [Retrieving Input Using Dynamic Properties](#-retrieve-input-using-dynamic-properties)
    - [Retrieving Portion Of Input:](#-retrieve-portion-of-input)
    - [Check Input Exist](#-check-input-exist)
    - [Invoke Closure When Input Exist](#-invoke-closure-when-input-exist)
    - [Check Input Filled](#-check-input-filled)
    - [Invoke Closure When Input Filled](#-invoke-closure-when-input-filled)
    - [Check Missing Input](#-check-missing-input)
    - [Merging Additional Input](#-merging-additional-input)
    - [Old Input](#-old-input)
    - [Flashing Input To The Session](#-flashing-input-to-the-session)
    - [Redirect With Flash Input](#-redirect-with-flash-input)
    - [Retrieving Old Input](#-retrieving-old-input)
    . [Cookies](#-)
    . [Input Trimming & Normalization](#-)
. [Files](#-)
    . [Retrieving Uploaded Files](#-)
    . [Storing Uploaded Files](#-)
. [Configuring Trusted Proxies](#-)
. [Configuring Trusted Hosts](#-)

### &#10022; Access Request:

To access Request class to get information of current HTTP request via dependency injection.
To access before it need to be type-hinted `Illuminate\Http\Request` class on route closure or controller method. 

Refer [dependency injection](./controllers.md#-dependency-injection) in controllers.

Refer [dependency injection](./routing.md#-dependency-injection) in route definitions.

### &#10022; Request Path:

This `Request` class instance provides various methods to get information about incoming HTTP request. Which extends the Symfony\Component\HttpFoundation\Request class.

**Request Path:**

```php
$uri = $request->path();
```

This will retrieve URL path from `http://example.host/path/level` as `path/level`.

**Check Request Path:**

The `is` method from `Request` class checks the current request as specifies. Where `*` in `admin/*` is a wildcard to match URL path start with the characters and ignore to check after it.

```php
if ($request->is('admin/*')) {
    //
}
```

**Check Request Route:**

The `routeIs` method from `Request` class checks name of the current HTTP request which is defined in routes definitions.

```php
if ($request->routeIs('admin.*')) { // checks route name starts with `admin.`
    //
}
```

### &#10022; Request Method:

The `method` method from `Request` class will return incoming HTTP request method.

```php
$method = $request->method();
```

**Check Request Method:**

```php
if ($request->isMethod('post')) { // check whether incoming request is post method
    //
}
```

### &#10022; Request URL:

To get the full URL of the incoming request, then use the `url` or `fullUrl` methods from `Request` class. The `url` method will return the URL without the query string and the `fullUrl` method will return the complete URL including the query string.

```php
$url = $request->url();
```

```php
$fullUrl = $request->fullUrl();
```

**Append Query String:**

If it requires to append query string to the full URL, then use `fullUrlWithQuery` method from `Request` class. This method accepts array of query string variables to append with the current query string.

```php
// http://example.host/path/level?referrer=123
$request->fullUrlWithQuery(['lang' => 'tamil']); // http://example.host/path/level?referrer=123&lang=tamil
```

### &#10022; Request Headers:

The `header` method from `Request` class will return header information of the incoming request. If the header is not present on incoming request then it return `null`. 

```php
$value = $request->header('X-Header-Name');
```

It accepts optional parameter to use missing header value instead of `null`.

```php
$value = $request->header('X-Header-Name', 'default');
```

To retrieve bearer token from the Authorization header of incoming request then use `bearerToken` method. If it is not present then it return an empty string.

```php
$token = $request->bearerToken();
```

**Check Header Present:**

The `hasHeader` method is used to check specified header presents or not. 

```php
if ($request->hasHeader('X-Header-Name')) {
    //
}
```

### &#10022; Request IP Address:

The `ip` method used to retrieve the IP address of the client that made the request to the application.

```php
$ipAddress = $request->ip();
```

### &#10022; Content Negotiation:

Laravel provides the `getAcceptableContentTypes` method to get an array of all of the content types accepted by the request via the Accept header present in the request.

```php
$contentTypes = $request->getAcceptableContentTypes();
```

To check specified content type is acceptable or not by the request using the `accepts` method.

```php
if ($request->accepts(['text/html', 'application/json'])) {
   // ...
}
```

**Check Preferred Content Type:**

The `prefers` method is used to determine, which content type is most preferred by the request out of all content types as specified as an array. If nothing is acceptable content type then it returns `null`.

```php
$preferred = $request->prefers(['text/html', 'application/json']);
```

**Check JSON is acceptable content type:**

```php
if ($request->expectsJson()) {
    // ...
}
```

### &#10022; PSR-7 Requests:

The PSR-7 standard specifies interfaces for HTTP messages, HTTP requests and responses. But, Laravel uses `Symfony` HTTP Message Bridge component to convert Laravel requests and responses into PSR-7 compatible.

If it requires to use PSR-7 request instead of a Laravel request then it need below listed libraries.

```bash
composer require symfony/psr-http-message-bridge
```

```bash
composer require nyholm/psr7
```

**Usage:**

In `web.php` file:

```php
use Psr\Http\Message\ServerRequestInterface;
 
Route::get('/', function (ServerRequestInterface $request) {
    //
});
```

If the response from a route or controller is PSR-7 response instance, then it will be converted back into Laravel response instance and be displayed to the user.

### &#10022; Retrieving Input:

To access all input request data from user request with `Illuminate\Http\Request` class instance independent of HTTP request method. 

**Retrieve Specific Key Input:**

```php
$name = $request->input('name');
```

**Set Default Value If Input Missing:**

Optional, default value may be passed to the second argument. 

```php
$name = $request->input('name', 'Kumar');
```

**Access Array of Input Data:**

Use `.` dot symbol to create a key to access the value from array of input. 

```php
$name = $request->input('products.0.name');
```
 
Retrieve all data from array of inputs with `*` symbol as wildcard.

```php
$names = $request->input('products.*.name');
```

To retrieve all of the input data as an associative array when it is not specified.

```php
$input = $request->input();
```

### &#10022; Retrieve All Input:

Retrieve all of the input data from incoming request as an array using the `all` method. This method can be used to retrieve from HTML form or XHR request independently.

```php
$input = $request->all();
```

To retrieve all of the input data from incoming request as a collection and later it can be iterate with `each` method of `Request` class.

```php
$input = $request->collect();
```

It can retrieve a subset of the input data from the incoming request as a collection.

```php
$request->collect('users')->each(function ($user) {
    // ...
});
```

### &#10022; Retrieve From Query String:

The `query` method will retrieve values from the query string alone. Unlike the `input` method retrieves input data from the incoming request including the query string.

```php
$name = $request->query('name');
```

**Set Default Value If Query String Missing:**

Optional, default value may be passed to the second argument. 

```php
$name = $request->query('name', 'Velan');
```

**Retrieve All Query String:**

```php
$query = $request->query();
```

### &#10022; Retrieving JSON Input:

JSON data can be accessed when incoming request sends JSON data, By `input` method when the Content-Type header of the request is set to `application/json`. To access JSON input value by using `.` dot symbol with nested level within JSON arrays.

```php
$name = $request->input('user.name');
```

### &#10022; Retrieving Boolean Input:

When HTML form elements like checkboxes and radios submitted with type of `Boolean`, then it can be retrieved such a inputs with `boolean` method specified with key name.

The values such as 1, true, "true", "on", "yes" are treated and as boolean `true`. And all other values will be treated as boolean `false`.

```php
$acceptTerms = $request->boolean('terms');
```

### &#10022; Retrieving Date Input:

If the form submitted input values with dates and times, then it can be retrieved as Carbon instances using the `date` method of `Request` class. If the date input is not presents then it will return `null`.

```php
$publishDate = $request->date('publishDate');
```

**Manipulate Date Input:**

The `date` method may accept optional arguments to specify format as second argument and timezone as third argument.

```php
$publishDate = $request->date('publishDate', '!H:i', 'Asia/Kolkata');
```

**Note:**

If the input value is present and an invalid format, then an `InvalidArgumentException` will be thrown. So, it is important to validate before using the date method.

### &#10022; Retrieving Input Using Dynamic Properties:

To access user input via dynamic properties from the `Illuminate\Http\Request` class instance. 

To retrieve input with key `name` as below:

```php
$name = $request->name;
```

When using dynamic properties, Laravel will search in parameters value from the request and then matched route parameters.

### &#10022; Retrieving Portion Of Input:

To access or retrieve specified portion of input from incoming request, then it can be done with the `only` and `except` methods. Both methods accept a single array or a dynamic list of arguments.

```php
$input = $request->only(['username', 'password']);

$input = $request->only('username', 'password');
```

```php 
$input = $request->except(['_method']);
 
$input = $request->except('_method');
```

The only method returns all of the value specified and if it is not present then it will not return.

### &#10022; Check Input Exist:

The `has` method is used to check whether input key is exist or not.

```php
if ($request->has('username')) {
    //
}
```

Check all specified key inputs are exist or not.

```php
if ($request->has(['username', 'password'])) {
    //
}
```

Check any of the specified key input is exist or not.

```php
if ($request->hasAny(['username', 'email'])) {
    //
}
```

### &#10022; Invoke Closure When Input Exist:

The `whenHas` method will invoke closure specified in the second argument, when input key exist on the request.

```php
$request->whenHas('username', function ($input) {
    //
});
```

The `whenHas` method may accept optional second closure to deal with when input key is not exist scenario.

```php
$request->whenHas('username', function ($input) {
    // The "username" value exist...
}, function () {
    // The "username" value is not exist...
});
```

### &#10022; Check Input Filled:

Check the input key is exist and is not empty as filled with some value.

```php
if ($request->filled('name')) {
    //
}
```

### &#10022; Invoke Closure When Input Filled:

The `whenFilled` method will invoke closure specified in the second argument, when input key exist on the request and it is not empty.

```php
$request->whenFilled('username', function ($input) {
  //
});
```

The `whenFilled` method may accept optional second closure to deal with when input key is exist and not filled scenario.

```php
$request->whenFilled('username', function ($input) {
    // The "username" value is filled...
}, function () {
    // The "username" value is not filled...
});
```

### &#10022; Check Missing Input:

Check input key is missing from the request using `missing` method.

```php
if ($request->missing('username')) {
    //
}
```

### &#10022; Merging Additional Input:

Sometimes, it require to manually merge additional input to existing request input via `merge` method.

```php
$request->merge(['quantity' => 1]);
```

The `mergeIfMissing` method is used to merge additional input when corresponding input keys is not exist in the request input data.

```php
$request->mergeIfMissing(['quantity' => 1]);
```

### &#10022; Old Input:

Laravel preserve previous request input data until next request. This feature will be useful for repopulating form data after validation errors. This is possible by session flashing technique and it does not required to invoke manually when using validation features.

### &#10022; Flashing Input To The Session:

The `flash` method from the `Illuminate\Http\Request` class will stores the current input data to the session until next request. 

```php
$request->flash();
```

The `flashOnly` and `flashExcept` methods are useful to flash subset of the input data to the session. To ignore flash any sensitive information such as passwords, credit card details and so on.

```php
$request->flashOnly(['username', 'email']);
```

```php
$request->flashExcept('password');
```

### &#10022; Redirect With Flash Input:

Before redirect, it requires to flash current request input data as below:

```php
return redirect('product.create')->withInput();
```

```php
return redirect()->route('product.create')->withInput();
```
 
```php
return redirect('login')->withInput(
    $request->except('password')
);
```

### &#10022; Retrieving Old Input:

To retrieve flashed input data with the `old` method from `Illuminate\Http\Request` instance. 

```php
$username = $request->old('username');
```

To repopulate form fields with old data using Laravel `old` helper function.

```php
<input type="text" name="username" value="{{ old('username') }}">
```

### &#10022; Cookies:

### &#10022; Input Trimming & Normalization:

### &#10022; Files:

### &#10022; Retrieving Uploaded Files:

### &#10022; Storing Uploaded Files:

### &#10022; Configuring Trusted Proxies:

### &#10022; Configuring Trusted Hosts:



---
[&#8682; To Top](#-requests)

[&#10094; Previous Topic](./controllers.md) &emsp; [Next Topic &#10095;](./requests.md)

[&#8962; Goto Home Page](../README.md)