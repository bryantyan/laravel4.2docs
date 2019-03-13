# Routing

* [Basic Routing](routing.md#basic-routing)
* [Route Parameters](routing.md#route-parameters)
* [Route Filters](routing.md#route-filters)
* [Named Routes](routing.md#named-routes)
* [Route Groups](routing.md#route-groups)
* [Sub-Domain Routing](routing.md#sub-domain-routing)
* [Route Prefixing](routing.md#route-prefixing)
* [Route Model Binding](routing.md#route-model-binding)
* [Throwing 404 Errors](routing.md#throwing-404-errors)
* [Routing To Controllers](routing.md#routing-to-controllers)

## Basic Routing

Most of the routes for your application will be defined in the `app/routes.php` file. The simplest Laravel routes consist of a URI and a Closure callback.

### Basic GET Route

```text
Route::get('/', function()
{
    return 'Hello World';
});
```

### Basic POST Route

```text
Route::post('foo/bar', function()
{
    return 'Hello World';
});
```

### Registering A Route For Multiple Verbs

```text
Route::match(array('GET', 'POST'), '/', function()
{
    return 'Hello World';
});
```

### Registering A Route Responding To Any HTTP Verb

```text
Route::any('foo', function()
{
    return 'Hello World';
});
```

### Forcing A Route To Be Served Over HTTPS

```text
Route::get('foo', array('https', function()
{
    return 'Must be over HTTPS';
}));
```

Often, you will need to generate URLs to your routes, you may do so using the `URL::to` method:

```text
$url = URL::to('foo');
```

## Route Parameters

```text
Route::get('user/{id}', function($id)
{
    return 'User '.$id;
});
```

### Optional Route Parameters

```text
Route::get('user/{name?}', function($name = null)
{
    return $name;
});
```

### Optional Route Parameters With Defaults

```text
Route::get('user/{name?}', function($name = 'John')
{
    return $name;
});
```

### Regular Expression Route Constraints

```text
Route::get('user/{name}', function($name)
{
    //
})
->where('name', '[A-Za-z]+');

Route::get('user/{id}', function($id)
{
    //
})
->where('id', '[0-9]+');
```

### Passing An Array Of Wheres

Of course, you may pass an array of constraints when necessary:

```text
Route::get('user/{id}/{name}', function($id, $name)
{
    //
})
->where(array('id' => '[0-9]+', 'name' => '[a-z]+'))
```

### Defining Global Patterns

If you would like a route parameter to always be constrained by a given regular expression, you may use the `pattern` method:

```text
Route::pattern('id', '[0-9]+');

Route::get('user/{id}', function($id)
{
    // Only called if {id} is numeric.
});
```

### Accessing A Route Parameter Value

If you need to access a route parameter value outside of a route, you may use the `Route::input` method:

```text
Route::filter('foo', function()
{
    if (Route::input('id') == 1)
    {
        //
    }
});
```

## Route Filters

Route filters provide a convenient way of limiting access to a given route, which is useful for creating areas of your site which require authentication. There are several filters included in the Laravel framework, including an `auth` filter, an `auth.basic` filter, a `guest` filter, and a `csrf` filter. These are located in the `app/filters.php` file.

### Defining A Route Filter

```text
Route::filter('old', function()
{
    if (Input::get('age') < 200)
    {
        return Redirect::to('home');
    }
});
```

If the filter returns a response, that response is considered the response to the request and the route will not execute. Any `after` filters on the route are also cancelled.

### Attaching A Filter To A Route

```text
Route::get('user', array('before' => 'old', function()
{
    return 'You are over 200 years old!';
}));
```

### Attaching A Filter To A Controller Action

```text
Route::get('user', array('before' => 'old', 'uses' => 'UserController@showProfile'));
```

### Attaching Multiple Filters To A Route

```text
Route::get('user', array('before' => 'auth|old', function()
{
    return 'You are authenticated and over 200 years old!';
}));
```

### Attaching Multiple Filters Via Array

```text
Route::get('user', array('before' => array('auth', 'old'), function()
{
    return 'You are authenticated and over 200 years old!';
}));
```

### Specifying Filter Parameters

```text
Route::filter('age', function($route, $request, $value)
{
    //
});

Route::get('user', array('before' => 'age:200', function()
{
    return 'Hello World';
}));
```

After filters receive a `$response` as the third argument passed to the filter:

```text
Route::filter('log', function($route, $request, $response)
{
    //
});
```

### Pattern Based Filters

You may also specify that a filter applies to an entire set of routes based on their URI.

```text
Route::filter('admin', function()
{
    //
});

Route::when('admin/*', 'admin');
```

In the example above, the `admin` filter would be applied to all routes beginning with `admin/`. The asterisk is used as a wildcard, and will match any combination of characters.

You may also constrain pattern filters by HTTP verbs:

```text
Route::when('admin/*', 'admin', array('post'));
```

### Filter Classes

For advanced filtering, you may wish to use a class instead of a Closure. Since filter classes are resolved out of the application [IoC Container](https://github.com/bryantyan/laravel4.2docs/tree/f12ffb53f9f16c3968c58e9dd508247dc98deb70/docs/ioc/README.md), you will be able to utilize dependency injection in these filters for greater testability.

### Registering A Class Based Filter

```text
Route::filter('foo', 'FooFilter');
```

By default, the `filter` method on the `FooFilter` class will be called:

```text
class FooFilter {

    public function filter()
    {
        // Filter logic...
    }

}
```

If you do not wish to use the `filter` method, just specify another method:

```text
Route::filter('foo', 'FooFilter@foo');
```

## Named Routes

Named routes make referring to routes when generating redirects or URLs more convenient. You may specify a name for a route like so:

```text
Route::get('user/profile', array('as' => 'profile', function()
{
    //
}));
```

You may also specify route names for controller actions:

```text
Route::get('user/profile', array('as' => 'profile', 'uses' => 'UserController@showProfile'));
```

Now, you may use the route's name when generating URLs or redirects:

```text
$url = URL::route('profile');

$redirect = Redirect::route('profile');
```

You may access the name of a route that is running via the `currentRouteName` method:

```text
$name = Route::currentRouteName();
```

## Route Groups

Sometimes you may need to apply filters to a group of routes. Instead of specifying the filter on each route, you may use a route group:

```text
Route::group(array('before' => 'auth'), function()
{
    Route::get('/', function()
    {
        // Has Auth Filter
    });

    Route::get('user/profile', function()
    {
        // Has Auth Filter
    });
});
```

You may also use the `namespace` parameter within your `group` array to specify all controllers within that group as being in a given namespace:

```text
Route::group(array('namespace' => 'Admin'), function()
{
    //
});
```

## Sub-Domain Routing

Laravel routes are also able to handle wildcard sub-domains, and pass you wildcard parameters from the domain:

### Registering Sub-Domain Routes

```text
Route::group(array('domain' => '{account}.myapp.com'), function()
{

    Route::get('user/{id}', function($account, $id)
    {
        //
    });

});
```

## Route Prefixing

A group of routes may be prefixed by using the `prefix` option in the attributes array of a group:

```text
Route::group(array('prefix' => 'admin'), function()
{

    Route::get('user', function()
    {
        //
    });

});
```

## Route Model Binding

Model binding provides a convenient way to inject model instances into your routes. For example, instead of injecting a user's ID, you can inject the entire User model instance that matches the given ID. First, use the `Route::model` method to specify the model that should be used for a given parameter:

### Binding A Parameter To A Model

```text
Route::model('user', 'User');
```

Next, define a route that contains a `{user}` parameter:

```text
Route::get('profile/{user}', function(User $user)
{
    //
});
```

Since we have bound the `{user}` parameter to the `User` model, a `User` instance will be injected into the route. So, for example, a request to `profile/1` will inject the `User` instance which has an ID of 1.

> **Note:** If a matching model instance is not found in the database, a 404 error will be thrown.

If you wish to specify your own "not found" behavior, you may pass a Closure as the third argument to the `model` method:

```text
Route::model('user', 'User', function()
{
    throw new NotFoundHttpException;
});
```

Sometimes you may wish to use your own resolver for route parameters. Simply use the `Route::bind` method:

```text
Route::bind('user', function($value, $route)
{
    return User::where('name', $value)->first();
});
```

## Throwing 404 Errors

There are two ways to manually trigger a 404 error from a route. First, you may use the `App::abort` method:

```text
App::abort(404);
```

Second, you may throw an instance of `Symfony\Component\HttpKernel\Exception\NotFoundHttpException`.

More information on handling 404 exceptions and using custom responses for these errors may be found in the [errors](https://github.com/bryantyan/laravel4.2docs/tree/f12ffb53f9f16c3968c58e9dd508247dc98deb70/docs/errors/README.md#handling-404-errors) section of the documentation.

## Routing To Controllers

Laravel allows you to not only route to Closures, but also to controller classes, and even allows the creation of [resource controllers](https://github.com/bryantyan/laravel4.2docs/tree/f12ffb53f9f16c3968c58e9dd508247dc98deb70/docs/controllers/README.md#resource-controllers).

See the documentation on [Controllers](https://github.com/bryantyan/laravel4.2docs/tree/f12ffb53f9f16c3968c58e9dd508247dc98deb70/docs/controllers/README.md) for more details.

