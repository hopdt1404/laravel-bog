#Eloquent: Relationships

####**Introduction**
####**Defining Relationships**
    + One To One
    + One To Many
    + One To Many (Inverse)
    + Many To Many
    + Defining Custom Intermediate Table Models
    + Has One Through
    + Has Many Through
####**Polymorphic Relationships**
    + One To One
    + One To Many
    + Many To Many
    + Custom Polymorphic Types
####**Dynamic Relationships**
####**Querying Relations**
    + Relationship Methods Vs. Dynamic Properties
    + Querying Relationship Existence
    + Querying Relationship Absence
    + Querying Polymorphic Relationships
    + Counting Related Models
    + Counting Related Models On Polymorphic Relationships
####**Eager Loading**
    + Constraining Eager Loads
    + Lazy Eager Loading
####**Inserting & Updating Related Models**
    + The save Method
    + The create Method
    + Belongs To Relationships
    + Many To Many Relationships
####**Touching Parent Timestamps**

##**#Introduction**

- Database tables are often related to one another. 
- For example, a blog post may have many comments, or an order could be related to the user who placed it. 
- Eloquent makes managing and working with these relationships easy, and supports several different types of relationships:
    + One To One
    + One To Many
    + Many To Many
    + Has One Through
    + Has Many Through
    + One To One (Polymorphic)
    + One To Many (Polymorphic)
    + Many To Many (Polymorphic)

##**#Defining Relationships**

- Eloquent relationships are defined as methods on your Eloquent model classes.
- Since, like Eloquent models themselves, relationships also serve as powerful query builders, defining relationships as methods provides powerful method chaining and querying capabilities. 
- For example, we may chain additional constraints on this `posts` relationship:
```php
$user->posts()->where('active', 1)->get();
```

- But, before diving too deep into using relationships, let's learn how to define each type.

**Note: Relationship names cannot collide with attribute names as that could lead to your model not being able to know which one to resolve.**

###**#One To One**

- A one-to-one relationship is a very basic relation.
- For example, a `User` model might be associated with one `Phone`.
- To define this relationship, we place a `phone` method on the `User` model. 
- The phone method should call the hasOne method and return its result

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Get the phone record associated with the user.
     */
    public function phone()
    {
        return $this->hasOne('App\Phone');
    }
}
```

**Note:

- Realationships phát huy tác dụng: ta có thể implement and query between tables without join
- 

**

- The first argument passed to the `hasOne` method is the name of the related model. 
- Once the relationship is defined, we may retrieve the related record using Eloquent's dynamic properties. 
- Dynamic properties allow you to access relationship methods as if they were properties defined on the model:

```php
$phone = User::find(1)->phone;
```

- Eloquent determines the foreign key of the relationship based on the model name. 
- In this case, the `Phone` model is automatically assumed to have a `user_id` foreign key. 
- If you wish to override this convention, you may pass a second argument to the hasOne method:

```php
return $this->hasOne('App\Phone', 'foreign_key');
```
**Note: Chỉ cần điền khóa phụ của bảng Phone match với khóa chính của bảng User**

- Additionally, Eloquent assumes that the foreign key should have a value matching the `id` (or the custom `$primaryKey`) column of the parent. 
- In other words, Eloquent will look for the value of the user's `id` column in the `user_id` column of the `Phone` record. 
- If you would like the relationship to use a value other than `id`, you may pass a third argument to the `hasOne` method specifying your custom key:

```php
return $this->hasOne('App\Phone', 'foreign_key', 'local_key');
```

####**Defining The Inverse Of The Relationship**

- So, we can access the `Phone` model from our `User`. 
- Now, let's define a relationship on the `Phone` model that will let us access the `User` that owns the phone. 
- We can define the inverse of a `hasOne` relationship using the `belongsTo` method:

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Phone extends Model
{
    /**
     * Get the user that owns the phone.
     */
    public function user()
    {
        return $this->belongsTo('App\User');
    }
}
```

- In the example above, Eloquent will try to match the `user_id` from the `Phone` model to an id on the User model. 
- Eloquent determines the default foreign key name by examining the name of the relationship method and suffixing the method name with `_id`. 
- However, if the foreign key on the Phone model is not `user_id`, you may pass a custom key name as the second argument to the belongsTo method:

```php
/**
 * Get the user that owns the phone.
 */
public function user()
{
    return $this->belongsTo('App\User', 'foreign_key');
}
```

- If your parent model does not use `id` as its primary key, or you wish to join the child model to a different column, you may pass a third argument to the `belongsTo` method specifying your parent table's custom key:

