#Database: Query Builder

Link document: https://laravel.com/docs/7.x/queries
Position: Done basic knowledge


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


##**#Joins**

####**Inner Join Clause**

- The query builder may also be used to write join statements. To perform a basic "inner join", you may use the `join` method on a query builder instance.
- The first argument passed to the `join` method is the name of the table you need to join to, Các tham số sau là ràng buộc để nối bảng
- You can even join to multiple tables in a single query

```php
$users = DB::table('users')
            ->join('contacts', 'users.id', '=', 'contacts.user_id')
            ->join('orders', 'users.id', '=', 'orders.user_id')
            ->select('users.*', 'contacts.phone', 'orders.price')
            ->get();
```

#####**Left Join / Right Join Clause**

- If you would like to perform a "left join" or "right join" instead of an "inner join", use the `leftJoin` or `rightJoin` methods. These methods have the same signature as the join method:

```php
$users = DB::table('users')
            ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
            ->get();

$users = DB::table('users')
            ->rightJoin('posts', 'users.id', '=', 'posts.user_id')
            ->get();
```

#####**Cross Join Clause**
- Phép nối tự nhiên, mỗi bản ghi ở bảng 1 sẽ join với mỗi bản ghi ở bảng 2

```php
$sizes = DB::table('sizes')
            ->crossJoin('colors')
            ->get();
```

#####**Advanced Join Clauses**

- You may also specify more advanced join clauses. To get started, pass a Closure as the second argument into the join method. The Closure will receive a JoinClause object which allows you to specify constraints on the join clause:

```php
DB::table('users')
        ->join('contacts', function ($join) {
            $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
        })
        ->get();
```

- If you would like to use a "where" style clause on your joins, you may use the `where` and `orWhere` methods on a join. Instead of comparing two columns, these methods will compare the column against a value:

```php
DB::table('users')
        ->join('contacts', function ($join) {
            $join->on('users.id', '=', 'contacts.user_id')
                 ->where('contacts.user_id', '>', 5);
        })
        ->get();
```

#####**Subquery Joins**

- You may use the `joinSub`, `leftJoinSub`, and `rightJoinSub` methods to join a query to a subquery. 
- Each of these methods receive three arguments: 
	+ the subquery, 
	+ its table alias
	+ a Closure that defines the related columns

```php
$latestPosts = DB::table('posts')
                   ->select('user_id', DB::raw('MAX(created_at) as last_post_created_at'))
                   ->where('is_published', true)
                   ->groupBy('user_id');

$users = DB::table('users')
        ->joinSub($latestPosts, 'latest_posts', function ($join) {
            $join->on('users.id', '=', 'latest_posts.user_id');
        })->get();
```

##**#Unions**

- The query builder also provides a quick way to "union" two queries together. For example, you may create an initial query and use the `union` method to union it with a second query:

```php
$first = DB::table('users')
            ->whereNull('first_name');

$users = DB::table('users')
            ->whereNull('last_name')
            ->union($first)
            ->get();
```

** Tip: 
The unionAll method is also available and has the same method signature as union**


##**#Where Clauses**

#####**Simple Where Clauses**

- `where` method on a query builder instance to add `where` clauses to the query- The most basic call to where requires three arguments
- The first argument is the name of the column.
- The second argument is an operator, which can be any of the database's supported operators.
- Finally, the third argument is the value to evaluate against the column.

```php
$users = DB::table('users')->where('votes', '=', 100)->get();
```

For example, here is a query that verifies the value of the "votes" column is equal to 100:

- You may use a variety of other operators when writing a where clause:

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

- You may also pass an array of conditions to the where function:

```php
$users = DB::table('users')->where([
    ['status', '=', '1'],
    ['subscribed', '<>', '1'],
])->get();
```

#####**Or Statements**

Các cách thực Or statements
- Thêm or vào mệnh đề truy vấn
- The orWhere method accepts the same arguments as the where method

```php
$users = DB::table('users')
                    ->where('votes', '>', 100)
                    ->orWhere('name', 'John')
                    ->get();
```
- Custome `orWhere` method:

```php
$users = DB::table('users')
            ->where('votes', '>', 100)
            ->orWhere(function($query) {
                $query->where('name', 'Abigail')
                      ->where('votes', '>', 50);
            })
            ->get();

// SQL: select * from users where votes > 100 or (name = 'Abigail' and votes > 50)
```

#####**Additional Where Clauses**
Readlater



##**#Parameter Grouping**

- Sometimes you may need to create more advanced where clauses such as "where exists" clauses or nested parameter groupings. The Laravel query builder can handle these as well. To get started, let's look at an example of grouping constraints within parenthesis:

```php
$users = DB::table('users')
           ->where('name', '=', 'John')
           ->where(function ($query) {
               $query->where('votes', '>', 100)
                     ->orWhere('title', '=', 'Admin');
           })
           ->get();
```

- As you can see, passing a `Closure` into the `where` method instructs the query builder to begin a constraint group. The `Closure` will receive a query builder instance which you can use to set the constraints that should be contained within the parenthesis group. The example above will produce the following SQL:

```php
select * from users where name = 'John' and (votes > 100 or title = 'Admin')
```

** Tip:

You should always group `orWhere` calls in order to avoid unexpected behavior when global scopes are applied.
**


##**#Where Exists Clauses**

- The `whereExists` method allows you to write `where exists` SQL clauses.
- The `whereExists` method accepts a `Closure` argument, which will receive a query builder instance allowing you to define the query that should be placed inside of the "exists" clause:

```php
$users = DB::table('users')
           ->whereExists(function ($query) {
               $query->select(DB::raw(1))
                     ->from('orders')
                     ->whereRaw('orders.user_id = users.id');
           })
           ->get();
```

- The query above will produce the following SQL:

```sql
select * from users
where exists (
    select 1 from orders where orders.user_id = users.id
)
```
(return all user ordered )

##**#Subquery Where Clauses**

- Sometimes you may need to construct a where clause that compares the results of a subquery to a given value. 
- You may accomplish this by passing a Closure and a value to the where method.
-  For example, the following query will retrieve all users who have a recent "membership" of a given type:

```php
use App\User;

$users = User::where(function ($query) {
    $query->select('type')
        ->from('membership')
        ->whereColumn('user_id', 'users.id')
        ->orderByDesc('start_date')
        ->limit(1);
}, 'Pro')->get();

```

##**#JSON Where Clauses**

- Laravel also supports querying JSON column types on databases that provide support for JSON column types.
- Currently, this includes MySQL 5.7, PostgreSQL, SQL Server 2016, and SQLite 3.9.0 (with the JSON1 extension).
- To query a JSON column, use the -> operator:

```php
$users = DB::table('users')
                ->where('options->language', 'en')
                ->get();

$users = DB::table('users')
                ->where('preferences->dining->meal', 'salad')
                ->get();
```

- You may use `whereJsonContains` to query JSON arrays (not supported on SQLite):

```php
$users = DB::table('users')
                ->whereJsonContains('options->languages', 'en')
                ->get();
```

- MySQL and PostgreSQL support `whereJsonContains` with multiple values:

```php
$users = DB::table('users')
                ->whereJsonContains('options->languages', ['en', 'de'])
                ->get();
```

- You may use `whereJsonLength` to query JSON arrays by their length:

```php
$users = DB::table('users')
                ->whereJsonLength('options->languages', 0)
                ->get();

$users = DB::table('users')
                ->whereJsonLength('options->languages', '>', 1)
                ->get();
```


##**#Ordering, Grouping, Limit & Offset**

#####**orderBy**

- `orderBy` method, option `asc`: default or `desc`

```php
$users = DB::table('users')
                ->orderBy('name', 'desc')
                ->get();
```

#####**latest / oldest**

- The `latest` and `oldest` methods allow you to easily order results by date. 
- By default, result will be ordered by the `created_at` column.
- Or, you may pass the column name that you wish to sort by:

```php
$user = DB::table('users')
                ->latest()
                ->first();
```

#####**inRandomOrder**

- The inRandomOrder method may be used to sort the query results randomly

```php
$randomUser = DB::table('users')
                ->inRandomOrder()
                ->first();
```

#####**reorder**

- The reorder method allows you to remove all the existing orders and optionally apply a new order.

```php
$query = DB::table('users')->orderBy('name');

$unorderedUsers = $query->reorder()->get();
```

Or

```php
$query = DB::table('users')->orderBy('name');

$usersOrderedByEmail = $query->reorder('email', 'desc')->get();
```

#####**groupBy / having**
- The `groupBy` and `having` methods may be used to group the query results
- The `having` method's signature is similar to that of the `where` method
```php
$users = DB::table('users')
                ->groupBy('account_id')
                ->having('account_id', '>', 100)
                ->get();
```
- You may pass multiple arguments to the groupBy method to group by multiple columns:

```php
$users = DB::table('users')
                ->groupBy('first_name', 'status')
                ->having('account_id', '>', 100)
                ->get();
```	

** Tip: For more advanced having statements, see the havingRaw method.**


#####**skip / take**

- To limit the number of results returned from the query, or to skip a given number of results in the query
- You may use the `skip` and `take` methods:

```php
$users = DB::table('users')->skip(10)->take(5)->get();
```

