# Session

* [Configuration](session.md#configuration)
* [Session Usage](session.md#session-usage)
* [Flash Data](session.md#flash-data)
* [Database Sessions](session.md#database-sessions)
* [Session Drivers](session.md#session-drivers)

## Configuration

Since HTTP driven applications are stateless, sessions provide a way to store information about the user across requests. Laravel ships with a variety of session back-ends available for use through a clean, unified API. Support for popular back-ends such as [Memcached](http://memcached.org), [Redis](http://redis.io), and databases is included out of the box.

The session configuration is stored in `app/config/session.php`. Be sure to review the well documented options available to you in this file. By default, Laravel is configured to use the `file` session driver, which will work well for the majority of applications.

### Reserved Keys

The Laravel framework uses the `flash` session key internally, so you should not add an item to the session by that name.

## Session Usage

### Storing An Item In The Session

```text
Session::put('key', 'value');
```

### Push A Value Onto An Array Session Value

```text
Session::push('user.teams', 'developers');
```

### Retrieving An Item From The Session

```text
$value = Session::get('key');
```

### Retrieving An Item Or Returning A Default Value

```text
$value = Session::get('key', 'default');

$value = Session::get('key', function() { return 'default'; });
```

### Retrieving An Item And Forgetting It

```text
$value = Session::pull('key', 'default');
```

### Retrieving All Data From The Session

```text
$data = Session::all();
```

### Determining If An Item Exists In The Session

```text
if (Session::has('users'))
{
    //
}
```

### Removing An Item From The Session

```text
Session::forget('key');
```

### Removing All Items From The Session

```text
Session::flush();
```

### Regenerating The Session ID

```text
Session::regenerate();
```

## Flash Data

Sometimes you may wish to store items in the session only for the next request. You may do so using the `Session::flash` method:

```text
Session::flash('key', 'value');
```

### Reflashing The Current Flash Data For Another Request

```text
Session::reflash();
```

### Reflashing Only A Subset Of Flash Data

```text
Session::keep(array('username', 'email'));
```

## Database Sessions

When using the `database` session driver, you will need to setup a table to contain the session items. Below is an example `Schema` declaration for the table:

```text
Schema::create('sessions', function($table)
{
    $table->string('id')->unique();
    $table->text('payload');
    $table->integer('last_activity');
});
```

Of course, you may use the `session:table` Artisan command to generate this migration for you!

```text
php artisan session:table

composer dump-autoload

php artisan migrate
```

## Session Drivers

The session "driver" defines where session data will be stored for each request. Laravel ships with several great drivers out of the box:

* `file` - sessions will be stored in `app/storage/sessions`.
* `cookie` - sessions will be stored in secure, encrypted cookies.
* `database` - sessions will be stored in a database used by your application.
* `memcached` / `redis` - sessions will be stored in one of these fast, cached based stores.
* `array` - sessions will be stored in a simple PHP array and will not be persisted across requests.

> **Note:** The array driver is typically used for running [unit tests](https://github.com/bryantyan/laravel4.2docs/tree/f12ffb53f9f16c3968c58e9dd508247dc98deb70/docs/testing/README.md), so no session data will be persisted.

