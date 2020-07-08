#Database: Migrations

Link: https://laravel.com/docs/7.x/migrations
Position: Done basic knowledge
####**#Generating Migrations**
####**#Migration Structure**
####**#Running Migrations**
   + Rolling Back Migrations
####**#Tables**
   + Creating Tables
   + Renaming / Dropping Tables
####**#Columns**
   + Creating Columns
   + Column Modifiers
   + Modifying Columns
   + Dropping Columns
####**#Indexes**
   + Creating Indexes
   + Renaming Indexes
   + Dropping Indexes
   + Foreign Key Constraints

##**#Introduction**

- Migrations are like version control for your database
- Allowing your team to modify and share the application's database schema
- Migrations are typically paired with Laravel's schema builder to build your application's database schema. 
- If you have ever had to tell a teammate to manually add a column to their local database schema, you've faced the problem that database migrations solve.

*
- The Laravel `Schema` facade provides database agnostic support for creating and manipulating tables across all of Laravel's supported database systems.


##**#Generating Migrations**

- To create a migration, use the `make:migration` Artisan command:

```sh
php artisan make:migration create_users_table
```

- The new migration will be placed in your `database/migrations` directory
- Each migration file name contains a timestamp, which allows Laravel to determine the order of the migrations.

**Note: Migration stubs may be customized using stub publishing**

*
- The `--table` and `--create` options may also be used: chỉ ra tên bảng, hoặc thao tác có tạo bảng mới hay không 

```php
php artisan make:migration create_users_table --create=users

php artisan make:migration add_votes_to_users_table --table=users
```

- If you would like to specify a custom output path for the generated migration, you may use the --path option when executing the make:migration command.
- The given path should be relative to your application's base path.


##**#Migration Structure**

- A migration class contains two methods: `up` and `down`.
- The `up` method is used to add new tables, columns, or indexes to your database, while the `down` method should đảo ngược các phương thưc của `up`

*
- Within both of these methods you may use the Laravel schema builder to expressively create and modify tables.
- To learn about all of the methods available on the Schema builder, check out its documentation. For example, the following migration creates a `flights` table:

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateFlightsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('airline');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::drop('flights');
    }
}
```


##**#Running Migrations**

- To run all of your outstanding migrations, execute the `migrate` Artisan command:

```
php artisan migrate
```

####**#Forcing Migrations To Run In Production**
- use the --force flag to thực thi câu lệnh ngay cả khi sẽ gây nguy hiểm tới DB

```
php artisan migrate --force
```

##**#Rolling Back Migrations**

- To roll back the latest migration operation, you may use the `rollback` command. This command rolls back the last "batch" of migrations, which may include multiple migration files:

```php
php artisan migrate:rollback
```

- You may roll back a limited number of migrations by providing the `step` option to the `rollback` command. For example, the following command will roll back the last five migrations:

```php
php artisan migrate:rollback --step=5
```

- The migrate:reset command will roll back all of your application's migrations:

```php
php artisan migrate:reset
```

####**Roll Back & Migrate Using A Single Command**

- The `migrate:refresh` command will roll back all of your migrations and then execute the `migrate` command. This command effectively re-creates your entire database:

```
php artisan migrate:refresh

// Refresh the database and run all database seeds...
php artisan migrate:refresh --seed
```

- You may roll back & re-migrate a limited number of migrations by providing the step option to the `refresh` command. For example, the following command will roll back & re-migrate the last five migrations:

```
php artisan migrate:refresh --step=5
```

####**Drop All Tables & Migrate**

- The `migrate:fresh` command will drop all tables from the database and then execute the `migrate` command:

```
php artisan migrate:fresh

php artisan migrate:fresh --seed
```

##**#Tables**

###**#Creating Tables**

- To create a new database table, use the `create` method on the `Schema` facade. The `create` method accepts two arguments: the first is the name of the table, while the second is a `Closure` which receives a `Blueprint` object that may be used to define the new table:

```php
Schema::create('users', function (Blueprint $table) {
    $table->id();
});
```

- When creating the table, you may use any of the schema builder's `column methods` to define the table's columns.


####**Checking For Table / Column Existence**
- You may check for the existence of a table or column using the `hasTable` and `hasColumn` methods:

```php
if (Schema::hasTable('users')) {
    //
}

