## &#10162; Artisan Commands:

### &#9780; Overview:
1. [What is Artisan](#-what-is-artisan)
2. [Basic Usage](#-basic-usage)
3. [Common Artisan Commands](#-common-artisan-commands)

### &#10022; What is Artisan:

Artisan is a command-line tool included with Laravel that automates and simplifies many repetitive development tasks. It is designed to automate tasks such as:

- Generating boilerplate code such as controllers, models and migrations.
- Running database migrations and seeders.
- Managing routes.
- Caching and clearing caches.
- Running tests.
- And much more.

### &#10022; Basic Usage:

- To run an Artisan command, open the terminal and navigate to the installed Laravel project root directory.
- Then, use the following syntax:
```bash
php artisan command:name [options] [arguments]
```
- Where,
	- `command:name`: The name of the Artisan command you want to run.
	- `[options]`: Optional flags that sets the command's behavior.
	- `[arguments]`: Optional values that the command might requires.


### &#10022; Common Artisan Commands:

- Make Commands for Generating Code:
  - `php artisan make:controller ControllerName`: Creates a new controller.
  - `php artisan make:model ModelName`: Creates a new Eloquent model.
  - `php artisan make:migration create_table_name_table`: Creates a new migration file.
  - `php artisan make:seeder DatabaseSeederName`: Creates a new database seeder.
  - `php artisan make:factory ModelFactoryName`: Creates a new factory.
  - `php artisan make:command CommandName`: Creates a new custom Artisan command.
  - `php artisan make:middleware MiddlewareName`: Creates a new middleware.
  - `php artisan make:request RequestName`: Creates a new form request class.
  - `php artisan make:resource ResourceName`: Creates a new resource.
- Database Commands:
  - `php artisan migrate`: Runs database migrations.
  - `php artisan migrate:fresh`: Drops all tables and re-runs all migrations.
  - `php artisan migrate:rollback`: Rolls back the latest migration.
  - `php artisan migrate:status`: Shows the status of migrations.
  - `php artisan db:seed`: Runs database seeders.
  - `php artisan db:seed --class=DatabaseSeederName`: Runs a specific seeder.
- Routing Commands:
  - `php artisan route:list`: Lists all registered routes.
  - `php artisan route:cache`: Caches route definitions.
  - `php artisan route:clear`: Clears the route cache.
- Cache Commands:
  - `php artisan config:cache`: Caches configuration files.
  - `php artisan config:clear`: Clears the configuration cache.
  - `php artisan cache:clear`: Clears the application cache.
  - `php artisan view:clear`: Clears compiled view files.
- Environment Commands:
  - `php artisan key:generate`: Generates an application key.
  - `php artisan env`: Displays the current application environment.
- Server Commands:
  - `php artisan serve`: Starts the built-in development server.
- Queue Commands:
  - `php artisan queue:work`: Processes queued jobs.
  - `php artisan queue:listen`: Listens for queued jobs.
- Testing Commands:
  - `php artisan test`: Runs PHPUnit tests.
- Optimization Commands:
  - `php artisan optimize`: Optimizes the application for production.
  - `php artisan optimize:clear`: Clears the optimized bytecode cache.
- Artisan list:
  - `php artisan list`: Displays a list of all available Artisan commands.

---
[&#8682; To Top](#-artisan-commands)

[&#10094; Previous Topic](./introduction.md) &emsp; [Next Topic &#10095;](./routing.md)

[&#8962; Goto Home Page](../README.md)