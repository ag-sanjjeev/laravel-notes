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
. [Input](#-)
    . [Retrieving Input](#-)
    . [Determining If Input Is Present](#-)
    . [Merging Additional Input](#-)
    . [Old Input](#-)
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

### &#10022; Input:

### &#10022; Retrieving Input:

### &#10022; Determining If Input Is Present:

### &#10022; Merging Additional Input:

### &#10022; Old Input:

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