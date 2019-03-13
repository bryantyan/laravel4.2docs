# Query Builder

* [Introduction](queries.md#introduction)
* [Selects](queries.md#selects)
* [Joins](queries.md#joins)
* [Advanced Wheres](queries.md#advanced-wheres)
* [Aggregates](queries.md#aggregates)
* [Raw Expressions](queries.md#raw-expressions)
* [Inserts](queries.md#inserts)
* [Updates](queries.md#updates)
* [Deletes](queries.md#deletes)
* [Unions](queries.md#unions)
* [Pessimistic Locking](queries.md#pessimistic-locking)
* [Caching Queries](queries.md#caching-queries)

## Introduction

The database query builder provides a convenient, fluent interface to creating and running database queries. It can be used to perform most database operations in your application, and works on all supported database systems.

> **Note:** The Laravel query builder uses PDO parameter binding throughout to protect your application against SQL injection attacks. There is no need to clean strings being passed as bindings.

## Selects

### Retrieving All Rows From A Table

```text
$users = DB::table('users')->get();

foreach ($users as $user)
{
    var_dump($user->name);
}
```

### Retrieving A Single Row From A Table

```text
$user = DB::table('users')->where('name', 'John')->first();

var_dump($user->name);
```

### Retrieving A Single Column From A Row

```text
$name = DB::table('users')->where('name', 'John')->pluck('name');
```

### Retrieving A List Of Column Values

```text
$roles = DB::table('roles')->lists('title');
```

This method will return an array of role titles. You may also specify a custom key column for the returned array:

```text
$roles = DB::table('roles')->lists('title', 'name');
```

### Specifying A Select Clause

```text
$users = DB::table('users')->select('name', 'email')->get();

$users = DB::table('users')->distinct()->get();

$users = DB::table('users')->select('name as user_name')->get();
```

### Adding A Select Clause To An Existing Query

```text
$query = DB::table('users')->select('name');

$users = $query->addSelect('age')->get();
```

### Using Where Operators

```text
$users = DB::table('users')->where('votes', '>', 100)->get();
```

### Or Statements

```text
$users = DB::table('users')
                    ->where('votes', '>', 100)
                    ->orWhere('name', 'John')
                    ->get();
```

### Using Where Between

```text
$users = DB::table('users')
                    ->whereBetween('votes', array(1, 100))->get();
```

### Using Where Not Between

```text
$users = DB::table('users')
                    ->whereNotBetween('votes', array(1, 100))->get();
```

### Using Where In With An Array

```text
$users = DB::table('users')
                    ->whereIn('id', array(1, 2, 3))->get();

$users = DB::table('users')
                    ->whereNotIn('id', array(1, 2, 3))->get();
```

### Using Where Null To Find Records With Unset Values

```text
$users = DB::table('users')
                    ->whereNull('updated_at')->get();
```

### Order By, Group By, And Having

```text
$users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->groupBy('count')
                    ->having('count', '>', 100)
                    ->get();
```

### Offset & Limit

```text
$users = DB::table('users')->skip(10)->take(5)->get();
```

## Joins

The query builder may also be used to write join statements. Take a look at the following examples:

### Basic Join Statement

```text
DB::table('users')
            ->join('contacts', 'users.id', '=', 'contacts.user_id')
            ->join('orders', 'users.id', '=', 'orders.user_id')
            ->select('users.id', 'contacts.phone', 'orders.price')
            ->get();
```

### Left Join Statement

```text
DB::table('users')
        ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
        ->get();
```

You may also specify more advanced join clauses:

```text
DB::table('users')
        ->join('contacts', function($join)
        {
            $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
        })
        ->get();
```

If you would like to use a "where" style clause on your joins, you may use the `where` and `orWhere` methods on a join. Instead of comparing two columns, these methods will compare the column against a value:

```text
DB::table('users')
        ->join('contacts', function($join)
        {
            $join->on('users.id', '=', 'contacts.user_id')
                 ->where('contacts.user_id', '>', 5);
        })
        ->get();
```

## Advanced Wheres

### Parameter Grouping

Sometimes you may need to create more advanced where clauses such as "where exists" or nested parameter groupings. The Laravel query builder can handle these as well:

```text
DB::table('users')
            ->where('name', '=', 'John')
            ->orWhere(function($query)
            {
                $query->where('votes', '>', 100)
                      ->where('title', '<>', 'Admin');
            })
            ->get();
```

The query above will produce the following SQL:

```text
select * from users where name = 'John' or (votes > 100 and title <> 'Admin')
```

### Exists Statements

```text
DB::table('users')
            ->whereExists(function($query)
            {
                $query->select(DB::raw(1))
                      ->from('orders')
                      ->whereRaw('orders.user_id = users.id');
            })
            ->get();
```

The query above will produce the following SQL:

```text
select * from users
where exists (
    select 1 from orders where orders.user_id = users.id
)
```

## Aggregates

The query builder also provides a variety of aggregate methods, such as `count`, `max`, `min`, `avg`, and `sum`.

### Using Aggregate Methods

```text
$users = DB::table('users')->count();

$price = DB::table('orders')->max('price');

$price = DB::table('orders')->min('price');

$price = DB::table('orders')->avg('price');

$total = DB::table('users')->sum('votes');
```

## Raw Expressions

Sometimes you may need to use a raw expression in a query. These expressions will be injected into the query as strings, so be careful not to create any SQL injection points! To create a raw expression, you may use the `DB::raw` method:

### Using A Raw Expression

```text
$users = DB::table('users')
                     ->select(DB::raw('count(*) as user_count, status'))
                     ->where('status', '<>', 1)
                     ->groupBy('status')
                     ->get();
```

### Incrementing or decrementing a value of a column

```text
DB::table('users')->increment('votes');

DB::table('users')->increment('votes', 5);

DB::table('users')->decrement('votes');

DB::table('users')->decrement('votes', 5);
```

You may also specify additional columns to update:

```text
DB::table('users')->increment('votes', 1, array('name' => 'John'));
```

## Inserts

### Inserting Records Into A Table

```text
DB::table('users')->insert(
    array('email' => 'john@example.com', 'votes' => 0)
);
```

### Inserting Records Into A Table With An Auto-Incrementing ID

If the table has an auto-incrementing id, use `insertGetId` to insert a record and retrieve the id:

```text
$id = DB::table('users')->insertGetId(
    array('email' => 'john@example.com', 'votes' => 0)
);
```

> **Note:** When using PostgreSQL the insertGetId method expects the auto-incrementing column to be named "id".

### Inserting Multiple Records Into A Table

```text
DB::table('users')->insert(array(
    array('email' => 'taylor@example.com', 'votes' => 0),
    array('email' => 'dayle@example.com', 'votes' => 0),
));
```

## Updates

### Updating Records In A Table

```text
DB::table('users')
            ->where('id', 1)
            ->update(array('votes' => 1));
```

## Deletes

### Deleting Records In A Table

```text
DB::table('users')->where('votes', '<', 100)->delete();
```

### Deleting All Records From A Table

```text
DB::table('users')->delete();
```

### Truncating A Table

```text
DB::table('users')->truncate();
```

## Unions

The query builder also provides a quick way to "union" two queries together:

```text
$first = DB::table('users')->whereNull('first_name');

$users = DB::table('users')->whereNull('last_name')->union($first)->get();
```

The `unionAll` method is also available, and has the same method signature as `union`.

## Pessimistic Locking

The query builder includes a few functions to help you do "pessimistic locking" on your SELECT statements.

To run the SELECT statement with a "shared lock", you may use the `sharedLock` method on a query:

```text
DB::table('users')->where('votes', '>', 100)->sharedLock()->get();
```

To "lock for update" on a SELECT statement, you may use the `lockForUpdate` method on a query:

```text
DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();
```

## Caching Queries

You may easily cache the results of a query using the `remember` method:

```text
$users = DB::table('users')->remember(10)->get();
```

In this example, the results of the query will be cached for ten minutes. While the results are cached, the query will not be run against the database, and the results will be loaded from the default cache driver specified for your application.

If you are using a [supported cache driver](https://github.com/bryantyan/laravel4.2docs/tree/f12ffb53f9f16c3968c58e9dd508247dc98deb70/docs/cache/README.md#cache-tags), you can also add tags to the caches:

```text
$users = DB::table('users')->cacheTags(array('people', 'authors'))->remember(10)->get();
```

