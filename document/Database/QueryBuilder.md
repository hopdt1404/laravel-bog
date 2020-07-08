#Database: Query Builder

####**#Introduction**
####**#Retrieving Results**
   ####Chunking Results
   ####Aggregates
####**#Selects**
####**#Raw Expressions**
####**#Joins**
####**#Unions**
####**#Where Clauses**
    ####**#Parameter Grouping**
    ####**#Where Exists Clauses**
    ####**#Subquery Where Clauses**
    ####**#JSON Where Clauses**
####**#Ordering, Grouping, Limit & Offset**
####**#Conditional Clauses**
####**#Inserts**
####**#Updates**
####**#Updating JSON Columns**
####**#Increment & Decrement***
####**#Deletes**
####**#Pessimistic Locking**
####**#Debugging**

##**#Introduction**

"Laravel's database query builder provides a convenient, fluent interface to creating and running database queries. It can be used to perform most database operations in your application and works on all supported database systems."

- Laravel query builder uses PDO parameter binding to protect your application against SQL injection attacks

** Note:
PDO does not support binding column names. Therefore, you should never allow user input to dictate the column names referenced by your queries, including "order by" columns, etc. If you must allow the user to select certain columns to query against, always validate the column names against a white-list of allowed columns.
 **

##**#Retrieving Results**

#####**Retrieving All Rows From A Table**
- Use: `table` method on the  `DB` facade to begin a query, using the `get` method

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\DB;

class UserController extends Controller
{
    /**
     * Show a list of all of the application's users.
     *
     * @return Response
     */
    public function index()
    {
        $users = DB::table('users')->get();

        return view('user.index', ['users' => $users]);
    }
}
```

- The get method returns an Illuminate\Support\Collection containing the results where each result is an instance of the PHP stdClass object. 
- You may access each column's value by accessing the column as a property of the object:

```php
foreach ($users as $user) {
    echo $user->name;
}
```

#####**Retrieving A Single Row / Column From A Table**
- Use: `frist` method, this method will return a single stdClass object:

```php
$user = DB::table('users')->where('name', 'John')->first();

echo $user->name;
```

- Extract s single value use: `value` method, this method will return the value of the column directly.

```$email = DB::table('users')->where('name', 'John')->value('email');```

- To retrieve a single row by its `id` column value, use the `find` method:

```$user = DB::table('users')->find(3);```

######**Retrieving A List Of Column Values**
- Retrieve a Collection containing the values of a single column, you may use the pluck method. In this example, we'll retrieve a Collection of role titles:

```php
$titles = DB::table('roles')->pluck('title');

foreach ($titles as $title) {
    echo $title;
}
```

- You may also specify a custom key column for the returned Collection:

```php
$roles = DB::table('roles')->pluck('title', 'name');

foreach ($roles as $name => $title) {
    echo $title;
}
```

##**#Chunking Results**
- `chunk` method, method retrieves a small chunk of the results at a time and feeds each chunk into a `Closure` for processing
- This method is very useful for writing `Artisan commands` that process thousands of records. 
- For example, let's work with the entire users table in chunks of 100 records at a time: 
```php
DB::table('users')->orderBy('id')->chunk(100, function ($users) {
    foreach ($users as $user) {
        //
    }
});
```

- Can stop further chunks from being processed by returning `false` from the `Closure`:

- If you are updating database records while chunking results, your chunk results could change in unexpected ways. So, when updating records while chunking, it is always best to use the `chunkById` method instead. This method will automatically paginate the results based on the record's primary key:

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

**Note:
When updating or deleting records inside the chunk callback, any changes to the primary key or foreign keys could affect the chunk query. This could potentially result in records not being included in the chunked results.
**


##**Aggregates**

- Aggregate methods: `count`, `max`, `min`, `avg`, and `sum`.

- You may call any of these methods after constructing your query:
```php
$users = DB::table('users')->count();

$price = DB::table('orders')->max('price');
```

- You may combine these methods with other clauses:
```
$price = DB::table('orders')
                ->where('finalized', 1)
                ->avg('price');
```


#####**Determining If Records Exist**
- `count` method to determine if any records exist that match your query's constraints, you may use the exists and doesntExist methods:
```php
return DB::table('orders')->where('finalized', 1)->exists();

return DB::table('orders')->where('finalized', 1)->doesntExist();
```

##**#Selects**
#####**Specifying A Select Clause**

- You want select some columns from database table. 
- Using the `select` method, you can specify a custom select clause for the query

```php
$users = DB::table('users')->select('name', 'email as user_email')->get();
```

- The distinct method allows you to force the query to return distinct results:
```php
$users = DB::table('users')->distinct()->get();
```

- If you already have a query builder instance and you wish to add a column to its existing select clause, you may use the `addSelect` method:

```
$query = DB::table('users')->select('name');

$users = $query->addSelect('age')->get();
```

##**#Raw Expressions**

- You may need to use a raw expression in a query. To create a raw expression, you may use the DB::raw method

```php
$users = DB::table('users')
                     ->select(DB::raw('count(*) as user_count, status'))
                     ->where('status', '<>', 1)
                     ->groupBy('status')
                     ->get();
```

** Note: 
Raw statements will be injected into the query as strings, so you should be extremely careful to not create SQL injection vulnerabilities.**


#####**#Raw Methods**

- Instead of using `DB::raw`, you may also use the following methods to insert a raw expression into various parts of your query.

- `selectRaw` có thể thay thế cho `addSelect(DB::raw(...)). Phương thức chấp nhận một mảng các ràng buộc tùy ý làm đối số thứ hai


```php
$orders = DB::table('orders')
                ->selectRaw('price * ? as price_with_tax', [1.0825])
                ->get();
```
bindings: price_with_tax = price * argument(1.0825)

Read later:
`whereRaw / orWhereRaw`

`havingRaw / orHavingRaw`

`orderByRaw`

`groupByRaw`


##**#









Link: https://laravel.com/docs/7.x/queries
Position: Retrieving Results