if (Schema::hasColumn('users', 'email')) {
    //
}
```

####**Database Connection & Table Options**

- If you want to perform a schema operation on a database connection that is not your default `connection`, use the connection method:

```php
Schema::connection('foo')->create('users', function (Blueprint $table) {
    $table->id();
});
```

- You may use the following commands on the schema builder to define the table's options:

Readlater

####**#Renaming / Dropping Tables**

- To rename an existing database table, use the `rename` method:
```php
Schema::rename($from, $to);
```

- To drop an existing table, you may use the `drop` or `dropIfExists` methods:

```php
Schema::drop('users');

Schema::dropIfExists('users');
```

####**Renaming Tables With Foreign Keys**
- Before renaming a table, you should verify that any foreign key constraints on the table have an tên rõ ràng thay vì để cho Laravel tự gán. 
- Otherwise, the foreign key constraint name will refer to the old table name.

##**#Columns**

###**#Creating Columns**

- The `table` method on the `Schema` facade may be used to update existing tables. Like the `create` method, the `table` method accepts two arguments: the name of the table and a `Closure` that receives a `Blueprint` instance you may use to add columns to the table:

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('email');
});
```

#####**Available Column Types**
In document online

##**#Column Modifiers**

- There are several column "modifiers"

- For example, to make the column "nullable", you may use the `nullable` method

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('email')->nullable();
});
```

- The following list contains all available column modifiers. This list does not include the index modifiers:

In document online

####**Default Expressions**

- The `default` modifier accepts a value or an `\Illuminate\Database\Query\Expression` instance
- Using an `Expression` instance will prevent wrapping the value in quotes and allow you to use database specific functions.
- One situation where this is particularly useful is when you need to assign default values to JSON columns:

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Query\Expression;
use Illuminate\Database\Migrations\Migration;

class CreateFlightsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->id();
            $table->json('movies')->default(new Expression('(JSON_ARRAY())'));
            $table->timestamps();
        });
    }
}
```

**Note: 
Support for default expressions depends on your database driver, database version, and the field type. Please refer to the appropriate documentation for compatibility. Also note that using database specific functions may tightly couple you to a specific driver.**


####**#Modifying Columns**

#####**Prerequisites**

- Before modifying a column, be sure to add the `doctrine/dbal` dependency to your `composer.json` file. The Doctrine DBAL library is used to determine the current state of the column and create the SQL queries needed to make the required adjustments:

```sh
composer require doctrine/dbal
```

#####**Updating Column Attributes**

- The `change` method allows you to `modify type` and `attributes` of existing columns.
- For example, you may wish to increase the size of a `string` column.
- To see the `change` method in action, let's increase the size of the `name` column from 25 to 50:

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('name', 50)->change();
});
```

- We could also modify a column to be nullable:

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('name', 50)->nullable()->change();
});
```

**Note: 
Only the following column types can be "changed": bigInteger, binary, boolean, date, dateTime, dateTimeTz, decimal, integer, json, longText, mediumText, smallInteger, string, text, time, unsignedBigInteger, unsignedInteger, unsignedSmallInteger and uuid.**

####**Renaming Columns**

- To rename a column, you may use the `renameColumn` method on the schema builder.
- Before renaming a column, be sure to add the `doctrine/dbal` dependency to your `composer.json` file:

```php
Schema::table('users', function (Blueprint $table) {
    $table->renameColumn('from', 'to');
});
```


**Note: Renaming any column in a table that also has a column of type enum is not currently supported.**


##**#Dropping Columns**

- To drop a column, use the `dropColumn` method on the schema builder
- Before dropping columns from a SQLite database, you will need to add the `doctrine/dbal` dependency to your `composer.json` file and run the `composer update` command in your terminal to install the library:

```php
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn('votes');
});
```

- You may drop multiple columns from a table by passing an array of column names to the dropColumn method:

```php
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn(['votes', 'avatar', 'location']);
});
```

**Note: Dropping or modifying multiple columns within a single migration while using a SQLite database is not supported.**

####**Available Command Aliases**
Read later in document online