- Alternatively, you may use the limit and offset methods:

```php
$users = DB::table('users')
                ->offset(10)
                ->limit(5)
                ->get();
```


##**#Conditional Clauses**

- Sometimes you may want clauses to apply to a query only when something else is true. 
- For instance you may only want to apply a `where` statement if a given input value is present on the incoming request.
- You may accomplish this using the `when` method:

```php
role = $request->input('role');

$users = DB::table('users')
                ->when($role, function ($query, $role) {
                    return $query->where('role_id', $role);
                })
                ->get();
```

- The `when` method only executes the given Closure when the first parameter is `true`. If the first parameter is `false`, the Closure will not be executed.

*

- You may pass another Closure as the third parameter to the when method. 
- This Closure will execute if the first parameter evaluates as false. 
- To illustrate how this feature may be used, we will use it to configure the default sorting of a query:

```php
$sortBy = null;

$users = DB::table('users')
                ->when($sortBy, function ($query, $sortBy) {
                    return $query->orderBy($sortBy);
                }, function ($query) {
                    return $query->orderBy('name');
                })
                ->get();
```

##**#Inserts**

- The query builder also provides an `insert` method for inserting records into the database table. The `insert` method accepts an array of column names and values:

```php
DB::table('users')->insert(
    ['email' => 'john@example.com', 'votes' => 0]
);
```

- Insert several records by passing an array of arrays:

```php
DB::table('users')->insert([
    ['email' => 'taylor@example.com', 'votes' => 0],
    ['email' => 'dayle@example.com', 'votes' => 0],
]);
```

- The `insertOrIgnore` method will ignore duplicate record errors while inserting records into the database:

```php
DB::table('users')->insertOrIgnore([
    ['id' => 1, 'email' => 'taylor@example.com'],
    ['id' => 2, 'email' => 'dayle@example.com'],
]);
```

#####**Auto-Incrementing IDs**
- `insertGetId` method to insert a record and then retrieve the ID:

```php
$id = DB::table('users')->insertGetId(
    ['email' => 'john@example.com', 'votes' => 0]
);
```

** Tip: When using PostgreSQL the insertGetId method expects the auto-incrementing column to be named id. If you would like to retrieve the ID from a different "sequence", you may pass the column name as the second parameter to the insertGetId method.**



##**#Updates**

- `update` method update existing records
- The `update` method, like the `insert` method, accepts an array of column and value pairs containing the columns to be updated.
- You may constrain the `update` query using `where` clauses:

```php
$affected = DB::table('users')
              ->where('id', 1)
              ->update(['votes' => 1]);
```


#####**Update Or Insert**
- `updateOrInsert`: update an existing record in the database or create it if no matching record exists
- The updateOrInsert method accepts two arguments: an array of conditions by which to find the record, and an array of column and value pairs containing the columns to be updated.

```php
DB::table('users')
    ->updateOrInsert(
        ['email' => 'john@example.com', 'name' => 'John'],
        ['votes' => '2']
    );
```

##**#Updating JSON Columns**

- When updating a JSON column, you should use -> syntax to access the appropriate key in the JSON object. This operation is supported on MySQL 5.7+ and PostgreSQL 9.5+:

```php
$affected = DB::table('users')
              ->where('id', 1)
              ->update(['options->enabled' => true]);
```

##**#Increment & Decrement**
Read later


##**#Deletes**

- Delete records from the table via the `delete` method. 

- You may constrain delete statements by adding where clauses before calling the delete method:

```php
DB::table('users')->delete();

DB::table('users')->where('votes', '>', 100)->delete();
```
- If you wish to truncate the entire table, which will remove all rows and reset the auto-incrementing ID to zero, you may use the truncate method

```php
DB::table('users')->truncate();
```

##**#Pessimistic Locking**

- The query builder also includes a few functions to help you do "pessimistic locking" on your `select` statements.
- To run the statement with a "shared lock", you may use the `sharedLock` method on a query.
- A shared lock prevents the selected rows from being modified until your transaction commits:

```php
DB::table('users')->where('votes', '>', 100)->sharedLock()->get();
```

- Alternatively, you may use the `lockForUpdate` method.
- A "for update" lock prevents the rows from being modified or from being selected with another shared lock:

```php
DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();
```

##**#Debugging**

- You may use the `dd` or `dump` methods while building a query to dump the query bindings and SQL. 
- The `dd` method will display the debug information and then stop executing the request.
- The dump method will display the debug information but allow the request to keep executing:


```php
DB::table('users')->where('votes', '>', 100)->dd();

DB::table('users')->where('votes', '>', 100)->dump();
```

Done

