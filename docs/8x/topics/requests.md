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
    - [Retrieve JSON Input](#-retrieve-json-input)
    - [Retrieve Boolean Input](#-retrieve-boolean-input)
    - [Retrieve Date Input](#-retrieve-date-input)
    - [Retrieve Input Using Dynamic Properties](#-retrieve-input-using-dynamic-properties)
    - [Retrieve Portion Of Input:](#-retrieve-portion-of-input)
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
    - [Cookies](#-cookies)
    - [Input Trimming and Normalization](#-input-trimming-and-normalization)
3. [Retrieve Uploaded Files](#-retrieve-uploaded-files)
    - [Check File Is Uploaded](#-check-file-is-uploaded)
    - [Validate Uploaded Files](#-validate-uploaded-files)
    - [Uploaded Files Path and Extension](#-uploaded-files-path-and-extension)
    - [Store Uploaded Files](#-store-uploaded-files)
    - [Other Files Methods](#-other-files-methods)
4. [Configure Trusted Proxies](#-configure-trusted-proxies)
5. [Configure Trusted Hosts](#-configure-trusted-hosts)

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

### &#10022; Retrieve Input:

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

### &#10022; Retrieve JSON Input:

JSON data can be accessed when incoming request sends JSON data, By `input` method when the Content-Type header of the request is set to `application/json`. To access JSON input value by using `.` dot symbol with nested level within JSON arrays.

```php
$name = $request->input('user.name');
```

### &#10022; Retrieve Boolean Input:

When HTML form elements like checkboxes and radios submitted with type of `Boolean`, then it can be retrieved such a inputs with `boolean` method specified with key name.

The values such as 1, true, "true", "on", "yes" are treated and as boolean `true`. And all other values will be treated as boolean `false`.

```php
$acceptTerms = $request->boolean('terms');
```

### &#10022; Retrieve Date Input:

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

### &#10022; Retrieve Input Using Dynamic Properties:

To access user input via dynamic properties from the `Illuminate\Http\Request` class instance. 

To retrieve input with key `name` as below:

```php
$name = $request->name;
```

When using dynamic properties, Laravel will search in parameters value from the request and then matched route parameters.

### &#10022; Retrieve Portion Of Input:

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

- Laravel cookies are encrypted and signed, preventing client-side alteration.
- Altered cookies are deemed invalid.
- To get a cookie's value, use the `cookie` method on `Illuminate\Http\Request`.
- These cookies are retrieved from HTTP requests.

```php
$value = $request->cookie('name');
```

### &#10022; Input Trimming and Normalization:

- Laravel global middleware stack has `TrimStrings` and `ConvertEmptyStringsToNull`.
- These middleware, in `App\Http\Kernel`, automatically trim request strings and convert empty strings to null.
- This simplifies route and controller normalization.
- To disable, remove these two middleware from `$middleware` in `App\Http\Kernel`.
- This process involves input trimming and normalization of data.

### &#10022; Retrieve Uploaded Files:

- Uploaded files are retrieved via `file` method or dynamic properties from `Illuminate\Http\Request` class.
- `file` method returns `Illuminate\Http\UploadedFile` class, which is extending PHP `SplFileInfo`.
- Laravel `UploadedFile` class offers methods for file interaction.

*Example: Accessing photo upload*

```php
$file = $request->file('photo');
```

```php
$file = $request->photo;
```


### &#10022; Check File Is Uploaded:

- `hasFile` method checks if a file exists in the request. It's used to determine file presence.
- This method verifies file uploads.
- It helps in conditional file processing.

*Example:* 

```php
if ($request->hasFile('photo')) { 
    // 
}
```

### &#10022; Validate Uploaded Files:

- `isValid` method checks for file upload errors.
- It verifies successful file uploads.
- This method ensures file integrity.
- It helps in robust file handling.

*Example:* 

```php
if ($request->file('photo')->isValid()) { 
// 
}
```

### &#10022; Uploaded Files Path and Extension:

- `UploadedFile` class provides methods for uploaded file path and extension access.
- `path()` gets the uploaded file's full path.
- `extension()` guesses the extension based on file content.
- The guessed extension may differ from the client's supplied one.

*Example:* 

```php
$path = $request->photo->path();
```

```php
$extension = $request->photo->extension();
```

### &#10022; Store Uploaded Files:

- Uploaded files are stored using configured filesystems.
- `store` method from `UploadedFile` class, which moves uploaded files to a disk like local or cloud storage.
- `store` method takes a path relative to the disk's root, excluding a filename.
- A unique ID is generated as the filename.
- `store` method can accepts an optional disk name.
- `store` returns the file's path relative to the disk's root after successful store.

*Example:* 

Uploaded file `photo` will be stored inside `images/` directory of configured storage directory.

```php
$path = $request->photo->store('images');
```

Similarly, which stores uploaded photo file under `images/` directory under `s3` disk.

```php
$path = $request->photo->store('images', 's3');
```

**storeAs Method:**

- `storeAs` method allows specifying the filename.
- It accepts path, filename, and optional disk name.
- This avoids automatic unique filename generation.
- Refer to file storage documentation for details.

*Example:* 

```php
$path = $request->photo->storeAs('images', 'filename.jpg');
```

```php
$path = $request->photo->storeAs('images', 'filename.jpg', 's3');
```

### &#10022; Other Files Methods:

- `UploadedFile` has various other methods.
- API documentation provides details.
- These methods offer further file handling capabilities.

### &#10022; Configure Trusted Proxies:

- Applications behind TLS/SSL terminating load balancers may not generate HTTPS links.
- This occurs when traffic is forwarded on port 80.
- The application doesn't recognize the need for secure links.
- This issue arises with the `url` helper.
- Configuring trusted proxies addresses this problem.
- `TrustProxies` middleware solves HTTPS link issues behind load balancers.
- Trusted proxies are listed in the `$proxies` array.
- `$headers` configures trusted proxy headers.

*Example:*

This middleware customizes trusted load balancers or proxies.

```php
namespace App\Http\Middleware;
 
use Illuminate\Http\Middleware\TrustProxies as Middleware;
use Illuminate\Http\Request;
 
class TrustProxies extends Middleware
{
    /**
     * The trusted proxies for this application.
     *
     * @var string|array
     */
    protected $proxies = [
        '192.168.0.21',
        '192.168.0.22',
    ];
 
    /**
     * The headers that should be used to detect proxies.
     *
     * @var int
     */
    protected $headers = Request::HEADER_X_FORWARDED_FOR | Request::HEADER_X_FORWARDED_HOST | Request::HEADER_X_FORWARDED_PORT | Request::HEADER_X_FORWARDED_PROTO;
}
```

**AWS Elastic Load Balancing:**

- `$headers` should be `Request::HEADER_X_FORWARDED_AWS_ELB`.
- Check Symfony's proxy documentation details `$headers` constants.
- This is for specific cloud load balancer setup.
- It ensures correct header handling.
- This configuration is required for cloud environment.

**Trusting All Proxies:**

- Use `*` in proxies property to trusts all proxies.
- This is for cloud load balancer providers.
- It's used when balancer IPs are unknown.
- This simplifies configuration in cloud environments.

*Example:*

```php
/**
 * The trusted proxies for this application.
 *
 * @var string|array
 */
protected $proxies = '*';
```

### &#10022; Configure Trusted Hosts:

- Laravel handles all requests by default.
- It disregards the HTTP request's Host header.
- The Host header value is used for absolute URL generation.
- This applies during web requests.
- This is the default behaviour.

- Web servers should typically filter requests based on host names.
- If web server customization is limited, `App\Http\Middleware\TrustHosts` middleware can be used.
- `TrustHosts` restricts Laravel's responses to specific host names.
- This is a workaround for server configuration limitations.
- It ensures Laravel only responds to desired hosts.

- `TrustHosts` middleware is in the `$middleware` stack.
- It needs to be uncommented to activate.
- The `hosts` method specifies allowed host names.
- Requests with other Host headers are rejected.
- This ensures only specified hosts can access the application.

- `allSubdomainsOfApplicationUrl` method will returns a regex for all subdomains.
- It matches subdomains of `app.url` configuration.
- This helper simplifies utilization for wildcard subdomain.
- It allows all application subdomains.

*Example:*

In `App\Http\Middleware\TrustHosts` middleware class:

```php
/**
 * Get the host patterns that should be trusted.
 *
 * @return array
 */
public function hosts()
{
    return [
        'laravel.test',
        $this->allSubdomainsOfApplicationUrl(),
    ];
}
```

---
[&#8682; To Top](#-requests)

[&#10094; Previous Topic](./controllers.md) &emsp; [Next Topic &#10095;](./responses.md)

[&#8962; Goto Home Page](../README.md)