##**#Indexes**

###**#Creating Indexes**

- The Laravel schema builder supports several types of indexes. 
- The following example creates a new `email` column and specifies that its values should be `unique`.
- To create the index, we can chain the unique method onto the column definition:

```php
$table->string('email')->unique();
```

- Alternatively, you may create the index after defining the column. For example:
```php
$table->unique('email');
```

- Tạo index tổng hợp từ các trường dữ liệu
```php
$table->index(['account_id', 'created_at']);
```

- Laravel will automatically generate an index name based on the table, column names, and the index type, but you may pass a second argument to the method to specify the index name yourself:

```$table->unique('email', 'unique_email');```

####**Available Index Types**

- Each index method accepts an optional second argument to specify the name of the index. 
- If omitted, the name will be derived from the names of the table and column(s) used for the index, as well as the index type.

Read later

####**Index Lengths & MySQL / MariaDB**

- Laravel uses the `utf8mb4` character set by default, which includes support for storing "emojis" in the database
-  If you are running a version of MySQL older than the 5.7.7 release or MariaDB older than the 10.2.2 release, you may need to manually configure the default string length generated by migrations in order for MySQL to create indexes for them
- You may configure this by calling the `Schema::defaultStringLength` method within your `AppServiceProvider`:

```php
use Illuminate\Support\Facades\Schema;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Schema::defaultStringLength(191);
}
```

- Alternatively, you may enable the `innodb_large_prefix` option for your database.
- Refer to your database's documentation for instructions on how to properly enable this option.

##**#Renaming Indexes**

- To rename an index, you may use the `renameIndex` method
- This method accepts the current index name as its first argument and the desired new name as its second argument:

```php
$table->renameIndex('from', 'to')
```

##**#Dropping Indexes**

- To drop an index, you must specify the index's name.
- By default, Laravel automatically assigns an index name based on the table name, the name of the indexed column, and the index type. Here are some examples:

Read later

- If you pass an array of columns into a method that drops indexes

```php
Schema::table('geo', function (Blueprint $table) {
    $table->dropIndex(['state']); // Drops index 'geo_state_index'
});
```

##**#Foreign Key Constraints**

- Laravel also provides support for creating foreign key constraints, which are used to force referential integrity at the database level
- For example, let's define a `user_id` column on the `posts` table that references the `id` column on a `users` table:

```php
Schema::table('posts', function (Blueprint $table) {
    $table->unsignedBigInteger('user_id');

    $table->foreign('user_id')->references('id')->on('users');
});
```

- Hoặc cú pháp ngắn gọn hơn

```php
Schema::table('posts', function (Blueprint $table) {
    $table->foreignId('user_id')->constrained();
});
```

- The `foreignId` method is an alias for `unsignedBigInteger` while the `constrained` method will use convention to determine the table and column name being referenced

- If your table name does not match the convention, you may specify the table name by passing it as an argument to the `constrained` method:

```php
Schema::table('posts', function (Blueprint $table) {
    $table->foreignId('user_id')->constrained('users');
});
```

- You may also specify the desired action for the "on delete" and "on update" properties of the constraint:

```php
$table->foreignId('user_id')
      ->constrained()
      ->onDelete('cascade');
```

- Any additional column modifiers must be called before constrained:
```php
$table->foreignId('user_id')
      ->nullable()
      ->constrained();
```

- To drop a foreign key, you may use the `dropForeign` method, passing the foreign key constraint to be deleted as an argument. 
- Foreign key constraints use the same naming convention as indexes

```php
$table->dropForeign('posts_user_id');
```

- Alternatively, you may pass an array containing the column name that holds the foreign key to the dropForeign method. The array will be automatically converted using the constraint name convention used by Laravel's schema builder:
```php
- $table->dropForeign(['user_id']);
```

- You may enable or disable foreign key constraints within your migrations by using the following methods:

```php
Schema::enableForeignKeyConstraints();

Schema::disableForeignKeyConstraints();
```

**Note: SQLite disables foreign key constraints by default. When using SQLite, make sure to enable foreign key support in your database configuration before attempting to create them in your migrations. In addition, SQLite only supports foreign keys upon creation of the table and not when tables are altered.**