```php
/**
 * Get the user that owns the phone.
 */
public function user()
{
    return $this->belongsTo('App\User', 'foreign_key', 'other_key');
}
```

##**#One To Many**

- A one-to-many relationship is used to define relationships where a single model owns any amount of other models. 
- For example, a blog post may have an infinite number of comments. 
- Like all other Eloquent relationships, one-to-many relationships are defined by placing a function on your Eloquent model:

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    /**
     * Get the comments for the blog post.
     */
    public function comments()
    {
        return $this->hasMany('App\Comment');
    }
}
```

- Remember, Eloquent will automatically determine the proper foreign key column on the `Comment` model. 
- By convention, Eloquent will take the "snake case" name of the owning model and suffix it with `_id`
- So, for this example, Eloquent will assume the foreign key on the Comment model is `post_id`.

- Once the relationship has been defined, we can access the collection of comments by accessing the `comments` property. 
- Remember, since Eloquent provides "dynamic properties", we can access relationship methods as if they were defined as properties on the model:

```php
$comments = App\Post::find(1)->comments;

foreach ($comments as $comment) {
    //
}
```

- Since all relationships also serve as query builders, you can add further constraints to which comments are retrieved by calling the `comments` method and continuing to chain conditions onto the query:

```php
$comment = App\Post::find(1)->comments()->where('title', 'foo')->first();
```

- Like the `hasOne` method, you may also override the foreign and local keys by passing additional arguments to the `hasMany` method:

```php
return $this->hasMany('App\Comment', 'foreign_key');

return $this->hasMany('App\Comment', 'foreign_key', 'local_key');
```

###**#One To Many (Inverse)**

- Now that we can access all of a post's comments, let's define a relationship to allow a comment to access its parent post. 
- To define the inverse of a `hasMany` relationship, define a relationship function on the child model which calls the `belongsTo` method:

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Comment extends Model
{
    /**
     * Get the post that owns the comment.
     */
    public function post()
    {
        return $this->belongsTo('App\Post');
    }
}
```

- Once the relationship has been defined, we can retrieve the `Post` model for a Comment by accessing the post "dynamic property":

```php
$comment = App\Comment::find(1);

echo $comment->post->title;
```

- In the example above, Eloquent will try to match the `post_id` from the `Comment` model to an `id` on the `Post` model. 
- Eloquent determines the default foreign key name by examining the name of the relationship method and suffixing the method name with a `_id` followed by the name of the primary key column. 
- However, if the foreign key on the Comment model is not `post_id`, you may pass a custom key name as the second argument to the `belongsTo` method

```php

/**
 * Get the post that owns the comment.
 */
public function post()
{
    return $this->belongsTo('App\Post', 'foreign_key');
}
```

- If your parent model does not use `id` as its primary key, or you wish to join the child model to a different column, you may pass a third argument to the `belongsTo` method specifying your parent table's custom key:

```php
/**
 * Get the post that owns the comment.
 */
public function post()
{
    return $this->belongsTo('App\Post', 'foreign_key', 'other_key');
}
```

###**#Many To Many**

- Many-to-many relations are slightly more complicated than `hasOne` and `hasMany` relationships. 
- An example of such a relationship is a user with many roles, where the roles are also shared by other users. 
- For example, many users may have the role of "Admin".

#####**Table Structure**

- To define this relationship, three database tables are needed: `users`, `roles`, and `role_user`. 
- The `role_user` table is derived from the alphabetical order of the related model names, and contains the `user_id` and `role_id` columns:

```
users
    id - integer
    name - string

roles
    id - integer
    name - string

role_user
    user_id - integer
    role_id - integer
```

####**Model Structure**

- Many-to-many relationships are defined by writing a method that returns the result of the `belongsToMany` method. 
- For example, let's define the `roles` method on our `User` model:

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * The roles that belong to the user.
     */
    public function roles()
    {
        return $this->belongsToMany('App\Role');
    }
}
```

- Once the relationship is defined, you may access the user's `roles` using the roles dynamic property:

```php
$user = App\User::find(1);

foreach ($user->roles as $role) {
    //
}
```

- Like all other relationship types, you may call the `roles` method to continue chaining query constraints onto the relationship:

```php
$roles = App\User::find(1)->roles()->orderBy('name')->get();
```

Link: https://laravel.com/docs/7.x/eloquent-relationships#one-to-one
position: As mentioned previously, to determine the table name of the relationship's joining
