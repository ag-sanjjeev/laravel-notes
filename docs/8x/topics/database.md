## &#10162; Database:

- Laravel simplifies database interactions.
- It offers raw SQL, a fluent query builder, and Eloquent ORM.
- Laravel supports several databases.
    - MariaDB 10.2+
    - MySQL 5.7+
    - PostgreSQL 9.6+
    - SQLite 3.8.8+
    - SQL Server 2017+
- Each database has their version policy.

### &#9780; Overview:

1. [Configuration](#-configuration)
    - [Read & Write Connections](#-read--write-connections)
2. [Running SQL Queries](#-running-sql-queries)
    - [Using Multiple Database Connections](#-using-multiple-database-connections)
    - [Listening For Query Events](#-listening-for-query-events)
3. [Database Transactions](#-database-transactions)
4. [Connecting To The Database CLI](#-connecting-to-the-database-cli)

### &#10022; Configuration:

- Laravel's database configuration resides in `config/database.php`.
- This file defines all database connections and the default connection.
- Environment variables drive most configuration options.
- The file provides examples for supported database systems.
- Database configuration can be modified for local databases.

**SQLite Configuration:**

- SQLite databases are single files.
- Create a new SQLite database with `touch database/database.sqlite`.
- Set the `DB_DATABASE` environment variable to the database's absolute path.

```env
DB_CONNECTION=sqlite
DB_DATABASE=/absolute/path/to/database.sqlite
```

- Enable foreign key constraints by setting `DB_FOREIGN_KEYS` to `true`.

```env
DB_FOREIGN_KEYS=true
```

**Microsoft SQL Server Configuration:**

- Install `sqlsrv` and `pdo_sqlsrv` enable PHP extensions.
- Install any required dependencies like the Microsoft SQL ODBC driver.

**Configuration Using URLs:**

- Database connections use host, database, username, and password.
- Each has a corresponding environment variable.
- Production servers require managing multiple environment variables.
- Some providers like AWS and Heroku offer a single database URL.
- Example URL: `mysql://root:[email protected]/forge?charset=UTF-8`.
- URL schema: `driver://username:password@host:port/database?options`.
- Laravel supports URLs as an alternative to multiple configuration options.
- If the `url` or `DATABASE_URL` is present, it will be used.

### &#10022; Read & Write Connections:

- Separate database connections can be used for `SELECT` and `INSERT/UPDATE/DELETE` statements.
- Laravel handles connection selection for raw queries, query builder, and Eloquent ORM.
- Configuration example:

```php
'mysql' => [
    'read' => [
        'host' => [
            '192.168.1.1',
            '196.168.1.2',
        ],
    ],
    'write' => [
        'host' => [
            '196.168.1.3',
        ],
    ],
    'sticky' => true,
    'driver' => 'mysql',
    'database' => 'database_name',
    'username' => 'root',
    'password' => '',
    'charset' => 'utf8mb4',
    'collation' => 'utf8mb4_unicode_ci',
    'prefix' => '',
],
```

- `read`, `write`, and `sticky` keys are added to the configuration array.
- `read` and `write` contain `host` arrays.
- Other database options are merged from the main `mysql` array.
- `read` and `write` arrays override values from the main `mysql` array.
- Multiple hosts in the `host` array result in random host selection per request.
- **The `sticky` Option:**
    - `sticky` allows immediate reading of written records in the current request.
    - If `sticky` is enabled and a "write" operation occurs, subsequent "read" operations use the "write" connection.
    - This ensures data written in the request can be immediately read.
    - Deciding to use this behavior is up to the application's needs.

### &#10022; Running SQL Queries:

- Laravel allows query execution via the `DB` facade.
- The `DB` facade provides methods for `select`, `update`, `insert`, `delete`, and `statement`.

**Running A Select Query:**

- Use `DB::select()` to execute a `SELECT` query.
- The first argument is the SQL query.
- The second argument is the parameter bindings.
- Parameter bindings protect against SQL injection.
- The method returns an array of `stdClass` objects.

```php
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\DB;

class UserController extends Controller
{
    public function index()
    {
        $users = DB::select('select * from users where active = ?', [1]);

        return view('user.index', ['users' => $users]);
    }
}

use Illuminate\Support\Facades\DB;

$users = DB::select('select * from users');

foreach ($users as $user) {
    echo $user->name;
}
```

**Using Named Bindings:**

- Use named bindings instead of positional bindings.

```php
$results = DB::select('select * from users where id = :id', ['id' => 1]);
```

**Running An Insert Statement:**

- Use `DB::insert()` to execute an `INSERT` query.

```php
use Illuminate\Support\Facades\DB;

DB::insert('insert into users (id, name) values (?, ?)', [1, 'kumar']);
```

**Running An Update Statement:**

- Use `DB::update()` to execute an `UPDATE` query.
- The method returns the number of affected rows.

```php
use Illuminate\Support\Facades\DB;

$affected = DB::update('update users set votes = 100 where name = ?', ['kumar']);
```

**Running A Delete Statement:**

- Use `DB::delete()` to execute a `DELETE` query.
- The method returns the number of affected rows.

```php
use Illuminate\Support\Facades\DB;

$deleted = DB::delete('delete from users');
```

**Running A General Statement:**

- Use `DB::statement()` for queries without return values.

```php
DB::statement('drop table users');
```

**Running An Unprepared Statement:**

- Use `DB::unprepared()` to execute SQL without bindings.
- This is vulnerable to SQL injection.

```php
DB::unprepared('update users set votes = 100 where name = "kumar"');
```

**Implicit Commits:**

- `DB::statement()` and `DB::unprepared()` in transactions may cause implicit commits.
- Avoid statements that cause implicit commits, such as `CREATE TABLE`.

```php
DB::unprepared('create table a (col varchar(1) null)');
```

### &#10022; Using Multiple Database Connections:

- Applications can define multiple connections in `config/database.php`.
- Access each connection using the `connection` method of the `DB` facade.
- The connection name should match a configuration entry.

```php
use Illuminate\Support\Facades\DB;

$users = DB::connection('sqlite')->select(...);
```

- Access the underlying PDO instance using `getPdo()`.

```php
$pdo = DB::connection()->getPdo();
```

### &#10022; Listening For Query Events:

- A closure can be specified to execute for each SQL query.
- The `DB::listen()` method registers this closure.
- Useful for logging or debugging queries.
- Register the listener in the `boot()` method of a service provider.

```php
namespace App\Providers;

use Illuminate\Support\Facades\DB;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register()
    {
        //
    }

    public function boot()
    {
        DB::listen(function ($query) {
            // $query->sql;
            // $query->bindings;
            // $query->time;
        });
    }
}
```

### &#10022; Database Transactions:

- Use `DB::transaction()` to execute operations within a transaction.
- Exceptions within the closure trigger automatic rollbacks.
- Successful execution commits the transaction.

```php
use Illuminate\Support\Facades\DB;

DB::transaction(function () {
    DB::update('update users set votes = 1');
    DB::delete('delete from posts');
});
```

**Handling Deadlocks:**

- `DB::transaction()` accepts an optional second argument.
- This argument defines the number of retries for deadlocks.
- An exception is thrown after retries are exhausted.

```php
use Illuminate\Support\Facades\DB;

DB::transaction(function () {
    DB::update('update users set votes = 1');
    DB::delete('delete from posts');
}, 5);
```

**Manually Using Transactions:**

- Use `DB::beginTransaction()` for manual transaction control.

```php
use Illuminate\Support\Facades\DB;

DB::beginTransaction();
```

- Rollback with `DB::rollBack()`.

```php
DB::rollBack();
```

- Commit with `DB::commit()`.

```php
DB::commit();
```

- The `DB` facade's transaction methods control transactions for both the query builder and Eloquent ORM.

### &#10022; Connecting To The Database CLI:

- Use the `db` Artisan command to connect to the database CLI.

```bash
php artisan db
```

- Specify a connection name to connect to a non-default connection.

```bash
php artisan db mysql
```

---
[&#8682; To Top](#-database)

[&#10094; Previous Topic](./logging.md) &emsp; [Next Topic &#10095;](./migrations.md)

[&#8962; Goto Home Page](../README.md)