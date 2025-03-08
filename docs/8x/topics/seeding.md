## &#10162; Seeding:

-   Laravel allows seeding databases with data using seed classes.
-   Seed classes are stored in the `database/seeders` directory.
-   A default `DatabaseSeeder` class is provided.
-   Use the `call` method in `DatabaseSeeder` to run other seed classes and control seeding order.
-   Mass assignment protection is disabled during database seeding.

### &#9780; Overview:

1. [Writing Seeders](#-writing-seeders)
    - [Using Model Factories](#-using-model-factories)
    - [Calling Additional Seeders](#-calling-additional-seeders)
2. [Running Seeders](#-running-seeders)

### &#10022; Writing Seeders:

-   Use the `make:seeder` Artisan command to generate seeders.
    ```bash
    php artisan make:seeder UserSeeder
    ```
-   Generated seeders are placed in the `database/seeders` directory.
-   Seeder classes contain a `run` method.
-   The `run` method executes when the `db:seed` Artisan command is called.
-   Insert data into the database using the query builder or Eloquent model factories.

**Example Seeder:**

```php
namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;

class DatabaseSeeder extends Seeder
{
    public function run()
    {
        DB::table('users')->insert([
            'name' => Str::random(10),
            'email' => Str::random(10).'@gmail.com',
            'password' => Hash::make('password'),
        ]);
    }
}
```

-   Type-hint dependencies in the `run` method signature for automatic resolution via the Laravel service container.

### &#10022; Using Model Factories:

-   Manually specifying attributes for each model seed is cumbersome.
-   Model factories generate large amounts of database records conveniently.
-   Review model factory documentation to define factories.

*Example:*

```php
use App\Models\User;

/**
 * Run the database seeders.
 *
 * @return void
 */
public function run()
{
    User::factory()
        ->count(50)
        ->hasPosts(1)
        ->create();
}
```

-   This example creates 50 users, each with one related post.

### &#10022; Calling Additional Seeders:

-   Use the `call` method in `DatabaseSeeder` to execute other seed classes.
-   This breaks up database seeding into multiple files for better organization.
-   The `call` method accepts an array of seeder classes to execute.

*Example:*

```php
/**
 * Run the database seeders.
 *
 * @return void
 */
public function run()
{
    $this->call([
        UserSeeder::class,
        PostSeeder::class,
        CommentSeeder::class,
    ]);
}
```

### &#10022; Running Seeders:

-   Use the `db:seed` Artisan command to seed the database.
-   By default, it runs `Database\Seeders\DatabaseSeeder`.
-   Use the `--class` option to run a specific seeder.
    ```bash
    php artisan db:seed
    php artisan db:seed --class=UserSeeder
    ```
-   Seed during migration refresh using `migrate:fresh` and `--seed`.
    ```bash
    php artisan migrate:fresh --seed
    ```

**Forcing Seeders To Run In Production:**

-   Seeding operations may alter or lose data.
-   Confirmation is prompted in production.
-   Use the `--force` flag to run seeders without a prompt.
    ```bash
    php artisan db:seed --force
    ```

---
[&#8682; To Top](#-seeding)

[&#10094; Previous Topic](./migrations.md) &emsp; [Next Topic &#10095;](./query-builder.md)

[&#8962; Goto Home Page](../README.md)