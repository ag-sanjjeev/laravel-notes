## &#10162; Query Builder:

- Laravel query builder performs most database operations.
- It is compatible with all of Laravel supported databases.
- The query builder uses PDO parameter binding.
- This protects against SQL injection.
- Sanitization of query bindings is not needed.
- PDO does not support binding column names.
- User input should not determine column names, including `order by` columns.

### &#9780; Overview:

1. [Running Database Queries](#-running-database-queries)
    - [Chunking Results](#-chunking-results)
    - [Streaming Results Lazily](#-streaming-results-lazily)
    - [Aggregates](#-aggregates)
2. [Select Statements](#-select-statements)
3. [Raw Expressions](#-raw-expressions)
4. [Joins](#-joins)
5. [Unions](#-unions)
6. [Where Clauses](#-where-clauses)
    - [Or Where Clauses](#-or-where-clauses)
    - [JSON Where Clauses](#-json-where-clauses)
    - [Additional Where Clauses](#-additional-where-clauses)
    - [Logical Grouping](#-logical-grouping)
7. [Advanced Where Clauses](#-advanced-where-clauses)
    - [Where Exists Clauses](#-where-exists-clauses)
8. [Ordering](#-ordering)
9. [Grouping](#-grouping)
10. [Limit and Offset](#-limit-and-offset)
11. [Conditional Clauses](#-conditional-clauses)
12. [Insert Statements](#-insert-statements)
    - [Upserts](#-upserts)
13. [Update Statements](#-update-statements)
    - [Updating JSON Columns](#-updating-json-columns)
    - [Increment and Decrement](#-increment-and-decrement)
14. [Delete Statements](#-delete-statements)
15. [Pessimistic Locking](#-pessimistic-locking)
16. [Debugging](#-debugging)

### &#10022; Running Database Queries:

**Retrieving All Rows From A Table:**

- Use `DB::table()` to start a query.
- `table()` returns a query builder instance.
- Chain constraints and use `get()` to retrieve results.
- `get()` returns an `Illuminate\Support\Collection` of `stdClass` objects.

```php
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\DB;

class UserController extends Controller
{
    public function index()
    {
        $users = DB::table('users')->get();

        return view('user.index', ['users' => $users]);
    }
}

use Illuminate\Support\Facades\DB;

$users = DB::table('users')->get();

foreach ($users as $user) {
    echo $user->name;
}
```

**Retrieving A Single Row / Column From A Table:**

- Use `first()` to retrieve a single `stdClass` object.

```php
$user = DB::table('users')->where('name', 'kumar')->first();

return $user->email;
```

- Use `value()` to extract a single column value.

```php
$email = DB::table('users')->where('name', 'kumar')->value('email');
```

- Use `find()` to retrieve a row by its `id`.

```php
$user = DB::table('users')->find(3);
```

**Retrieving A List Of Column Values:**

- Use `pluck()` to retrieve a collection of single column values.

```php
use Illuminate\Support\Facades\DB;

$products = DB::table('products')->pluck('product_name');

foreach ($products as $product) {
    echo $product;
}
```

- Provide a second argument to `pluck()` to use a different column as keys.

```php
$products = DB::table('products')->pluck('product_name', 'name');

foreach ($products as $name => $product) {
    echo $product;
}
```

### &#10022; Chunking Results:

- Use `chunk()` to process large datasets in smaller chunks.
- `chunk()` retrieves a limited number of records at a time.
- Each chunk is passed to a closure for processing.

```php
use Illuminate\Support\Facades\DB;

DB::table('users')->orderBy('id')->chunk(100, function ($users) {
    foreach ($users as $user) {
        //
    }
});
```

- Return `false` from the closure to stop further chunk processing.

```php
DB::table('users')->orderBy('id')->chunk(100, function ($users) {
    // Process the records...

    return false;
});
```

- Use `chunkById()` for safer updates during chunking.
- `chunkById()` paginates results based on the primary key.

```php
DB::table('users')->where('active', false)
    ->chunkById(100, function ($users) {
        foreach ($users as $user) {
            DB::table('users')
                ->where('id', $user->id)
                ->update(['active' => true]);
        }
    });
```

- Modifying primary or foreign keys during chunk callbacks can affect results.
- This may lead to records being skipped in chunked results.

### &#10022; Streaming Results Lazily:

- The `lazy()` method executes queries in chunks, similar to `chunk()`.
- `lazy()` returns a `LazyCollection` for stream-like result interaction.

```php
use Illuminate\Support\Facades\DB;

DB::table('users')->orderBy('id')->lazy()->each(function ($user) {
    //
});
```

- Use `lazyById()` or `lazyByIdDesc()` for safer updates during iteration.
- These methods paginate based on the primary key.

```php
DB::table('users')->where('active', false)
    ->lazyById()->each(function ($user) {
        DB::table('users')
            ->where('id', $user->id)
            ->update(['active' => true]);
    });
```

- Modifying primary or foreign keys during iteration can affect results.
- This may cause records to be excluded from the results.

### &#10022; Aggregates:

- The query builder provides aggregate methods like `count`, `max`, `min`, `avg`, and `sum`.
- These methods retrieve aggregate values.

```php
use Illuminate\Support\Facades\DB;

$users = DB::table('users')->count();

$price = DB::table('orders')->max('price');
```

- Aggregate methods can be combined with other clauses.

```php
$price = DB::table('orders')
    ->where('finalized', 1)
    ->sum('price');
```

**Determining If Records Exist:**

- Use `exists()` and `doesntExist()` to check for record existence.
- These methods are more efficient than `count()`.

```php
if (DB::table('orders')->where('finalized', 1)->exists()) {
    // ...
}

if (DB::table('orders')->where('finalized', 1)->doesntExist()) {
    // ...
}
```

### &#10022; Select Statements:

**Specifying A Select Clause:**

- Use `select()` to specify custom columns.

```php
use Illuminate\Support\Facades\DB;

$users = DB::table('users')
    ->select('name', 'email as user_email')
    ->get();
```

- Use `distinct()` to get unique results.

```php
$users = DB::table('users')->distinct()->get();
```

- Use `addSelect()` to add columns to an existing select clause.

```php
$query = DB::table('users')->select('name');

$users = $query->addSelect('age')->get();
```

### &#10022; Raw Expressions:

- Use `DB::raw()` to insert arbitrary strings into queries.
- Raw statements are injected as strings, posing SQL injection risks.

```php
$users = DB::table('users')
    ->select(DB::raw('count(*) as user_count, status'))
    ->where('status', '<>', 1)
    ->groupBy('status')
    ->get();
```

**Raw Methods:**

- Raw methods insert raw expressions into various query parts.
- Laravel does not guarantee SQL injection protection for raw expressions.

**`selectRaw`:**

- Alternative to `addSelect(DB::raw(...))`.
- Accepts an optional bindings array.

```php
$orders = DB::table('orders')
    ->selectRaw('price * ? as price_with_tax', [1.0825])
    ->get();
```

**`whereRaw` / `orWhereRaw`:**

- Injects raw "where" clauses.
- Accepts an optional bindings array.

```php
$orders = DB::table('orders')
    ->whereRaw('price > IF(state = "TX", ?, 100)', [200])
    ->get();
```

**`havingRaw` / `orHavingRaw`:**

- Provides raw "having" clause values.
- Accepts an optional bindings array.

```php
$orders = DB::table('orders')
    ->select('department', DB::raw('SUM(price) as total_sales'))
    ->groupBy('department')
    ->havingRaw('SUM(price) > ?', [2500])
    ->get();
```

**`orderByRaw`:**

- Provides raw "order by" clause values.

```php
$orders = DB::table('orders')
    ->orderByRaw('updated_at - created_at DESC')
    ->get();
```

**`groupByRaw`:**

- Provides raw "group by" clause values.

```php
$orders = DB::table('orders')
    ->select('city', 'state')
    ->groupByRaw('city, state')
    ->get();
```

### &#10022; Joins:

-   The query building tool allows for the insertion of join statements.
-   The `join` action executes a fundamental internal linking of tables.
-   The `join` action requires the name of the table and the restrictions on the columns.
-   A single query can combine multiple tables, as below:
    ```php
    use Illuminate\Support\Facades\DB;
    $users = DB::table('users')
        ->join('contacts', 'users.id', '=', 'contacts.user_id')
        ->join('orders', 'users.id', '=', 'orders.user_id')
        ->select('users.*', 'contacts.phone', 'orders.price')
        ->get();
    ```

**Left Join / Right Join Clause:**

-   `leftJoin` or `rightJoin` actions perform either a left or right table linking.
-   These actions utilize the same input values as the `join` action, for instance:
    ```php
    $users = DB::table('users')
        ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
        ->get();

    $users = DB::table('users')
        ->rightJoin('posts', 'users.id', '=', 'posts.user_id')
        ->get();
    ```

**Cross Join Clause:**

-   The `crossJoin` action carries out a complete combination of all rows.
-   Complete combinations produce every possible row pairing, such as:
    ```php
    $sizes = DB::table('sizes')
        ->crossJoin('colors')
        ->get();
    ```

**Advanced Join Clauses:**

-   Complex join statements are defined through a function within a function.
-   The internal function receives a `Illuminate\Database\Query\JoinClause` object.
-   `JoinClause` enables the definition of limitations, for example:
    ```php
    DB::table('users')
        ->join('contacts', function ($join) {
            $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
        })
        ->get();
    ```
-   `where` and `orWhere` functions, provided by `JoinClause`, introduce conditional statements.
-   These functions compare a column to a value, as depicted:
    ```php
    DB::table('users')
        ->join('contacts', function ($join) {
            $join->on('users.id', '=', 'contacts.user_id')
                ->where('contacts.user_id', '>', 5);
        })
        ->get();
    ```

**Subquery Joins:**

-   `joinSub`, `leftJoinSub`, and `rightJoinSub` functions link a query to a query within a query.
-   These functions accept the inner query, its alias, and an inner function.
-   The inner function defines the connected columns, as in the following instance:
    ```php
    $latestPosts = DB::table('posts')
        ->select('user_id', DB::raw('MAX(created_at) as last_post_created_at'))
        ->where('is_published', true)
        ->groupBy('user_id');

    $users = DB::table('users')
        ->joinSub($latestPosts, 'latest_posts', function ($join) {
            $join->on('users.id', '=', 'latest_posts.user_id');
        })
        ->get();
    ```

### &#10022; Unions:

-   The query builder offers an easy way to combine two or more queries.
-   An initial query can be created and then combined with other queries using the `union` method.
-   An example of combining queries:
    ```php
    use Illuminate\Support\Facades\DB;

    $first = DB::table('users')
        ->whereNull('first_name');

    $users = DB::table('users')
        ->whereNull('last_name')
        ->union($first)
        ->get();
    ```
-   The query builder also provides a `unionAll` method.
-   Queries combined with `unionAll` will keep duplicate results.
-   The `unionAll` method uses the same inputs as the `union` method.


### &#10022; Where Clauses:

-   The query builder's `where` function adds conditional clauses.
-   The `where` function needs three inputs: the column name, an operator, and a comparison value.
-   An example query:
    ```php
    $users = DB::table('users')
        ->where('votes', '=', 100)
        ->where('age', '>', 35)
        ->get();
    ```
-   When checking for equality, the second argument can be the value, assuming `=` as the operator.
    ```php
    $users = DB::table('users')->where('votes', 100)->get();
    ```
-   Any database-supported operator can be used:
    ```php
    $users = DB::table('users')
        ->where('votes', '>=', 100)
        ->get();

    $users = DB::table('users')
        ->where('votes', '<>', 100)
        ->get();

    $users = DB::table('users')
        ->where('name', 'like', 'T%')
        ->get();
    ```
-   An array of conditions can be passed to the `where` function, with each condition as an array.
    ```php
    $users = DB::table('users')->where([
        ['status', '=', '1'],
        ['subscribed', '<>', '1'],
    ])->get();
    ```
-   Column names should not be dictated by user input due to PDO limitations.

### &#10022; Or Where Clauses:

-   Chained `where` function calls are combined with the `and` operator.
-   The `orWhere` function combines a clause with the `or` operator.
-   `orWhere` uses the same inputs as `where`.
    ```php
    $users = DB::table('users')
        ->where('votes', '>', 100)
        ->orWhere('name', 'Kumar')
        ->get();
    ```
-   To group an `or` condition in parentheses, pass a closure to `orWhere`.
    ```php
    $users = DB::table('users')
        ->where('votes', '>', 100)
        ->orWhere(function($query) {
            $query->where('name', 'kumar')
                ->where('votes', '>', 50);
        })->get();
    ```
-   The above example produces SQL like: `select * from users where votes > 100 or (name = 'kumar' and votes > 50)`
-   Group `orWhere` calls to prevent unexpected behavior with global scopes.

### &#10022; JSON Where Clauses:

-   Laravel supports querying JSON columns on databases with JSON support.
-   Supported databases: MySQL 5.7+, PostgreSQL, SQL Server 2016, and SQLite 3.9.0 (with JSON1 extension).
-   The `->` operator is used to query a JSON column.
    ```php
    $users = DB::table('users')
        ->where('preferences->dining->meal', 'rice')
        ->get();
    ```
-   `whereJsonContains` queries JSON arrays. SQLite does not support this feature.
    ```php
    $users = DB::table('users')
        ->whereJsonContains('options->languages', 'en')
        ->get();
    ```
-   MySQL and PostgreSQL allow passing an array of values to `whereJsonContains`.
    ```php
    $users = DB::table('users')
        ->whereJsonContains('options->languages', ['en', 'ta'])
        ->get();
    ```
-   `whereJsonLength` queries JSON arrays by their length.
    ```php
    $users = DB::table('users')
        ->whereJsonLength('options->languages', 0)
        ->get();

    $users = DB::table('users')
        ->whereJsonLength('options->languages', '>', 1)
        ->get();
    ```

### &#10022; Additional Where Clauses:

-   `whereBetween` checks if a column's value is within two values.
    ```php
    $users = DB::table('users')
        ->whereBetween('votes', [1, 100])
        ->get();
    ```
-   `whereNotBetween` verifies a column's value is outside two values.
    ```php
    $users = DB::table('users')
        ->whereNotBetween('votes', [1, 100])
        ->get();
    ```
-   `whereIn` checks if a column's value is in a given array.
    ```php
    $users = DB::table('users')
        ->whereIn('id', [1, 2, 3])
        ->get();
    ```
-   `whereNotIn` checks if a column's value is not in a given array.
    ```php
    $users = DB::table('users')
        ->whereNotIn('id', [1, 2, 3])
        ->get();
    ```
-   `whereIntegerInRaw` or `whereIntegerNotInRaw` reduce memory usage with large integer array bindings.
-   `whereNull` checks if a column's value is NULL.
    ```php
    $users = DB::table('users')
        ->whereNull('updated_at')
        ->get();
    ```
-   `whereNotNull` checks if a column's value is not NULL.
    ```php
    $users = DB::table('users')
        ->whereNotNull('updated_at')
        ->get();
    ```
-   `whereDate` compares a column's value against a date.
    ```php
    $users = DB::table('users')
        ->whereDate('created_at', '2025-02-08')
        ->get();
    ```
-   `whereMonth` compares a column's value against a month.
    ```php
    $users = DB::table('users')
        ->whereMonth('created_at', '2')
        ->get();
    ```
-   `whereDay` compares a column's value against a day.
    ```php
    $users = DB::table('users')
        ->whereDay('created_at', '8')
        ->get();
    ```
-   `whereYear` compares a column's value against a year.
    ```php
    $users = DB::table('users')
        ->whereYear('created_at', '2025')
        ->get();
    ```
-   `whereTime` compares a column's value against a time.
    ```php
    $users = DB::table('users')
        ->whereTime('created_at', '=', '12:25:10')
        ->get();
    ```
-   `whereColumn` checks if two columns are equal.
    ```php
    $users = DB::table('users')
        ->whereColumn('first_name', 'last_name')
        ->get();
    ```
-   `whereColumn` accepts a comparison operator.
    ```php
    $users = DB::table('users')
        ->whereColumn('updated_at', '>', 'created_at')
        ->get();
    ```
-   `whereColumn` accepts an array of column comparisons, combined with `and`.
    ```php
    $users = DB::table('users')
        ->whereColumn([
            ['first_name', '=', 'last_name'],
            ['updated_at', '>', 'created_at'],
        ])->get();
    ```

### &#10022; Logical Grouping:

-   Grouping `where` clauses in parentheses is needed for complex logical conditions.
-   Grouping `orWhere` calls avoids unexpected query behavior.
-   A closure passed to the `where` method creates a constraint group.
    ```php
    $users = DB::table('users')
        ->where('name', '=', 'kumar')
        ->where(function ($query) {
            $query->where('age', '>', 18)
                ->orWhere('title', '=', 'Admin');
        })->get();
    ```
-   The closure receives a query builder instance for setting constraints within the group.
-   The example above produces SQL: `select * from users where name = 'kumar' and (age > 18 or title = 'Admin')`
-   Always group `orWhere` calls to prevent issues with global scopes.

### &#10022; Advanced Where Clauses:

-   `whereExists` method allows for "where exists" SQL clauses.
-   `whereExists` accepts a closure with a query builder instance.
-   The closure defines the query within the "exists" clause.
    ```php
    $users = DB::table('users')
        ->whereExists(function ($query) {
            $query->select(DB::raw(1))
                ->from('orders')
                ->whereColumn('orders.user_id', 'users.id');
        })->get();
    ```
-   The above query produces SQL:
    ```sql
    select * from users
    where exists (
        select 1
        from orders
        where orders.user_id = users.id
    )
    ```

### &#10022; Where Exists Clauses:

**Subquery Where Clauses:**

-   A "where" clause can compare subquery results to a value.
-   Pass a closure and value to the `where` method.
-   Example: Get users with a recent "Pro" membership.
    ```php
    use App\Models\User;

    $users = User::where(function ($query) {
        $query->select('type')
            ->from('membership')
            ->whereColumn('membership.user_id', 'users.id')
            ->orderByDesc('membership.start_date')
            ->limit(1);
    }, 'Pro')->get();
    ```
-   A "where" clause can compare a column to subquery results.
-   Pass a column, operator, and closure to the `where` method.
-   Example: Get income records where amount is less than average.
    ```php
    use App\Models\Income;

    $incomes = Income::where('amount', '<', function ($query) {
        $query->selectRaw('avg(i.amount)')->from('incomes as i');
    })->get();
    ```

### &#10022; Ordering:

-   The `orderBy` method sorts query results by a column.
-   The first `orderBy` argument is the column, the second is `asc` or `desc`.
    ```php
    $users = DB::table('users')
        ->orderBy('name', 'desc')
        ->get();
    ```
-   Multiple columns can be ordered by calling `orderBy` multiple times.
    ```php
    $users = DB::table('users')
        ->orderBy('name', 'desc')
        ->orderBy('email', 'asc')
        ->get();
    ```
-   `latest` and `oldest` methods order by date, defaulting to `created_at`.
    ```php
    $user = DB::table('users')
        ->latest()
        ->first();
    ```
-   `inRandomOrder` method sorts query results randomly.
    ```php
    $randomUser = DB::table('users')
        ->inRandomOrder()
        ->first();
    ```
-   `reorder` method removes existing "order by" clauses.
    ```php
    $query = DB::table('users')->orderBy('name');
    $unorderedUsers = $query->reorder()->get();
    ```
-   `reorder` can apply a new order by passing a column and direction.
    ```php
    $query = DB::table('users')->orderBy('name');
    $usersOrderedByEmail = $query->reorder('email', 'desc')->get();
    ```

### &#10022; Grouping:

-   `groupBy` and `having` methods group query results.
-   `having` method's inputs are similar to the `where` method.
    ```php
    $users = DB::table('users')
        ->groupBy('account_id')
        ->having('account_id', '>', 100)
        ->get();
    ```
-   `havingBetween` filters results within a specified range.
    ```php
    $report = DB::table('orders')
        ->selectRaw('count(id) as number_of_orders, customer_id')
        ->groupBy('customer_id')
        ->havingBetween('number_of_orders', [15, 25])
        ->get();
    ```
-   Multiple columns can be grouped by passing multiple arguments to `groupBy`.
    ```php
    $users = DB::table('users')
        ->groupBy('first_name', 'status')
        ->having('account_id', '>', 100)
        ->get();
    ```
-   `havingRaw` builds more complex `having` statements.

### &#10022; Limit and Offset:

-   `skip` and `take` methods limit result count or skip results.
    ```php
    $users = DB::table('users')->skip(10)->take(5)->get();
    ```
-   `limit` and `offset` methods are functionally equivalent to `take` and `skip`.
    ```php
    $users = DB::table('users')
        ->offset(10)
        ->limit(5)
        ->get();
    ```

### &#10022; Conditional Clauses:

-   Conditional query clauses apply based on other conditions.
-   The `when` method applies a clause if a condition is true.
-   Example: Apply a `where` clause if a role input is present.
    ```php
    $role = $request->input('role');

    $users = DB::table('users')
        ->when($role, function ($query, $role) {
            return $query->where('role_id', $role);
        })->get();
    ```
-   `when` executes the closure only if the first argument is true.
-   A third closure to `when` executes if the first argument is false.
-   Example: Set default ordering based on a condition.
    ```php
    $sortByAge = $request->input('sort_by_votes');

    $users = DB::table('users')
        ->when($sortByAge, function ($query, $sortByAge) {
            return $query->orderBy('age');
        }, function ($query) {
            return $query->orderBy('name');
        })->get();
    ```

### &#10022; Insert Statements:

-   The `insert` method adds records to a database table.
-   `insert` accepts an array of column names and values.
    ```php
    DB::table('users')->insert([
        'email' => '[email protected]',
        'age' => 19
    ]);
    ```
-   Multiple records can be inserted by passing an array of arrays.
    ```php
    DB::table('users')->insert([
        ['email' => '[email protected]', 'age' => 21],
        ['email' => '[email protected]', 'age' => 31],
    ]);
    ```
-   `insertOrIgnore` ignores errors during record insertion.
    ```php
    DB::table('users')->insertOrIgnore([
        ['id' => 1, 'email' => '[email protected]'],
        ['id' => 2, 'email' => '[email protected]'],
    ]);
    ```
-   `insertOrIgnore` ignores duplicate records and other database engine specific errors.
-   `insertGetId` inserts a record and retrieves the auto-incrementing ID.
    ```php
    $id = DB::table('users')->insertGetId(
        ['email' => '[email protected]', 'age' => 21]
    );
    ```
-   PostgreSQL's `insertGetId` expects the auto-incrementing column to be named `id`.
-   A different sequence column name can be passed as the second parameter to `insertGetId` in PostgreSQL.

### &#10022; Upserts:

-   The `upsert` method inserts new records or updates existing ones.
-   The first `upsert` argument holds the values for insert or update.
-   The second `upsert` argument lists columns that uniquely identify records.
-   The third `upsert` argument holds columns to update if a record exists.
    ```php
    DB::table('orders')->upsert([
        ['product_name' => 'Cake', 'category' => 'food', 'price' => 120],
        ['product_name' => 'Chocolate', 'category' => 'food', 'price' => 70],
    ], ['product_name', 'category'], ['price']);
    ```
-   The example attempts to insert two records, updating price if `product_name` and `category` match existing records.
-   All databases except SQL Server require `primary` or `unique` indexes for the second argument columns.
-   MySQL ignores the second argument and uses `primary` and `unique` indexes.

### &#10022; Update Statements:

-   The `update` method modifies existing database records.
-   `update` accepts an array of column and value pairs to update.
-   The `update` method returns the number of affected rows.
-   `where` clauses can constrain the update query.
    ```php
    $affected = DB::table('users')
        ->where('id', 1)
        ->update(['votes' => 1]);
    ```

**Update Or Insert:**

-   The `updateOrInsert` method updates a record or creates it if it doesn't exist.
-   `updateOrInsert` takes two arguments: conditions to find the record, and values to update.
-   If a record matches the conditions, it's updated.
-   If no record matches, a new record is inserted with merged attributes.
    ```php
    DB::table('products')
        ->updateOrInsert(
            ['product_name' => 'Cake', 'category' => 'food'],
            ['price' => 150]
        );
    ```

### &#10022; Updating JSON Columns:

-   Use `->` arrow syntax to update keys in JSON objects when updating JSON columns.
-   This operation is supported on MySQL 5.7+ and PostgreSQL 9.5+.
    ```php
    $affected = DB::table('users')
        ->where('id', 1)
        ->update(['theme->dark' => true]);
    ```

### &#10022; Increment and Decrement:

-   The query builder provides methods to increment or decrement column values.
-   Both methods require at least one argument: the column to modify.
-   A second argument specifies the increment or decrement amount.
    ```php
    DB::table('products')->where('id', 1)->increment('price');
    DB::table('products')->where('id', 3)->increment('price', 25);
    DB::table('products')->where('id', 1)->decrement('price');
    DB::table('products')->where('id', 3)->decrement('price', 25);
    ```
-   Additional columns can be updated during the increment or decrement.
    ```php
    DB::table('products')->where('id', 1)->increment('price', 100, ['dealer' => 'kumar']);
    ```

### &#10022; Delete Statements:

-   The query builder's `delete` method removes records from a table.
-   `delete` returns the number of affected rows.
-   `where` clauses can constrain delete statements.
    ```php
    $deleted = DB::table('products')->delete();
    $deleted = DB::table('products')->where('price', '<', 100)->delete();
    ```
-   `truncate` removes all records and resets the auto-incrementing ID to zero.
    ```php
    DB::table('products')->truncate();
    ```
-   PostgreSQL's `truncate` applies `CASCADE` behavior, deleting foreign key related records.

### &#10022; Pessimistic Locking:

-   The query builder supports `pessimistic locking` for select statements.
-   `sharedLock` executes a statement with a `shared lock`.
-   A shared lock prevents selected rows from modification until transaction commit.
    ```php
    DB::table('posts')
        ->where('id', 100)
        ->sharedLock()
        ->get();
    ```
-   `lockForUpdate` executes a statement with a `for update` lock.
-   A `for update` lock prevents selected records from modification or shared lock selection.
    ```php
    DB::table('posts')
        ->where('id', 100)
        ->lockForUpdate()
        ->get();
    ```

### &#10022; Debugging:

You may use the `dd` and `dump` methods while building a query to dump the current query bindings and SQL. The `dd` method will display the debug information and then stop executing the request. The `dump` method will display the debug information but allow the request to continue executing.

DB::table('users')->where('id', '>', 100)->dd();
 
DB::table('users')->where('id', '>', 100)->dump();

---
[&#8682; To Top](#-query-builder)

[&#10094; Previous Topic](./seeding.md) &emsp; [Next Topic &#10095;](./pagination.md)

[&#8962; Goto Home Page](../README.md)