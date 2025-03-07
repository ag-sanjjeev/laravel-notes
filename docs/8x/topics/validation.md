## &#10162; Validation:

- Laravel offers many ways to validate incoming request data.
- The `validate` method on HTTP requests is common.
- Other validation approaches exist.
- Laravel has many validation rules.
- These rules can check for unique values in a database table.
- This helps a user to use different validation features.

### &#9780; Overview:

1. [Implementing Validation Logic](#-implementing-validation-logic)
    - [Stopping On First Validation Failure](#-stopping-on-first-validation-failure)
    - [Nested Attributes Validation](#-nested-attributes-validation)
    - [Displaying The Validation Errors](#-displaying-the-validation-errors)
    - [Customizing Error Messages With Localization](#-customizing-error-messages-with-localization)
    - [XHR Requests and Validation](#-xhr-requests-and-validation)
    - [Error Directive](#-error-directive)
    - [Repopulating Forms](#-repopulating-forms)
    - [Optional Fields](#-optional-fields)
2. [Form Request Validation](#-form-request-validation)
    - [Adding After Hooks To Form Requests](#-adding-after-hooks-to-form-requests)
    - [Stopping On First Validation Failure Attribute](#-stopping-on-first-validation-failure-attribute)
    - [Customize Redirect Location](#-customize-redirect-location)
    - [Authorizing Form Requests](#-authorizing-form-requests)
    - [Customize Error Messages](#-customize-error-messages)
    - [Customize Validation Attributes](#-customize-validation-attributes)
    - [Preparing Input For Validation](#-preparing-input-for-validation)
3. [Manually Creating Validators](#-manually-creating-validators)
    - [Automatic Redirection](#-automatic-redirection)
    - [Named Error Bags](#-named-error-bags)
    - [Customize Error Messages in Manual Validator](#-customize-error-messages-in-manual-validator)
    - [After Validation Hook](#-after-validation-hook)
4. [Working With Validated Input](#-working-with-validated-input)
5. [Working With Error Messages](#-working-with-error-messages)
    - [Specifying Custom Messages In Language Files](#-specifying-custom-messages-in-language-files)
    - [Specifying Attributes In Language Files](#-specifying-attributes-in-language-files)
    - [Specifying Values In Language Files](#-specifying-values-in-language-files)
6. [Available Validation Rules](#-available-validation-rules)
7. [Conditionally Adding Rules](#-conditionally-adding-rules)
8. [Validating Arrays](#-validating-arrays)
    - [Excluding Unvalidated Array Keys](#-excluding-unvalidated-array-keys)
    - [Validating Nested Array Input](#-validating-nested-array-input)
9. [Validating Passwords](#-validating-passwords)
10. [Custom Validation Rules](#-custom-validation-rules)
    - [Using Rule Objects With Closures](#-using-rule-objects-with-closures)
    - [Implicit Rules](#-implicit-rules)

### &#10022; Implementing Validation Logic:

- The `store` method validates the new blog post.
- The `validate` method of the `Illuminate\Http\Request` object is used.
- If validation passes, the code continues.
- If validation fails, an `Illuminate\Validation\ValidationException` is thrown.
- An error response is automatically sent.
- For HTTP requests, a redirect to the previous URL is generated.
- For XHR requests, a JSON response with error messages is returned.
- Example `store` method:

```php
/**
 * Store a new blog post.
 *
 * @param  \Illuminate\Http\Request  $request
 * @return \Illuminate\Http\Response
 */
public function store(Request $request)
{
    $validated = $request->validate([
        'title' => 'required|unique:posts|max:255',
        'content' => 'required',
    ]);

    // The blog post is valid...
}
```

- Validation rules are passed to the `validate` method.
- If validation fails, a response is generated.
- If validation passes, the controller continues.
- Validation rules also be arrays instead of a string.

```php
$validatedData = $request->validate([
    'title' => ['required', 'unique:posts', 'max:255'],
    'content' => ['required'],
]);
```

- The `validateWithBag` method validates and stores errors in a named error bag.

```php
$validatedData = $request->validateWithBag('post', [
    'title' => ['required', 'unique:posts', 'max:255'],
    'content' => ['required'],
]);
```

### &#10022; Stopping On First Validation Failure:

- The `bail` rule stops validation after the first failure.

```php
$request->validate([
    'title' => 'bail|required|unique:posts|max:255',
    'body' => 'required',
]);
```

- Rules are validated in the order they are assigned.

### &#10022; Nested Attributes Validation:

- "Nested" field data is specified with `.` dot character syntax.

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'article.content' => 'required',
    'article.tags' => 'required',
]);
```

- A literal period in a field name is escaped with a backslash:

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'v1\.0' => 'required',
]);
```

### &#10022; Displaying The Validation Errors:

- If request fields fail validation, Laravel redirects the user back.
- Validation errors and request input are flashed to the session.
- An `$errors` variable is shared with all views by the `Illuminate\View\Middleware\ShareErrorsFromSession` middleware.
- This middleware is in the `web` middleware group.
- An `$errors` variable is always available in views.
- `$errors` is an instance of `Illuminate\Support\MessageBag`.
- In the example, the user is redirected to the `create` method on validation failure.
- This allows displaying error messages in the view:

```blade
<h1>Create Post</h1>

@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

```

### &#10022; Customizing Error Messages With Localization:

- Laravel's validation rules have error messages in `resources/lang/en/validation.php`.
- Each validation rule has a translation entry.
- A user can change or modify these messages.
- A user can copy this file to translate messages for other languages.
- For Laravel localization, see the [localization](./localization.md) notes.

### &#10022; XHR Requests and Validation:

- The example uses a traditional form.
- Many applications use XHR requests.
- During an XHR request, Laravel does not redirect.
- Laravel generates a JSON response with validation errors.
- The JSON response has a `422` HTTP status code.

### &#10022; Error Directive:

- The `@error` Blade directive checks for validation error messages.
- Inside `@error`, the `$message` variable displays the error message.

```blade
<label for="title">Post Title</label>

<input id="title" type="text" name="title" class="@error('title') is-invalid @enderror">

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

- For named error bags, pass the error bag name as the second argument to `@error`:

```blade
<input ... class="@error('title', 'post') is-invalid @enderror">
```

Refer [Blade Template](./blade-template.md)

### &#10022; Repopulating Forms:

- When Laravel redirects due to a validation error, the framework flashes all request input to the session.
- This allows accessing the input in the next request.
- This repopulates the form the user tried to submit.
- To get flashed input from the previous request, use the `old` method on an `Illuminate\Http\Request` instance:

```php
$title = $request->old('title');
```

- Laravel also provides a global `old` helper.
- In Blade templates, the `old` helper is more convenient for repopulating the form.
- If no old input exists for the field, `null` is returned.

```blade
<input type="text" name="title" value="{{ old('title') }}">
```

### &#10022; Optional Fields:

- Laravel includes the `TrimStrings` and `ConvertEmptyStringsToNull` middleware by default.
- These middleware are in the `App\Http\Kernel` class.
- "Optional" request fields should be marked as `nullable`.
- This prevents the validator from considering `null` values as invalid.
- Example:

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'content' => 'required',
    'publish_at' => 'nullable|date',
]);
```

- In this example, `publish_at` can be `null` or a valid date.
- Without the `nullable` modifier, `null` is considered an invalid date.

### &#10022; Form Request Validation:

**Creating Form Requests:**

- For complex validation, create a "form request".
- Form requests are custom request classes.
- They encapsulate validation and authorization logic.
- Use the `make:request` Artisan command:

```bash
php artisan make:request StorePostRequest
```

- Form requests are in the `app/Http/Requests` directory.
- Each form request has `authorize` and `rules` methods.
- `authorize` checks if the authenticated user can perform the action.
- `rules` returns validation rules for the request's data:

```php
/**
 * Get the validation rules that apply to the request.
 *
 * @return array
 */
public function rules()
{
    return [
        'title' => 'required|unique:posts|max:255',
        'content' => 'required',
    ];
}
```

- Dependencies can be type-hinted in the `rules` method.
- They are resolved by the Laravel service container.
- Type-hint the form request on the controller method.
- The form request is validated before the controller method is called.
- Example `store` method:

```php
/**
 * Store a new blog post.
 *
 * @param  \App\Http\Requests\StorePostRequest  $request
 * @return Illuminate\Http\Response
 */
public function store(StorePostRequest $request)
{
    // The incoming request is valid...

    // Retrieve the validated input data...
    $validated = $request->validated();

    // Retrieve a portion of the validated input data...
    $validated = $request->safe()->only(['name', 'email']);
    $validated = $request->safe()->except(['name', 'email']);
}
```

- If validation fails, a redirect response is generated.
- Errors are flashed to the session.
- For XHR requests, a `422` HTTP response with JSON errors is returned.

### &#10022; Adding After Hooks To Form Requests:

- Use the `withValidator` method to add an `after` validation hook.
- This method receives the validator.
- Example:

```php
/**
 * Configure the validator instance.
 *
 * @param  \Illuminate\Validation\Validator  $validator
 * @return void
 */
public function withValidator($validator)
{
    $validator->after(function ($validator) {
        if ($this->somethingElseIsInvalid()) {
            $validator->errors()->add('field', 'Something is wrong with this field!');
        }
    });
}
```

### &#10022; Stopping On First Validation Failure Attribute:

- Add a `stopOnFirstFailure` property in request class to stop validation after the first failure.

```php
/**
 * Indicates if the validator should stop on the first rule failure.
 *
 * @var bool
 */
protected $stopOnFirstFailure = true;
```

### &#10022; Customize Redirect Location:

- By default, a redirect sends the user back.
- Customize this with a `$redirect` property in form request class.

```php
/**
 * The URI that users should be redirected to if validation fails.
 *
 * @var string
 */
protected $redirect = '/dashboard';
```

- Or, redirect to a named route with `$redirectRoute`:

```php
/**
 * The route that users should be redirected to if validation fails.
 *
 * @var string
 */
protected $redirectRoute = 'dashboard';
```

### &#10022; Authorizing Form Requests:

- Form requests have an `authorize` method.
- This method checks if the authenticated user can update a resource.
- A user can check if a user owns a blog comment.
- Authorization gates and policies are used in this method.

```php
use App\Models\Comment;

/**
 * Determine if the user is authorized to make this request.
 *
 * @return bool
 */
public function authorize()
{
    $comment = Comment::find($this->route('comment'));

    return $comment && $this->user()->can('update', $comment);
}
```

- Form requests extend the base Laravel request class.
- The `user` method accesses the authenticated user.
- The `route` method accesses URI parameters.
- Example route:

```php
Route::post('/comment/{comment}');
```

- With route model binding, the resolved model can be accessed as a property.

```php
return $this->user()->can('update', $this->comment);
```

- If `authorize` returns `false`, a `403` HTTP response is returned.
- The controller method does not execute.
- If authorization is handled elsewhere, return `true` from `authorize` method.

```php
/**
 * Determine if the user is authorized to make this request.
 *
 * @return bool
 */
public function authorize()
{
    return true;
}
```

- Dependencies can be type-hinted in the `authorize` method.
- They are resolved by the Laravel service container.

### &#10022; Customize Error Messages:

- Customize error messages by overriding the `messages` method.
- This method returns an array of attribute/rule pairs and their error messages.

```php
/**
 * Get the error messages for the defined validation rules.
 *
 * @return array
 */
public function messages()
{
    return [
        'title.required' => 'A title is required',
        'content.required' => 'A article content is required',
    ];
}
```

### &#10022; Customize Validation Attributes:

- Laravel's validation error messages contain an `:attribute` placeholder.
- Replace the `:attribute` placeholder with a custom attribute name.
- Override the `attributes` method to specify custom names.
- This method returns an array of attribute/name pairs:

```php
/**
 * Get custom attributes for validator errors.
 *
 * @return array
 */
public function attributes()
{
    return [
        'email' => 'email address',
    ];
}
```

### &#10022; Preparing Input For Validation:

- If data needs to be prepared or sanitized before validation, use the `prepareForValidation` method:

```php
use Illuminate\Support\Str;

/**
 * Prepare the data for validation.
 *
 * @return void
 */
protected function prepareForValidation()
{
    $this->merge([
        'slug' => Str::slug($this->slug),
    ]);
}
```

### &#10022; Manually Creating Validators:

- If the `validate` method on the request is not used, a validator instance can be manually created.
- Use the `Validator` facade.
- The `make` method on the facade generates a new validator instance:

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Validator;

class PostController extends Controller
{
    /**
     * Store a new blog post.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'title' => 'required|unique:posts|max:255',
            'content' => 'required',
        ]);

        if ($validator->fails()) {
            return redirect('post/create')
                        ->withErrors($validator)
                        ->withInput();
        }

        // Retrieve the validated input...
        $validated = $validator->validated();

        // Retrieve a portion of the validated input...
        $validated = $validator->safe()->only(['name', 'email']);
        $validated = $validator->safe()->except(['name', 'email']);

        // Store the blog post...
    }
}
```

- The first argument to `make` is the data under validation.
- The second argument is an array of validation rules.
- After checking if validation failed, use `withErrors` to flash error messages to the session.
- The `$errors` variable is shared with views after redirection.
- `withErrors` accepts a validator, a `MessageBag`, or a PHP array.

**Stopping On First Validation Failure:**

- The `stopOnFirstFailure` method stops validation after the first failure.

```php
if ($validator->stopOnFirstFailure()->fails()) {
    // ...
}
```

### &#10022; Automatic Redirection:

- To create a validator instance manually and use automatic redirection, call the `validate` method.
- If validation fails, the user is redirected, or a JSON response is returned for XHR requests.

```php
Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
])->validate();
```

- Use `validateWithBag` to store error messages in a named error bag.

```php
Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
])->validateWithBag('post');
```

### &#10022; Named Error Bags:

- If there are multiple forms on a page, name the `MessageBag` with validation errors.
- This allows retrieving errors for a specific form.
- Pass a name as the second argument to `withErrors`:

```php
return redirect('register')->withErrors($validator, 'login');
```

- Access the named `MessageBag` instance from the `$errors` variable:

```blade
{{ $errors->login->first('email') }}
```

### &#10022; Customize Error Messages in Manual Validator:

- Custom error messages can be provided to a validator instance.
- Pass custom messages as the third argument to `Validator::make`:

```php
$validator = Validator::make($input, $rules, $messages = [
    'required' => 'The :attribute field is required.',
]);
```

- The `:attribute` placeholder is replaced by the field name.
- Other placeholders can be used in validation messages:

```php
$messages = [
    'same' => 'The :attribute and :other must match.',
    'size' => 'The :attribute must be exactly :size.',
    'between' => 'The :attribute value :input is not between :min - :max.',
    'in' => 'The :attribute must be one of the following types: :values',
];
```

**Specifying A Custom Message For A Given Attribute:**

- A custom error message can be specified for a specific attribute.
- Use `.` dot character notation.

```php
$messages = [
    'email.required' => 'Please give email address to proceed',
];
```

**Specifying Custom Attribute Values:**

- Laravel's error messages include an `:attribute` placeholder.
- Customize the values used to replace placeholders for specific fields.
- Pass an array of custom attributes as the fourth argument to `Validator::make`:

```php
$validator = Validator::make($input, $rules, $messages, [
    'email' => 'email address',
]);
```

### &#10022; After Validation Hook:

- Callbacks can be attached to run after validation.
- This allows performing further validation and adding error messages.
- Call the `after` method on a validator instance:

```php
$validator = Validator::make(...);

$validator->after(function ($validator) {
    if ($this->somethingElseIsInvalid()) {
        $validator->errors()->add(
            'field', 'Something is wrong with this field!'
        );
    }
});

if ($validator->fails()) {
    //
}
```

### &#10022; Working With Validated Input:

- After validating request data, retrieve the validated data.
- Use the `validated` method on a form request or validator instance.
- This returns an array of validated data.

```php
$validated = $request->validated();

$validated = $validator->validated();
```

- Or, use the `safe` method.
- This returns an `Illuminate\Support\ValidatedInput` instance.
- This object has `only`, `except`, and `all` methods.

```php
$validated = $request->safe()->only(['name', 'email']);

$validated = $request->safe()->except(['name', 'email']);

$validated = $request->safe()->all();
```

- The `Illuminate\Support\ValidatedInput` instance can be iterated and accessed like an array.

```php
// Validated data may be iterated...
foreach ($request->safe() as $key => $value) {
    //
}

// Validated data may be accessed as an array...
$validated = $request->safe();

$email = $validated['email'];
```

- Add fields to the validated data with the `merge` method:

```php
$validated = $request->safe()->merge(['name' => 'Kumar']);
```

- Retrieve validated data as a collection instance with the `collect` method.

```php
$collection = $request->safe()->collect();
```

### &#10022; Working With Error Messages:

- The `errors` method on a `Validator` instance returns an `Illuminate\Support\MessageBag` instance.
- The `$errors` variable in views is also a `MessageBag` instance.

**Retrieving First Error Message For A Field:**

- Use the `first` method to get the first error message for a field.

```php
$errors = $validator->errors();

echo $errors->first('email');
```

**Retrieving All Error Messages For A Field:**

- Use the `get` method to get an array of all messages for a field.

```php
foreach ($errors->get('email') as $message) {
    //
}
```

- For array form fields, use `*` to get messages for each element.

```php
foreach ($errors->get('attachments.*') as $message) {
    //
}
```

**Retrieving All Error Messages For All Fields:**

- Use the `all` method to get an array of all messages for all fields.

```php
foreach ($errors->all() as $message) {
    //
}
```

**Determining If Messages Exist For A Field:**

- Use the `has` method to check if error messages exist for a field.

```php
if ($errors->has('email')) {
    //
}
```

### &#10022; Specifying Custom Messages In Language Files.

- Laravel's validation rules have error messages in `resources/lang/en/validation.php`.
- Each validation rule has a translation entry.
- A user can change or modify these messages.
- A user can copy this file to translate messages for other languages.
- For Laravel localization, see the [localization](./localization.md) notes.

**Custom Messages For Specific Attributes:**

- Customize error messages for specific attribute and rule combinations in validation language files.
- Add message customizations to the `custom` array of `resources/lang/xx/validation.php`:

```php
'custom' => [
    'email' => [
        'required' => 'Please give email address!',
        'max' => 'Given Email address is too long!'
    ],
],
```

### &#10022; Specifying Attributes In Language Files:

- Laravel's error messages include an `:attribute` placeholder.
- Replace the `:attribute` placeholder with a custom value.
- Specify the custom attribute name in the `attributes` array of `resources/lang/xx/validation.php`:

```php
'attributes' => [
    'email' => 'email address',
],
```

### &#10022; Specifying Values In Language Files:

- Laravel's error messages contain a `:value` placeholder.
- This placeholder is replaced with the request attribute's current value.
- Sometimes, a custom representation of the value is needed.
- Example rule:

```php
Validator::make($request->all(), [
    'credit_card_number' => 'required_if:payment_type,cc'
]);
```

- If this rule fails, the error message is as below:

```
The credit card number details is required when payment type is choosen as cc.
```

- Instead of `cc`, a user-friendly value can be specified.
- Define a `values` array in `resources/lang/xx/validation.php`.

```php
'values' => [
    'payment_type' => [
        'cc' => 'credit card'
    ],
],
```

- After defining this value, the error message is:

```
The credit card number details is required when payment type is choose as credit card.
```

### &#10022; Available Validation Rules:

Here's the list of available validation rules:

- Accepted
- Accepted If
- Active URL
- After (Date)
- After Or Equal (Date)
- Alpha
- Alpha Dash
- Alpha Numeric
- Array
- Bail
- Before (Date)
- Before Or Equal (Date)
- Between
- Boolean
- Confirmed
- Current Password
- Date
- Date Equals
- Date Format
- Declined
- Declined If
- Different
- Digits
- Digits Between
- Dimensions (Image Files)
- Distinct
- Email
- Ends With
- Enum
- Exclude
- Exclude If
- Exclude Unless
- Exclude Without
- Exists (Database)
- File
- Filled
- Greater Than
- Greater Than Or Equal
- Image (File)
- In
- In Array
- Integer
- IP Address
- MAC Address
- JSON
- Less Than
- Less Than Or Equal
- Max
- MIME Types
- MIME Type By File Extension
- Min
- Multiple Of
- Not In
- Not Regex
- Nullable
- Numeric
- Password
- Present
- Prohibited
- Prohibited If
- Prohibited Unless
- Prohibits
- Regular Expression
- Required
- Required If
- Required Unless
- Required With
- Required With All
- Required Without
- Required Without All
- Same
- Size
- Sometimes
- Starts With
- String
- Timezone
- Unique (Database)
- URL
- UUID

**accepted**

- The field must be "yes", "on", 1, or true.
- Useful for validating "Terms of Service" acceptance.

**accepted_if:anotherfield,value,...**

- The field must be "yes", "on", 1, or true if another field equals a specified value.
- Useful for validating "Terms of Service" acceptance.

**active_url**

- The field must have a valid A or AAAA record.
- Uses the `dns_get_record` PHP function.
- Hostname is extracted using `parse_url` before being passed to `dns_get_record`.

**after:date**

- The field must be a value after a given date.
- Dates are passed to the `strtotime` PHP function.
- Example: `'start_date' => 'required|date|after:tomorrow'`
- Another field can be compared to the date: `'finish_date' => 'required|date|after:start_date'`

**after_or_equal:date**

- The field must be a value after or equal to the given date.
- Similar to `after:date` usage.

**alpha**

- The field must contain only alphabetic characters.

**alpha_dash**

- The field can contain alpha-numeric characters, dashes, and underscores.

**alpha_num**

- The field must contain only alpha-numeric characters.

**array**

- The field must be a PHP array.
- When additional values are given, each key in the input array must be in the list of values.
- Example:

```php
use Illuminate\Support\Facades\Validator;

$input = [
 'user' => [
  'name' => 'Kumar',
  'username' => 'kumar',
  'admin' => true,
 ],
];

Validator::make($input, [
 'user' => 'array:username,locale',
]);
```

- Always specify allowed array keys.
- Otherwise, `validate` and `validated` methods return all data, including unvalidated keys.
- To exclude unvalidated array keys, call `excludeUnvalidatedArrayKeys` in the `boot` method of `AppServiceProvider`.

```php
use Illuminate\Support\Facades\Validator;

/**
 * Register any application services.
 *
 * @return void
 */
public function boot()
{
 Validator::excludeUnvalidatedArrayKeys();
}
```

**bail**

- Stops running validation rules for a field after the first failure.
- `stopOnFirstFailure` stops validating all attributes after the first failure:

```php
if ($validator->stopOnFirstFailure()->fails()) {
 // ...
}
```

**before:date**

- The field must be a value preceding the given date.
- Dates are passed to `strtotime`.
- Another field can be used as the date.
- Similar to `after:date` usage.

**before_or_equal:date**

- The field must be a value preceding or equal to the given date.
- Dates are passed to `strtotime`.
- Another field can be used as the date.
- Similar to `after_or_equal:date` usage.

**between:min,max**

- The field must have a size between the given min and max.
- Strings, numerics, arrays, and files are evaluated like the `size` rule.

**boolean**

- The field must be able to be cast as a boolean.
- Accepted inputs: `true`, `false`, `1`, `0`, `"1"`, `"0"`.

**confirmed**

- The field must have a matching `{field}_confirmation` field.
- Example: `password` requires `password_confirmation`.

**current_password**

- The field must match the authenticated user's password.
- Specify an authentication guard: `'password' => 'current_password:api'`

**date**

- The field must be a valid, non-relative date according to `strtotime`.

**date_equals:date**

- The field must be equal to the given date.
- Dates are passed to `strtotime`.

**date_format:format**

- The field must match the given format.
- Use either `date` or `date_format`, not both.
- Supports all formats supported by PHP's `DateTime` class.

**declined**

- The field must be `"no"`, `"off"`, `0`, or `false`.

**declined_if:anotherfield,value,...**

- The field must be `"no"`, `"off"`, `0`, or `false` if another field equals a specified value.

**different:field**

- The field must have a different value than `field`.

**digits:value**

- The field must be numeric and have an exact length of `value`.

**digits_between:min,max**

- The field must be numeric and have a length between `min` and `max`.

**dimensions**

- The file must be an image meeting dimension constraints.
- Example: `'profile_photo' => 'dimensions:min_width=100,min_height=100'`
- Available constraints: `min_width`, `max_width`, `min_height`, `max_height`, `width`, `height`, `ratio`.
- Ratio: `width / height` (e.g., `16/9` or `1.78`).
- Use `Rule::dimensions` for fluent construction.

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
 'profile_photo' => [
  'required',
  Rule::dimensions()->maxWidth(1600)->maxHeight(900)->ratio(16 / 9),
 ],
]);
```

**distinct**

- For arrays, the field must not have duplicate values: `'something.*.id' => 'distinct'`
- Uses loose variable comparisons by default.
- Use strict comparisons: `'something.*.id' => 'distinct:strict'`
- Ignore capitalization differences: `'something.*.id' => 'distinct:ignore_case'`

**email**

- The field must be formatted as an email address.
- Uses the `egulias/email-validator` package.
- Default: `RFCValidation`.
- Other validation styles:
 - `rfc`: `RFCValidation`
 - `strict`: `NoRFCWarningsValidation`
 - `dns`: `DNSCheckValidation`
 - `spoof`: `SpoofCheckValidation`
 - `filter`: `FilterEmailValidation`
- Example: `'email' => 'email:rfc,dns'`
- `filter` uses PHP's `filter_var`.
- `dns` and `spoof` require the `intl` extension installed and enabled.

**ends_with:something,another,...**

- The field must end with one of the given values.

**enum**

- For enum support requires PHP >=8.1
- Class-based rule that validates enum values.
- Accepts the enum name as a constructor argument.
- Example:

```php
use App\Enums\ServerStatus;
use Illuminate\Validation\Rules\Enum;

$request->validate([
 'status' => [new Enum(ServerStatus::class)],
]);
```

**exclude**

- The field is excluded from the request data returned by `validate` and `validated`.

**exclude_if:anotherfield,value**

- The field is excluded if `anotherfield` equals `value`.

**exclude_unless:anotherfield,value**

- The field is excluded unless `anotherfield` equals `value`.
- If `value` is `null`, the field is excluded unless `anotherfield` is `null` or missing.

**exclude_without:anotherfield**

- The field is excluded if `anotherfield` is not present.

**exists:table,column**

- The field must exist in a given database table.

**Basic Usage Of Exists Rule**

- `'state' => 'exists:states'`
- If `column` is not specified, the field name is used.

**Specifying A Custom Column Name**

- `'state' => 'exists:states,abbreviation'`

**Specifying A Custom Connection**

- `'email' => 'exists:connection.staff,email'`

**Using Eloquent Model**

- `'user_id' => 'exists:App\Models\User,id'`

**Customizing The Query**

- Use the `Rule` class:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
 'email' => [
  'required',
  Rule::exists('staff')->where(function ($query) {
   return $query->where('account_id', 1);
  }),
 ],
]);
```

**file**

- The field must be a successfully uploaded file.

**filled**

- The field must not be empty when it is present.

**gt:field**

- The field must be greater than the given field.
- The two fields must be of the same type.
- Strings, numerics, arrays, and files are evaluated like the `size` rule.

**gte:field**

- The field must be greater than or equal to the given field.
- The two fields must be of the same type.
- Strings, numerics, arrays, and files are evaluated like the `size` rule.

**image**

- The file must be an image (jpg, jpeg, png, bmp, gif, svg, or webp).

**in:value1,value2,...**

- The field must be included in the given list of values.
- Use `Rule::in` to fluently construct the rule:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
 'std_code' => [
  'required',
  Rule::in(['044', '0422']),
 ],
]);
```

- When combined with the `array` rule, each value in the input array must be in the list of values.
- Example:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

$input = [
 'delivery_location' => ['HOME', 'OFFICE'],
];

Validator::make($input, [
 'delivery_location' => [
  'required',
  'array',
  Rule::in(['HOME', 'ADDRESS']),
 ],
]);
```

**in_array:anotherfield.***

- The field must exist in `anotherfield`'s values.

**integer**

- The field must be an integer.
- Does not verify the "integer" variable type, only a type accepted by `FILTER_VALIDATE_INT`.
- Use with the `numeric` rule to validate as a number.

**ip**

- The field must be an IP address.

**ipv4**

- The field must be an IPv4 address.

**ipv6**

- The field must be an IPv6 address.

**mac_address**

- The field must be a MAC address.

**json**

- The field must be a valid JSON string.

**lt:field**

- The field must be less than the given field.
- The two fields must be of the same type.
- Strings, numerics, arrays, and files are evaluated like the `size` rule.

**lte:field**

- The field must be less than or equal to the given field.
- The two fields must be of the same type.
- Strings, numerics, arrays, and files are evaluated like the `size` rule.

**max:value**

- The field must be less than or equal to a maximum value.
- Strings, numerics, arrays, and files are evaluated like the `size` rule.

**mimetypes:text/plain,...**

- The file must match one of the given MIME types.
- Example: `'video' => 'mimetypes:video/avi,video/mpeg,video/quicktime'`
- MIME type is determined by reading the file's contents.

**mimes:type1,type2,...**

- The file must have a MIME type corresponding to one of the listed extensions.
- Example: `'photo' => 'mimes:jpg,bmp,png'`
- Validates the MIME type by reading the file's contents.

**min:value**

- The field must have a minimum value.
- Strings, numerics, arrays, and files are evaluated like the `size` rule.

**multiple_of:value**

- The field must be a multiple of `value`.
- Requires the `bcmath` PHP extension.

**not_in:value1,value2,...**

- The field must not be included in the given list of values.
- Use `Rule::notIn` to fluently construct the rule:

```php
use Illuminate\Validation\Rule;

Validator::make($data, [
 'tags' => [
  'required',
  Rule::notIn(['html', 'css', 'js']),
 ],
]);
```

**not_regex:pattern**

- The field must not match the given regular expression.
- Uses `preg_match`.
- Pattern must obey `preg_match` formatting and include valid delimiters.
- Example: `'email' => 'not_regex:/^.+$/i'`
- Use an array for validation rules if the regular expression contains a `|` character.

**nullable**

- The field may be `null`.

**numeric**

- The field must be numeric.

**password**

- The field must match the authenticated user's password.
- Renamed to `current_password` and will be removed in Laravel 9.
- Use the `current_password` rule.

**present**

- The field must be present in the input data but can be empty.

**prohibited**

- The field must be empty or not present.

**prohibited_if:anotherfield,value,...**

- The field must be empty or not present if `anotherfield` equals any `value`.

**prohibited_unless:anotherfield,value,...**

- The field must be empty or not present unless `anotherfield` equals any `value`.

**prohibits:anotherfield,...**

- If the field is present, no fields in `anotherfield` can be present, even if empty.

**regex:pattern**

- The field must match the given regular expression.
- Uses `preg_match`.
- Pattern must obey `preg_match` formatting and include valid delimiters.
- Example: `'email' => 'regex:/^.+@.+$/i'`
- Use an array for validation rules if the regular expression contains a `|` character.

**required**

- The field must be present and not empty.
- "Empty" means:
 - The value is `null`.
 - The value is an empty string.
 - The value is an empty array or `Countable` object.
 - The value is an uploaded file with no path.

**required_if:anotherfield,value,...**

- The field must be present and not empty if `anotherfield` equals any `value`.
- Use `Rule::requiredIf` for complex conditions:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($request->all(), [
 'role_id' => Rule::requiredIf($request->user()->is_admin),
]);

Validator::make($request->all(), [
 'role_id' => Rule::requiredIf(function () use ($request) {
  return $request->user()->is_admin;
 }),
]);
```

**required_unless:anotherfield,value,...**

- The field must be present and not empty unless `anotherfield` equals any `value`.
- `anotherfield` must be present unless `value` is `null`.
- If `value` is `null` (`required_unless:name,null`), the field is required unless `anotherfield` is `null` or missing.

**required_with:field1,field2,...**

- The field must be present and not empty only if any of the other specified fields are present and not empty.

**required_with_all:field1,field2,...**

- The field must be present and not empty only if all of the other specified fields are present and not empty.

**required_without:field1,field2,...**

- The field must be present and not empty only when any of the other specified fields are empty or not present.

**required_without_all:field1,field2,...**

- The field must be present and not empty only when all of the other specified fields are empty or not present.

**same:field**

- The given field must match the field under validation.

**size:value**

- The field must have a size matching the given value.
- String: number of characters.
- Numeric: given integer value (requires `numeric` or `integer` rule).
- Array: count of the array.
- File: file size in kilobytes.
- Examples:
 - `'title' => 'size:12'` (string)
 - `'seats' => 'integer|size:10'` (numeric)
 - `'tags' => 'array|size:5'` (array)
 - `'image' => 'file|size:512'` (file)

**starts_with:value1,value2,...**

- The field must start with one of the given values.

**string**

- The field must be a string.
- Use `nullable` to allow `null`.

**timezone**

- The field must be a valid timezone identifier according to `timezone_identifiers_list`.

**unique:table,column**

- The field must not exist in the given database table.

**Specifying A Custom Table / Column Name:**

- Use an Eloquent model: `'email' => 'unique:App\Models\User,email_address'`
- Specify the column: `'email' => 'unique:users,email_address'`

**Specifying A Custom Database Connection:**

- Prepend the connection name: `'email' => 'unique:connection.users,email_address'`

**Forcing A Unique Rule To Ignore A Given ID:**

- Use the `Rule` class:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
 'email' => [
  'required',
  Rule::unique('users')->ignore($user->id),
 ],
]);
```

- Never pass user-controlled request input to `ignore`.
- Pass a system-generated ID or UUID from an Eloquent model.
- Pass the entire model instance: `Rule::unique('users')->ignore($user)`
- Specify a different primary key column: `Rule::unique('users')->ignore($user->id, 'user_id')`
- Specify a different column to check for uniqueness: `Rule::unique('users', 'email_address')->ignore($user->id)`

**Adding Additional Where Clauses:**

- Use the `where` method:

```php
'email' => Rule::unique('users')->where(function ($query) {
 return $query->where('account_id', 1);
})
```

**url**

- The field must be a valid URL.

**uuid**

- The field must be a valid RFC 4122 UUID (versions 1, 3, 4, or 5).


### &#10022; Conditionally Adding Rules:

**Skipping Validation When Fields Have Certain Values:**

- Use `exclude_if` to skip validation if another field has a certain value.
- Example:

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($data, [
    'has_appointment' => 'required|boolean',
    'appointment_date' => 'exclude_if:has_appointment,false|required|date',
    'doctor_name' => 'exclude_if:has_appointment,false|required|string',
]);
```

- Use `exclude_unless` to skip validation unless another field has a certain value.
- Example:

```php
$validator = Validator::make($data, [
    'has_appointment' => 'required|boolean',
    'appointment_date' => 'exclude_unless:has_appointment,true|required|date',
    'doctor_name' => 'exclude_unless:has_appointment,true|required|string',
]);
```

**Validating When Present:**

- Use the `sometimes` rule to run validation only if a field is present.
- Example:

```php
$v = Validator::make($data, [
    'email' => 'sometimes|required|email',
]);
```

**Complex Conditional Validation:**

- Use the `sometimes` method on a `Validator` instance for complex conditional logic.
- Example:

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($request->all(), [
    'email' => 'required|email',
    'games' => 'required|numeric',
]);

$validator->sometimes('reason', 'required|max:500', function ($input) {
    return $input->games >= 100;
});

$validator->sometimes(['reason', 'cost'], 'required', function ($input) {
    return $input->games >= 100;
});
```

- The `$input` parameter is an `Illuminate\Support\Fluent` instance.

**Complex Conditional Array Validation:**

- Use a second argument in the closure for nested array validation.
- Example:

```php
$input = [
    'channels' => [
        [
            'type' => 'email',
            'address' => '[email protected]',
        ],
        [
            'type' => 'url',
            'address' => 'https://example.host',
        ],
    ],
];

$validator->sometimes('channels.*.address', 'email', function ($input, $item) {
    return $item->type === 'email';
});

$validator->sometimes('channels.*.address', 'url', function ($input, $item) {
    return $item->type !== 'email';
});
```

- The `$item` parameter is an `Illuminate\Support\Fluent` instance or a string.

### &#10022; Validating Arrays:

- The `array` rule accepts a list of allowed array keys.
- Validation fails if additional keys are present.
- Example:

```php
use Illuminate\Support\Facades\Validator;

$input = [
    'user' => [
        'name' => 'Kumar',
        'username' => 'kumar',
        'admin' => true,
    ],
];

Validator::make($input, [
    'user' => 'array:username,locale',
]);
```

- Always specify allowed array keys.
- Otherwise, `validate` and `validated` methods return all data, including unvalidated keys.

### &#10022; Excluding Unvalidated Array Keys:

- Laravel's validator can be configured to exclude unvalidated array keys from the "validated" data.
- Call the `excludeUnvalidatedArrayKeys` method in the `boot` method of `AppServiceProvider`.
- This ensures only specifically validated array keys are included in the "validated" data.
- Example:

```php
use Illuminate\Support\Facades\Validator;

/**
 * Register any application services.
 *
 * @return void
 */
public function boot()
{
    Validator::excludeUnvalidatedArrayKeys();
}
```

### &#10022; Validating Nested Array Input:

- Use `.` dot character notation validates attributes within an array.
- Example:

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($request->all(), [
    'photos.profile' => 'required|image',
]);
```

- Each element of an array can be validated.
- Example:

```php
$validator = Validator::make($request->all(), [
    'person.*.email' => 'email|unique:users',
    'person.*.first_name' => 'required_with:person.*.last_name',
]);
```

- Use the `*` character for custom validation messages in language files.
- Example:

```php
'custom' => [
    'person.*.email' => [
        'unique' => 'Each person must possess a unique email address',
    ]
],
```

### &#10022; Validating Passwords:

- Use Laravel's `Password` rule object for password complexity.
- Example:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rules\Password;

$validator = Validator::make($request->all(), [
    'password' => ['required', 'confirmed', Password::min(8)],
]);
```

- Customize password complexity:
 - `Password::min(8)` (minimum 8 characters)
 - `Password::min(8)->letters()` (at least one letter)
 - `Password::min(8)->mixedCase()` (at least one uppercase and one lowercase letter)
 - `Password::min(8)->numbers()` (at least one number)
 - `Password::min(8)->symbols()` (at least one symbol)
 - `Password::min(8)->uncompromised()` (check for public password data breach leaks)

- `uncompromised` uses the `k-Anonymity` model and `haveibeenpwned.com`.
- Customize the `uncompromised` threshold: `Password::min(8)->uncompromised(3)`

**Defining Default Password Rules:**

- Use `Password::defaults` to specify default password rules.
- Call `Password::defaults` in the `boot` method of a service provider.
- Example:

```php
use Illuminate\Validation\Rules\Password;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Password::defaults(function () {
        $rule = Password::min(8);

        return $this->app->isProduction()
                    ? $rule->mixedCase()->uncompromised()
                    : $rule;
    });
}
```

- Use `Password::defaults()` to apply default rules: `'password' => ['required', Password::defaults()]`
- Add additional validation rules using the `rules` method:

```php
use App\Rules\ZxcvbnRule;

Password::defaults(function () {
    $rule = Password::min(8)->rules([new ZxcvbnRule]);

    // ...
});
```

### &#10022; Custom Validation Rules:

**Using Rule Objects:**

- Create custom validation rules using rule objects.
- Use `make:rule` Artisan command to generate a new rule object.
- Example: `php artisan make:rule Uppercase`
- Laravel places the rule in the `app/Rules` directory.
- Rule objects contain `passes` and `message` methods.
- `passes` method returns `true` or `false` based on validation result.
- `message` method returns the validation error message.
- Example:

```php
namespace App\Rules;

use Illuminate\Contracts\Validation\Rule;

class Uppercase implements Rule
{
    /**
     * Determine if the validation rule passes.
     *
     * @param  string  $attribute
     * @param  mixed  $value
     * @return bool
     */
    public function passes($attribute, $value)
    {
        return strtoupper($value) === $value;
    }

    /**
     * Get the validation error message.
     *
     * @return string
     */
    public function message()
    {
        return 'The :attribute must be uppercase.';
    }
}
```

- Use `trans` helper in the `message` method for translation files.
- Attach the rule to a validator:

```php
use App\Rules\Uppercase;

$request->validate([
    'name' => ['required', 'string', new Uppercase],
]);
```

**Accessing Additional Data:**

- Implement `Illuminate\Contracts\Validation\DataAwareRule` to access all data under validation.
- Define a `setData` method.
- Example:

```php
<?php

namespace App\Rules;

use Illuminate\Contracts\Validation\Rule;
use Illuminate\Contracts\Validation\DataAwareRule;

class Uppercase implements Rule, DataAwareRule
{
    /**
     * All of the data under validation.
     *
     * @var array
     */
    protected $data = [];

    // ...

    /**
     * Set the data under validation.
     *
     * @param  array  $data
     * @return $this
     */
    public function setData($data)
    {
        $this->data = $data;

        return $this;
    }
}
```

- Implement `ValidatorAwareRule` to access the validator instance.
- Example:

```php
namespace App\Rules;

use Illuminate\Contracts\Validation\Rule;
use Illuminate\Contracts\Validation\ValidatorAwareRule;

class Uppercase implements Rule, ValidatorAwareRule
{
    /**
     * The validator instance.
     *
     * @var \Illuminate\Validation\Validator
     */
    protected $validator;

    // ...

    /**
     * Set the current validator.
     *
     * @param  \Illuminate\Validation\Validator  $validator
     * @return $this
     */
    public function setValidator($validator)
    {
        $this->validator = $validator;

        return $this;
    }
}
```


### &#10022; Using Rule Objects With Closures:

- Use a closure for custom rules needed only once.
- The closure receives the attribute's name, value, and a `$fail` callback.
- Call `$fail` if validation fails.
- Example:

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($request->all(), [
    'title' => [
        'required',
        'max:255',
        function ($attribute, $value, $fail) {
            if ($value === 'value1') {
                $fail('The '.$attribute.' is invalid.');
            }
        },
    ],
]);
```


### &#10022; Implicit Rules:

- Normal validation rules (including custom rules) are not run when an attribute is not present or contains an empty string.
- Example: `unique` rule won't run against an empty string.

```php
use Illuminate\Support\Facades\Validator;

$rules = ['name' => 'unique:users,name'];

$input = ['name' => ''];

Validator::make($input, $rules)->passes(); // true
```

- To run a custom rule even when an attribute is empty, the rule must imply that the attribute is required.
- Implement the `Illuminate\Contracts\Validation\ImplicitRule` interface.
- This interface is a `marker` interface (no additional methods beyond those in `Rule`).
- Generate a new implicit rule object using `make:rule` with the `--implicit` option.
- Example: `php artisan make:rule Uppercase --implicit`
- An `implicit` rule only implies that the attribute is required.
- Whether it invalidates a missing or empty attribute is up to the rule's logic.

---
[&#8682; To Top](#-validation)

[&#10094; Previous Topic](./sessions.md) &emsp; [Next Topic &#10095;](./error-handling.md)

[&#8962; Goto Home Page](../README.md)