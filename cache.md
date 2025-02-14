# Cache

* [Configuration](cache.md#configuration)
* [Cache Usage](cache.md#cache-usage)
* [Increments & Decrements](cache.md#increments-and-decrements)
* [Cache Tags](cache.md#cache-tags)
* [Database Cache](cache.md#database-cache)

## Configuration

Laravel provides a unified API for various caching systems. The cache configuration is located at `app/config/cache.php`. In this file you may specify which cache driver you would like used by default throughout your application. Laravel supports popular caching backends like [Memcached](http://memcached.org) and [Redis](http://redis.io) out of the box.

The cache configuration file also contains various other options, which are documented within the file, so make sure to read over these options. By default, Laravel is configured to use the `file` cache driver, which stores the serialized, cached objects in the filesystem. For larger applications, it is recommended that you use an in-memory cache such as Memcached or APC.

## Cache Usage

### Storing An Item In The Cache

```text
Cache::put('key', 'value', $minutes);
```

### Using Carbon Objects To Set Expire Time

```text
$expiresAt = Carbon::now()->addMinutes(10);

Cache::put('key', 'value', $expiresAt);
```

### Storing An Item In The Cache If It Doesn't Exist

```text
Cache::add('key', 'value', $minutes);
```

The `add` method will return `true` if the item is actually **added** to the cache. Otherwise, the method will return `false`.

### Checking For Existence In Cache

```text
if (Cache::has('key'))
{
    //
}
```

### Retrieving An Item From The Cache

```text
$value = Cache::get('key');
```

### Retrieving An Item Or Returning A Default Value

```text
$value = Cache::get('key', 'default');

$value = Cache::get('key', function() { return 'default'; });
```

### Storing An Item In The Cache Permanently

```text
Cache::forever('key', 'value');
```

Sometimes you may wish to retrieve an item from the cache, but also store a default value if the requested item doesn't exist. You may do this using the `Cache::remember` method:

```text
$value = Cache::remember('users', $minutes, function()
{
    return DB::table('users')->get();
});
```

You may also combine the `remember` and `forever` methods:

```text
$value = Cache::rememberForever('users', function()
{
    return DB::table('users')->get();
});
```

Note that all items stored in the cache are serialized, so you are free to store any type of data.

### Pulling An Item From The Cache

If you need to retrieve an item from the cache and then delete it, you may use the `pull` method:

```text
$value = Cache::pull('key');
```

### Removing An Item From The Cache

```text
Cache::forget('key');
```

## Increments & Decrements

All drivers except `file` and `database` support the `increment` and `decrement` operations:

### Incrementing A Value

```text
Cache::increment('key');

Cache::increment('key', $amount);
```

### Decrementing A Value

```text
Cache::decrement('key');

Cache::decrement('key', $amount);
```

## Cache Tags

> **Note:** Cache tags are not supported when using the `file` or `database` cache drivers. Furthermore, when using multiple tags with caches that are stored "forever", performance will be best with a driver such as `memcached`, which automatically purges stale records.

### Accessing A Tagged Cache

Cache tags allow you to tag related items in the cache, and then flush all caches tagged with a given name. To access a tagged cache, use the `tags` method.

You may store a tagged cache by passing in an ordered list of tag names as arguments, or as an ordered array of tag names:

```text
Cache::tags('people', 'authors')->put('John', $john, $minutes);

Cache::tags(array('people', 'artists'))->put('Anne', $anne, $minutes);
```

You may use any cache storage method in combination with tags, including `remember`, `forever`, and `rememberForever`. You may also access cached items from the tagged cache, as well as use the other cache methods such as `increment` and `decrement`.

### Accessing Items In A Tagged Cache

To access a tagged cache, pass the same ordered list of tags used to save it.

```text
$anne = Cache::tags('people', 'artists')->get('Anne');

$john = Cache::tags(array('people', 'authors'))->get('John');
```

You may flush all items tagged with a name or list of names. For example, this statement would remove all caches tagged with either `people`, `authors`, or both. So, both "Anne" and "John" would be removed from the cache:

```text
Cache::tags('people', 'authors')->flush();
```

In contrast, this statement would remove only caches tagged with `authors`, so "John" would be removed, but not "Anne".

```text
Cache::tags('authors')->flush();
```

## Database Cache

When using the `database` cache driver, you will need to setup a table to contain the cache items. You'll find an example `Schema` declaration for the table below:

```text
Schema::create('cache', function($table)
{
    $table->string('key')->unique();
    $table->text('value');
    $table->integer('expiration');
});
```

