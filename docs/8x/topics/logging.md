## &#10162; Logging:

- Laravel offers logging services for application insights.
- Logs can be written to files, system error logs, or Slack.
- Logging is based on `channels`, each representing a log writing method.
- Example: `single` channel writes to a single file, `slack` channel sends messages to Slack.
- Log messages can be written to multiple channels based on severity.
- Laravel uses the `Monolog` library, which provides various log handlers.
- Laravel simplifies configuring these handlers for customized log handling.

### &#9780; Overview:

1. [Configuration](#-configuration)
    - [Configuring Log Channel Name](#-configuring-log-channel-name)
    - [Available Channel Drivers](#-available-channel-drivers)
    - [Channel Prerequisites](#-channel-prerequisites)
    - [Logging Deprecation Warnings](#-logging-deprecation-warnings)
2. [Building Log Stacks](#-building-log-stacks)
3. [Writing Log Messages](#-writing-log-messages)
    - [Contextual Information](#-contextual-information)
    - [Writing To Specific Channels](#-writing-to-specific-channels)
4. [Monolog Channel Customization](#-monolog-channel-customization)
    - [Customizing Monolog For Channels](#-customizing-monolog-for-channels)
    - [Creating Custom Channels Via Factories](#-creating-custom-channels-via-factories)

### &#10022; Configuration:

- Logging configuration is in `config/logging.php`.
- This file configures log channels.
- Laravel uses the `stack` channel by default.
- The `stack` channel aggregates multiple log channels into one.

### &#10022; Configuring Log Channel Name:

- Monolog is instantiated with a `channel name` matching the environment (e.g., `production`, `local`).
- Change the channel name with the `name` option in the channel's configuration.
- Example:

```php
'stack' => [
    'driver' => 'stack',
    'name' => 'channel-name',
    'channels' => ['single', 'slack'],
],
```

### &#10022; Available Channel Drivers:

- Each log channel uses a `driver` to determine how and where log messages are recorded.
- Available drivers in Laravel:
    - `custom`: Calls a factory to create a channel.
    - `daily`: `RotatingFileHandler` based, rotates daily.
    - `errorlog`: `ErrorLogHandler` based.
    - `monolog`: Monolog factory, uses any supported Monolog handler.
    - `null`: Discards all log messages.
    - `papertrail`: `SyslogUdpHandler` based.
    - `single`: Single file or path logger (`StreamHandler`).
    - `slack`: `SlackWebhookHandler` based.
    - `stack`: Creates "multi-channel" channels.
    - `syslog`: `SyslogHandler` based.

### &#10022; Channel Prerequisites:

**Configuring The Single and Daily Channels:**

- `single` and `daily` channels have optional configuration options: `bubble`, `permission`, and `locking`.
- Options:
    - `bubble`: Indicates if messages should bubble up to other channels after being handled. Default: `true`.
    - `locking`: Attempts to lock the log file before writing. Default: `false`.
    - `permission`: The log file's permissions. Default: `0644`.

**Configuring The Papertrail Channel:**

- The `papertrail` channel requires `host` and `port` configuration options.
- Obtain these values from Papertrail.

**Configuring The Slack Channel:**

- The `slack` channel requires a `url` configuration option.
- The URL should match an incoming webhook URL configured for your Slack team.
- By default, Slack receives logs at the `critical` level and above.
- Adjust this in `config/logging.php` by modifying the `level` configuration option within the Slack log channel's configuration array.

### &#10022; Logging Deprecation Warnings:

- PHP, Laravel, and libraries notify users of deprecated features.
- Log these warnings by specifying a deprecations log channel in `config/logging.php`.
- Example using `env`:

```php
'deprecations' => env('LOG_DEPRECATIONS_CHANNEL', 'null'),

'channels' => [
    ...
]
```

- Or, define a log channel named `deprecations`.
- If `deprecations` channel exists, it's used for logging deprecations.
- Example defining `deprecations` channel:

```php
'channels' => [
    'deprecations' => [
        'driver' => 'single',
        'path' => storage_path('logs/php-deprecation-warnings.log'),
    ],
],
```

### &#10022; Building Log Stacks:

- The `stack` driver combines multiple channels into one.
- Example configuration for a production application:

```php
'channels' => [
    'stack' => [
        'driver' => 'stack',
        'channels' => ['syslog', 'slack'],
    ],

    'syslog' => [
        'driver' => 'syslog',
        'level' => 'debug',
    ],

    'slack' => [
        'driver' => 'slack',
        'url' => env('LOG_SLACK_WEBHOOK_URL'),
        'username' => 'Laravel Log',
        'emoji' => ':boom:',
        'level' => 'critical',
    ],
],
```

- The `stack` channel aggregates `syslog` and `slack` channels.
- Both channels log messages based on severity/level.

**Log Levels:**

- The `level` option determines the minimum log level for a channel.
- Monolog uses log levels from RFC 5424: `emergency`, `alert`, `critical`, `error`, `warning`, `notice`, `info`, and `debug`.
- Example:

```php
Log::debug('An informational message.');
```

- The `syslog` channel logs this message, but `slack` does not (because it is set to critical).
- Example (emergency level):

```php
Log::emergency('The system is down!');
```

- Both `syslog` and `slack` channels log this message.


### &#10022; Writing Log Messages:

- Use the `Log` facade to write information to logs.
- Logger provides eight logging levels from RFC 5424: `emergency`, `alert`, `critical`, `error`, `warning`, `notice`, `info`, and `debug`.
- Example:

```php
use Illuminate\Support\Facades\Log;

Log::emergency($message);
Log::alert($message);
Log::critical($message);
Log::error($message);
Log::warning($message);
Log::notice($message);
Log::info($message);
Log::debug($message);
```

- Call these methods to log a message for the corresponding level.
- Messages are written to the default log channel (configured in `config/logging.php`).
- Example within a controller:

```php
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Models\User;
use Illuminate\Support\Facades\Log;

class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     *
     * @param  int  $id
     * @return \Illuminate\Http\Response
     */
    public function show($id)
    {
        Log::info('Showing the user profile for user: '.$id);

        return view('user.profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

### &#10022; Contextual Information:

- Pass an array of contextual data to log methods.
- This data is formatted and displayed with the log message.
- Example:

```php
use Illuminate\Support\Facades\Log;

Log::info('User failed to login.', ['id' => $user->id]);
```

- Specify contextual information for all subsequent log entries using `Log::withContext`.
- Example: Logging a request ID for each incoming request.

```php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Str;

class AssignRequestId
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        $requestId = (string) Str::uuid();

        Log::withContext([
            'request-id' => $requestId
        ]);

        return $next($request)->header('Request-Id', $requestId);
    }
}
```

### &#10022; Writing To Specific Channels:

- Log messages to channels other than the default using `Log::channel`.
- Retrieve and log to any channel defined in `config/logging.php`.
- Example:

```php
use Illuminate\Support\Facades\Log;

Log::channel('slack')->info('Something happened!');
```

- Create an on-demand logging stack with multiple channels using `Log::stack`.
- Example:

```php
Log::stack(['single', 'slack'])->info('Something happened!');
```

**On-Demand Channels:**

- Create on-demand channels by providing configuration at runtime.
- Pass a configuration array to `Log::build`.
- Example:

```php
use Illuminate\Support\Facades\Log;

Log::build([
    'driver' => 'single',
    'path' => storage_path('logs/custom.log'),
])->info('Something happened!');
```

- Include an on-demand channel in an on-demand logging stack.
- Include the on-demand channel instance in the array passed to `Log::stack`.
- Example:

```php
use Illuminate\Support\Facades\Log;

$channel = Log::build([
    'driver' => 'single',
    'path' => storage_path('logs/custom.log'),
]);

Log::stack(['slack', $channel])->info('Something happened!');
```

### &#10022; Monolog Channel Customization:

**Customizing Monolog For Channels:**

- Gain complete control over Monolog configuration for existing channels.
- Configure custom `Monolog\FormatterInterface` implementations.
- Define a `tap` array in the channel's configuration.
- `tap` array contains classes to customize the Monolog instance after creation.
- Example:

```php
'single' => [
    'driver' => 'single',
    'tap' => [App\Logging\CustomizeFormatter::class],
    'path' => storage_path('logs/laravel.log'),
    'level' => 'debug',
],
```

- Define the class that customizes the Monolog instance.
- The class needs a `__invoke` method that receives an `Illuminate\Log\Logger` instance.
- `Illuminate\Log\Logger` proxies method calls to the underlying Monolog instance.
- Example:

```php
namespace App\Logging;

use Monolog\Formatter\LineFormatter;

class CustomizeFormatter
{
    /**
     * Customize the given logger instance.
     *
     * @param  \Illuminate\Log\Logger  $logger
     * @return void
     */
    public function __invoke($logger)
    {
        foreach ($logger->getHandlers() as $handler) {
            $handler->setFormatter(new LineFormatter(
                '[%datetime%] %channel%.%level_name%: %message% %context% %extra%'
            ));
        }
    }
}
```

- `Tap` classes are resolved by the service container.
- Constructor dependencies are automatically injected.

### &#10022; Customizing Monolog For Channels:

**Creating Monolog Handler Channels:**

- Monolog has various handlers, but Laravel doesn't have a channel for each.
- Create custom channels using the `monolog` driver for specific Monolog handlers.
- Use the `handler` configuration option to specify the handler class.
- Use the `with` configuration option to provide constructor parameters.
- Example:

```php
'logentries' => [
    'driver'  => 'monolog',
    'handler' => Monolog\Handler\SyslogUdpHandler::class,
    'with' => [
        'host' => 'my.logentries.internal.datahubhost.company.com',
        'port' => '10000',
    ],
],
```

**Monolog Formatters:**

- The Monolog `LineFormatter` is the default formatter for the `monolog` driver.
- Customize the formatter using the `formatter` and `formatter_with` configuration options.
- Example:

```php
'browser' => [
    'driver' => 'monolog',
    'handler' => Monolog\Handler\BrowserConsoleHandler::class,
    'formatter' => Monolog\Formatter\HtmlFormatter::class,
    'formatter_with' => [
        'dateFormat' => 'Y-m-d',
    ],
],
```

- If a Monolog handler provides its own formatter, set `formatter` to `default`.
- Example:

```php
'newrelic' => [
    'driver' => 'monolog',
    'handler' => Monolog\Handler\NewRelicHandler::class,
    'formatter' => 'default',
],
```

### &#10022; Creating Custom Channels Via Factories:

- Define entirely custom channels with full control over Monolog instantiation.
- Use the `custom` driver type in `config/logging.php`.
- Include a `via` option with the factory class name to create the Monolog instance.
- Example:

```php
'channels' => [
    'example-custom-channel' => [
        'driver' => 'custom',
        'via' => App\Logging\CreateCustomLogger::class,
    ],
],
```

- Define the factory class to create the Monolog instance.
- The class needs a `__invoke` method that returns the Monolog logger instance.
- The method receives the channel's configuration array as its argument.
- Example:

```php
namespace App\Logging;

use Monolog\Logger;

class CreateCustomLogger
{
    /**
     * Create a custom Monolog instance.
     *
     * @param  array  $config
     * @return \Monolog\Logger
     */
    public function __invoke(array $config)
    {
        return new Logger(...);
    }
}
```

---
[&#8682; To Top](#-logging)

[&#10094; Previous Topic](./error-handling.md) &emsp; [Next Topic &#10095;](./migrations.md)

[&#8962; Goto Home Page](../README.md)