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

##**Introduction**

"Laravel's database query builder provides a convenient, fluent interface to creating and running database queries. It can be used to perform most database operations in your application and works on all supported database systems."

- Laravel query builder uses PDO parameter binding to protect your application against SQL injection attacks

** Note:
PDO does not support binding column names. Therefore, you should never allow user input to dictate the column names referenced by your queries, including "order by" columns, etc. If you must allow the user to select certain columns to query against, always validate the column names against a white-list of allowed columns.
 **



Link: https://laravel.com/docs/7.x/queries
Position: Retrieving Results
