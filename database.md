# Basic Database Usage

* [Configuration](database.md#configuration)
* [Read / Write Connections](database.md#read-write-connections)
* [Running Queries](database.md#running-queries)
* [Database Transactions](database.md#database-transactions)
* [Accessing Connections](database.md#accessing-connections)
* [Query Logging](database.md#query-logging)

## Configuration

Laravel makes connecting with databases and running queries extremely simple. The database configuration file is `app/config/database.php`. In this file you may define all of your database connections, as well as specify which connection should be used by default. Examples for all of the supported database systems are provided in this file.

Currently Laravel supports four database systems: MySQL, Postgres, SQLite, and SQL Server.

## Read / Write Connections

Sometimes you may wish to use one database connection for SELECT statements, and another for INSERT, UPDATE, and DELETE statements. Laravel makes this a breeze, and the proper connections will always be used whether you are using raw queries, the query builder, or the Eloquent ORM.

To see how read / write connections should be configured, let's look at this example:

```text
'mysql' => array(
    'read' => array(
        'host' => '192.168.1.1',
    ),
    'write' => array(
        'host' => '196.168.1.2'
    ),
    'driver'    => 'mysql',
    'database'  => 'database',
    'username'  => 'root',
    'password'  => '',
    'charset'   => 'utf8',
    'collation' => 'utf8_unicode_ci',
    'prefix'    => '',
),
```

Note that two keys have been added to the configuration array: `read` and `write`. Both of these keys have array values containing a single key: `host`. The rest of the database options for the `read` and `write` connections will be merged from the main `mysql` array. So, we only need to place items in the `read` and `write` arrays if we wish to override the values in the main array. So, in this case, `192.168.1.1` will be used as the "read" connection, while `192.168.1.2` will be used as the "write" connection. The database credentials, prefix, character set, and all other options in the main `mysql` array will be shared across both connections.

## Running Queries

Once you have configured your database connection, you may run queries using the `DB` class.

### Running A Select Query

```text
$results = DB::select('select * from users where id = ?', array(1));
```

The `select` method will always return an `array` of results.

### Running An Insert Statement

```text
DB::insert('insert into users (id, name) values (?, ?)', array(1, 'Dayle'));
```

### Running An Update Statement

```text
DB::update('update users set votes = 100 where name = ?', array('John'));
```

### Running A Delete Statement

```text
DB::delete('delete from users');
```

> **Note:** The `update` and `delete` statements return the number of rows affected by the operation.

### Running A General Statement

```text
DB::statement('drop table users');
```

### Listening For Query Events

You may listen for query events using the `DB::listen` method:

```text
DB::listen(function($sql, $bindings, $time)
{
    //
});
```

## Database Transactions

To run a set of operations within a database transaction, you may use the `transaction` method:

```text
DB::transaction(function()
{
    DB::table('users')->update(array('votes' => 1));

    DB::table('posts')->delete();
});
```

> **Note:** Any exception thrown within the `transaction` closure will cause the transaction to be rolled back automatically.

Sometimes you may need to begin a transaction yourself:

```text
DB::beginTransaction();
```

You can rollback a transaction via the `rollback` method:

```text
DB::rollback();
```

Lastly, you can commit a transaction via the `commit` method:

```text
DB::commit();
```

## Accessing Connections

When using multiple connections, you may access them via the `DB::connection` method:

```text
$users = DB::connection('foo')->select(...);
```

You may also access the raw, underlying PDO instance:

```text
$pdo = DB::connection()->getPdo();
```

Sometimes you may need to reconnect to a given database:

```text
DB::reconnect('foo');
```

If you need to disconnect from the given database due to exceeding the underlying PDO instance's `max_connections` limit, use the `disconnect` method:

```text
DB::disconnect('foo');
```

## Query Logging

By default, Laravel keeps a log in memory of all queries that have been run for the current request. However, in some cases, such as when inserting a large number of rows, this can cause the application to use excess memory. To disable the log, you may use the `disableQueryLog` method:

```text
DB::connection()->disableQueryLog();
```

To get an array of the executed queries, you may use the `getQueryLog` method:

```text
   $queries = DB::getQueryLog();
```

