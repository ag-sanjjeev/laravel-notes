## &#10162; Pagination:

-   Laravel's pagination aims to simplify the process compared to other frameworks.
-   The paginator integrates with the query builder and Eloquent ORM.
-   Laravel's paginator provides easy pagination of database records with no configuration.

### &#9780; Overview:

1. [Basic Usage](#-basic-usage)
    - [Paginating Query Builder Results](#-paginating-query-builder-results)
    - [Cursor Pagination](#-cursor-pagination)
    - [Manually Creating A Paginator](#-manually-creating-a-paginator)
    - [Customizing Pagination URLs](#-customizing-pagination-urls)
2. [Displaying Pagination Results](#-displaying-pagination-results)
    - [Adjusting The Pagination Link Window](#-adjusting-the-pagination-link-window)
    - [Converting Results To JSON](#-converting-results-to-json)
3. [Customizing The Pagination View](#-customizing-the-pagination-view)
    - [Using Bootstrap](#-using-bootstrap)
4. [Paginator and LengthAwarePaginator Instance Methods](#-paginator-and-lengthawarepaginator-instance-methods)
5. [Cursor Paginator Instance Methods](#-cursor-paginator-instance-methods)

### &#10022; Basic Usage:

-   By default, paginator HTML is compatible with Tailwind CSS.
-   Bootstrap pagination support is also available.
-   When using Tailwind JIT with Laravel's default pagination views, update `tailwind.config.js` to prevent class purging.
    ```javascript
    content: [
        './resources/**/*.blade.php',
        './resources/**/*.js',
        './resources/**/*.vue',
        './vendor/laravel/framework/src/Illuminate/Pagination/resources/views/*.blade.php',
    ],
    ```

**Paginating Query Builder Results:**

-   The `paginate` method paginates items from the query builder or Eloquent query.
-   `paginate` sets the query's `limit` and `offset` based on the current page.
-   The current page is detected by the `page` query string argument.
-   The `paginate` method's argument is the number of items to display `per page`.
-   Example: Display 10 products per page.
    ```php
    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\DB;

    class ProductController extends Controller
    {
        public function index()
        {
            return view('product.index', [
                'products' => DB::table('products')->paginate(10)
            ]);
        }
    }
    ```

**Simple Pagination:**

-   The `paginate` method counts total records matched by the query.
-   This count is used to determine total pages.
-   If total pages are not needed, the count query is unnecessary.
-   The `simplePaginate` method performs a single, efficient query.
-   `simplePaginate` is used when only `Next` and `Previous` links are needed.
    ```php
    $products = DB::table('products')->simplePaginate(10);
    ```

### &#10022; Paginating Query Builder Results:

**Paginating Eloquent Results:**

-   Eloquent queries can be paginated.
-   Example: Paginate `App\Models\Products` with 10 records per page.
    ```php
    use App\Models\Products;

    $products = Products::paginate(10);
    ```
-   `paginate` can be called after setting query constraints like `where` clauses.
    ```php
    $products = Products::where('price', '>', 100)->paginate(10);
    ```
-   `simplePaginate` can be used with Eloquent models.
    ```php
    $products = Products::where('price', '>', 100)->simplePaginate(10);
    ```
-   `cursorPaginate` can be used for cursor pagination with Eloquent models.
    ```php
    $products = Products::where('price', '>', 100)->cursorPaginate(10);
    ```

**Multiple Paginator Instances Per Page:**

-   Multiple paginators on a single page can cause conflicts if they use the same `page` query string parameter.
-   Pass the query string parameter name to `paginate`, `simplePaginate`, or `cursorPaginate` as the third argument.
-   Example: Use `products` as the query string parameter name.
    ```php
    use App\Models\Products;

    $products = Products::where('price', '>', 100)->paginate(
        $perPage = 10, $columns = ['*'], $pageName = 'products'
    );
    ```

### &#10022; Cursor Pagination:

-   Cursor pagination uses `where` clauses instead of the SQL `offset` clause.
-   Cursor pagination provides the most efficient database performance.
-   It is well-suited for large data-sets and `infinite` scrolling UIs.
-   Cursor pagination uses a `cursor` string in the query string instead of a page number.
    -   Example: `http://localhost/products?cursor=eyJpZCI6MTUsIl9wb2ludHNUb05leHRJdGVtcyI6dHJ1ZX0`
-   The `cursorPaginate` method returns an `Illuminate\Pagination\CursorPaginator` instance.
    ```php
    $products = DB::table('products')->orderBy('category')->cursorPaginate(10);
    ```
-   Cursor paginator results are displayed similarly to `paginate` and `simplePaginate`.
-   An `orderBy` clause is required for cursor pagination.

**Cursor vs. Offset Pagination:**

-   Example SQL queries to display the second page of products ordered by category:
    -   Offset Pagination: `select * from products order by category asc limit 10 offset 10;`
    -   Cursor Pagination: `select * from products where ... order by category asc limit 10;`

**Cursor Pagination Advantages:**

-   Better performance for large data-sets with indexed "order by" columns.
-   Avoids skipping or duplicating records in data-sets with frequent writes.

**Cursor Pagination Limitations:**

-   Only supports `Next` and `Previous` links, not page number links.
-   Requires ordering based on at least one unique column or a unique combination of columns.
-   Columns with null values are not supported.
-   Query expressions in "order by" clauses are supported only if they are aliased and added to the "select" clause.

### &#10022; Manually Creating A Paginator:

-   A pagination instance can be created manually with an array of in-memory items.
-   Use `Illuminate\Pagination\Paginator`, `Illuminate\Pagination\LengthAwarePaginator`, or `Illuminate\Pagination\CursorPaginator` instances.
-   `Paginator` and `CursorPaginator` don't require the total item count, thus lack last page index methods.
-   `LengthAwarePaginator` requires the total item count.
-   `Paginator` is equivalent to `simplePaginate`, `CursorPaginator` to `cursorPaginate`, and `LengthAwarePaginator` to `paginate`.
-   Manually slice the array of results passed to the paginator.
-   Use the `array_slice` PHP function for slicing.

### &#10022; Customizing Pagination URLs:

-   Paginator links default to the current request's URI.
-   `withPath` method customizes the URI used by the paginator for links.
-   Example: Generate links like `http://example.host/products?page=N`.
    ```php
    use App\Models\Products;

    Route::get('/products', function () {
        $products = Products::paginate(10);

        $products->withPath('/products');

        // ...
    });
    ```

**Appending Query String Values:**

-   `appends` method appends to the query string of pagination links.
-   Example: Append `sort=category` to each pagination link.
    ```php
    use App\Models\Products;

    Route::get('/products', function () {
        $products = Products::paginate(10);

        $products->appends(['sort' => 'category']);

        // ...
    });
    ```
-   `withQueryString` appends all current request query string values to pagination links.
    ```php
    $products = Products::paginate(10)->withQueryString();
    ```

**Appending Hash Fragments:**

-   `fragment` method appends a "hash fragment" to paginator-generated URLs.
-   Example: Append `#products` to each pagination link.
    ```php
    $products = Products::paginate(10)->fragment('products');
    ```

### &#10022; Displaying Pagination Results:

-   `paginate` returns an `Illuminate\Pagination\LengthAwarePaginator` instance.
-   `simplePaginate` returns an `Illuminate\Pagination\Paginator` instance.
-   `cursorPaginate` returns an `Illuminate\Pagination\CursorPaginator` instance.
-   These objects provide methods describing the result set.
-   Paginator instances are iterators and can be looped as arrays.
-   Example Blade template to display results and links:
    ```php
    <div class="container">
        @foreach ($products as $product)
            {{ $product->name }}
        @endforeach
    </div>

    {{ $products->links() }}
    ```
-   The `links` method renders links to the rest of the pages.
-   Links contain the correct page query string variable.
-   HTML generated by `links` is compatible with Tailwind CSS.

### &#10022; Adjusting The Pagination Link Window:

-   The paginator displays the current page and three pages before and after it.
-   The `onEachSide` method controls the number of additional links displayed on each side of the current page.
-   Example: Display five additional links on each side.
    ```php
    {{ $products->onEachSide(5)->links() }}
    ```

### &#10022; Converting Results To JSON:

-   Laravel paginator classes implement `Illuminate\Contracts\Support\Jsonable` and expose `toJson`.
-   Pagination results can be easily converted to JSON.
-   Returning a paginator instance from a route or controller action converts it to JSON.
    ```php
    use App\Models\Products;

    Route::get('/products', function () {
        return Products::paginate();
    });
    ```
-   The JSON includes meta information: `total`, `current_page`, `last_page`, etc.
-   Result records are available via the `data` key in the JSON array.
-   Example JSON output:
    ```json
    {
        "total": 150,
        "per_page": 15,
        "current_page": 1,
        "last_page": 10,
        "first_page_url": "http://laravel.app?page=1",
        "last_page_url": "http://laravel.app?page=10",
        "next_page_url": "http://laravel.app?page=2",
        "prev_page_url": null,
        "path": "http://example.host",
        "from": 1,
        "to": 15,
        "data":[
            {
                // Record...
            },
            {
                // Record...
            }
        ]
    }
    ```

### &#10022; Customizing The Pagination View:

-   Default pagination link views are compatible with Tailwind CSS.
-   Custom views can be defined for rendering these links.
-   Pass the view name as the first argument to the `links` method.
    ```php
    {{ $paginator->links('view.name') }}
    {{ $paginator->links('view.name', ['key' => 'value']) }}
    ```
-   Export pagination views to `resources/views/vendor` using `vendor:publish`.
    ```bash
    php artisan vendor:publish --tag=laravel-pagination
    ```
-   Views are placed in `resources/views/vendor/pagination` directory.
-   `tailwind.blade.php` is the default pagination view.
-   Edit this file to modify the pagination HTML.
-   `defaultView` and `defaultSimpleView` methods in `App\Providers\AppServiceProvider`'s `boot` method set a different default pagination view.
    ```php
    namespace App\Providers;

    use Illuminate\Pagination\Paginator;
    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        public function boot()
        {
            Paginator::defaultView('view-name');
            Paginator::defaultSimpleView('view-name');
        }
    }
    ```

### &#10022; Using Bootstrap:

-   Laravel includes pagination views built with Bootstrap CSS.
-   Use `useBootstrap` method in `App\Providers\AppServiceProvider`'s `boot` method to use Bootstrap views.
    ```php
    use Illuminate\Pagination\Paginator;

    public function boot()
    {
        Paginator::useBootstrap();
    }
    ```

### &#10022; Paginator and LengthAwarePaginator Instance Methods:
| Method | Description |
|---|---|
| `$paginator->count()` |  Gets the number of items for the current page. |
| `$paginator->currentPage()` |  Gets the current page number. |
| `$paginator->firstItem()` |  Gets the result number of the first item in the results. |
| `$paginator->getOptions()` |  Gets the paginator options. |
| `$paginator->getUrlRange($start, $end)` |  Creates a range of pagination URLs. |
| `$paginator->hasPages()` |  Determines if there are enough items to split into multiple pages. |
| `$paginator->hasMorePages()` |  Determines if there are more items in the data store. |
| `$paginator->items()` |  Gets the items for the current page. |
| `$paginator->lastItem()` |  Gets the result number of the last item in the results. |
| `$paginator->lastPage()` |  Gets the page number of the last available page (not for `simplePaginate`). |
| `$paginator->nextPageUrl()` |  Gets the URL for the next page. |
| `$paginator->onFirstPage()` |  Determines if the paginator is on the first page. |
| `$paginator->perPage()` |  Gets the number of items to be shown per page. |
| `$paginator->previousPageUrl()` |  Gets the URL for the previous page. |
| `$paginator->total()` |  Gets the total number of matching items (not for `simplePaginate`). |
| `$paginator->url($page)` |  Gets the URL for a given page number. |
| `$paginator->getPageName()` |  Gets the query string variable used to store the page. |
| `$paginator->setPageName($name)` |  Sets the query string variable used to store the page. |


### &#10022; Cursor Paginator Instance Methods:

| Method | Description |
|---|---|
| `$paginator->count()` | Get the number of items for the current page. |
| `$paginator->cursor()` | Get the current cursor instance. |
| `$paginator->getOptions()` | Get the paginator options. |
| `$paginator->hasPages()` | Determine if there are enough items to split into multiple pages. |
| `$paginator->hasMorePages()` | Determine if there are more items in the data store. |
| `$paginator->getCursorName()` | Get the query string variable used to store the cursor. |
| `$paginator->items()` | Get the items for the current page. |
| `$paginator->nextCursor()` | Get the cursor instance for the next set of items. |
| `$paginator->nextPageUrl()` | Get the URL for the next page. |
| `$paginator->onFirstPage()` | Determine if the paginator is on the first page. |
| `$paginator->perPage()` | The number of items to be shown per page. |
| `$paginator->previousCursor()`| Get the cursor instance for the previous set of items. |
| `$paginator->previousPageUrl()`| Get the URL for the previous page. |
| `$paginator->setCursorName()` | Set the query string variable used to store the cursor. |
| `$paginator->url($cursor)` | Get the URL for a given cursor instance. |

---
[&#8682; To Top](#-pagination)

[&#10094; Previous Topic](./query-builder.md) &emsp; [Next Topic &#10095;](./eloquent-orm.md)

[&#8962; Goto Home Page](../README.md)