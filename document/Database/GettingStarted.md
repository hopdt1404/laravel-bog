# Database: Getting Started

Link: https://laravel.com/docs/7.x/database
Done document

#####**#Introduction**
        #Configuration
        #Read & Write Connections
        #Using Multiple Database Connections
#####**#Running Raw SQL Queries**
#####**#Listening For Query Events**
#####**#Database Transactions**

##**#Introduction**

- Laravel hỗ trợ tương tác với các database backends using either raw SQL, tạo truy vấn, và Eloquent ORM.
- Currently, Laravel supports four databases:
```
+ MySQL 5.6+ (Version Policy)
+ PostgreSQL 9.4+ (Version Policy)
+ SQLite 3.8.8+
+ SQL Server 2017+ (Version Policy)
```
####**#Configuration**

- Configurate trong `config/database.php`. Define all of database connections, as well specify which connection should be used by default.
- Hầu hết các supported database systems are provided in this file


##### SQLite Configuration
- Sau khi tạo một database thì thực thi câu lệnh để thực hiện configure your enviroment variables 
```
touch database/database.sqlite
```

- This newly created database by using the database's absolute path (đường dẫn tuyệt đối)

```sql
DB_CONNECTION=sqlite
DB_DATABASE=/absolute/path/to/database.sqlite
```

- To enable foreign key constraints for SQLite connections, you should set the `DB_FOREIGN_KEYS` environment variable to `true`:

`DB_FOREIGN_KEYS=true`

##### Configuration Using URLs
- Typically, database connections are configured using multiple configuration values such as `host`, `database`, `username`, `password`, etc.
- Mỗi một variable enviroment sẽ có một giá trị riêng. Ta cần manage several enviroment before connection to database 
- Một số URL có thể chức các thông tin cho kết nối như
`mysql://root:password@127.0.0.1/forge?charset=UTF-8`

- Các URL thường tuân theo chuẩn schema convention:
`driver://username:password@host:port/database?options`

- Laravel hỗ trợ configuration. Option `url` sử dụng để extracr the database connection and credential(xác thực) information thông qua (DATABASE_URL enviroment variable)

####**# Read & Write Connections**
- Sometimes you may wish to use one database connection for SELECT statements, and another for INSERT, UPDATE, and DELETE statements. Laravel makes this a breeze, and the proper connections will always be used whether you are using raw queries, the query builder, or the Eloquent ORM.

- To see how read / write connections should be configured, let's look at this example:

```sql
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
    'database' => 'database',
    'username' => 'root',
    'password' => '',
    'charset' => 'utf8mb4',
    'collation' => 'utf8mb4_unicode_ci',
    'prefix' => '',
],
```
- Three keys have been added to the configuration array: `read`, `write` and `sticky`.
- The `read` and `write` keys have array values containing a single key: `host`.
- Phần còn lại của database options for the `read` and `write` connecitions will be gộp lại từ main `mysql` array.

- You only need to place items in the `read` and `write` arrays if you wish to override the values from the main array. So, in this case, `192.168.1.1` will be used as the host for the "read" connection, while `192.168.1.3` will be used for the "write" connection. Những option khác thì được cùng chia sẻ trên cả 2 kết nối

**The `sticky` option:
- `sticky` option used to allow the: đọc ngay các bản ghi đã được ghi vào cơ sở dữ l với các request trong chu kỳ hiện tại.
- If the `sticky` option is enabled and a "write" đã được kết nối đến database ttrong current request cycle, mọi thao tác `read` tiếp theo sẽ sử dụng để `write`- Tức là mọi dữ liệu sau khi đã được `write` thì có thể được `read` ngay lập tức trong cùng request cycle 
- Tuy vào ứng dụng thì ta sẽ xem xét yếu tố này


####**#Using Multiple Database Connections**

- When using multiple connections, you may access each `connection` via the connection method on the `DB`facade. The `name` passed to the `connection` method should correspond to one of the connections listed in your `config/database.php` configuration file:
```sql
$users = DB::connection('foo')->select(...);
```
- Có thể truy cập vào PDO instace using the `getPdo` method on a connection instace:

```php
$pdo = DB::connection()->getPdo();
```

####**#Running Raw SQL Queries**

- Once you have configured your database connection, you may run queries using the `DB` facade. The DB facade provides methods for each type of query: `select`, `update`, `insert`, `delete`, and `statement`.

#####**Running A Select Query**

- To run a basic query, you may use the select method on the DB facade:

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
        $users = DB::select('select * from users where active = ?', [1]);

        return view('user.index', ['users' => $users]);
    }
}
```
- The first argument passed to the select method is the raw SQL query.
- The second argument is bất cứ tham số nào cũng cần được binding với một query.- Typically, these are the values of the `where` clause constraints
- Parameter binding provides protection against SQL injection.
- The `select` method will always return an `array` of results. Each result within the array will be a PHP `stdClass` object, allowing you to access the values of the results:

```php
foreach ($users as $user) {
    echo $user->name;
}
```

#####**Using Named Bindings**
- Instead of using `?` to represent your parameter bindings, you may execute a query using named bindings:

```sql
$results = DB::select('select * from users where id = :id', ['id' => 1]);
```

#####**Running An Insert Statement**

- To execute an `insert` statement, you may use the `insert method` on the DB facade. Like `select`, this method takes the raw SQL query as its first argument and bindings as its second argument:
```
DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);
```

#####**Running An Update Statement**
- The `update` method should be used to update existing records in the database. The number of rows affected by the statement will be returned:
```
$affected = DB::update('update users set votes = 100 where name = ?', ['John']);
```

#####**Running A Delete Statement**
- The `delete` method should be used to delete records from the database. Like update, the number of rows affected will be returned:
```
$deleted = DB::delete('delete from users');
```

#####**Running A General Statement**
- Một vài statements không trả về bất cứ value
- Và ta sử dụng `statement` method on the `DB` facade:
```
DB::statement('drop table users');

```


##**#Listening For Query Events**
- If you would like to receive each SQL query executed by your application, you may use the `listen` method. This method is useful for logging queries or debugging. You may register your query listener in a service provider:

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\DB;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        //
    }

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        DB::listen(function ($query) {
            // $query->sql
            // $query->bindings
            // $query->time
        });
    }
}
```


##**#Database Transactions**
- You may use the `transaction` method on the `DB` facade to run a set of operations within a database transaction. If an exception is thrown within the transaction `Closure`, the transaction will automatically be rolled back. If the Closure executes successfully, the transaction will automatically be committed. You don't need to worry about manually rolling back or committing while using the `transaction` method:

```
DB::transaction(function () {
    DB::table('users')->update(['votes' => 1]);

    DB::table('posts')->delete();
})
```


#####**Handling Deadlocks**
- The `transaction` method accepts an optional second argument which defines số lần thực thi khi xảy ra deadlock. Khi không còn deadlock thì sẽ thrown exception.

```
DB::transaction(function () {
    DB::table('users')->update(['votes' => 1]);

    DB::table('posts')->delete();
}, 5);
```

#####**Manually Using Transactions**
- Sẽ có trường hợp bạn cần 1 `transaction` và các thao tác đó bạn tự configure đảm bảo viêc rollbacks and commits, thì có thể sử dụng `beginTransaction` method on the DB facade:
```
DB::beginTransaction();
```

- You can rollback the transaction via the `rollBack` method:
`DB::rollBack();`

- Lastly, you can commit a transaction via the commit method:

`DB::commit();`







