# Events

* [Basic Usage](events.md#basic-usage)
* [Wildcard Listeners](events.md#wildcard-listeners)
* [Using Classes As Listeners](events.md#using-classes-as-listeners)
* [Queued Events](events.md#queued-events)
* [Event Subscribers](events.md#event-subscribers)

## Basic Usage

The Laravel `Event` class provides a simple observer implementation, allowing you to subscribe and listen for events in your application.

#### Subscribing To An Event

```text
Event::listen('auth.login', function($user)
{
    $user->last_login = new DateTime;

    $user->save();
});
```

#### Firing An Event

```text
$event = Event::fire('auth.login', array($user));
```

#### Subscribing To Events With Priority

You may also specify a priority when subscribing to events. Listeners with higher priority will be run first, while listeners that have the same priority will be run in order of subscription.

```text
Event::listen('auth.login', 'LoginHandler', 10);

Event::listen('auth.login', 'OtherHandler', 5);
```

#### Stopping The Propagation Of An Event

Sometimes, you may wish to stop the propagation of an event to other listeners. You may do so using by returning `false` from your listener:

```text
Event::listen('auth.login', function($event)
{
    // Handle the event...

    return false;
});
```

### Where To Register Events

So, you know how to register events, but you may be wondering _where_ to register them. Don't worry, this is a common question. Unfortunately, it's a hard question to answer because you can register an event almost anywhere! But, here are some tips. Again, like most other bootstrapping code, you may register events in one of your `start` files such as `app/start/global.php`.

If your `start` files are getting too crowded, you could create a separate `app/events.php` file that is included from a `start` file. This is a simple solution that keeps your event registration cleanly separated from the rest of your bootstrapping. If you prefer a class based approach, you may register your events in a [service provider](https://github.com/bryantyan/laravel4.2docs/tree/f12ffb53f9f16c3968c58e9dd508247dc98deb70/docs/ioc/README.md#service-providers). Since none of these approaches is inherently "correct", choose an approach you feel comfortable with based on the size of your application.

## Wildcard Listeners

#### Registering Wildcard Event Listeners

When registering an event listener, you may use asterisks to specify wildcard listeners:

```text
Event::listen('foo.*', function($param)
{
    // Handle the event...
});
```

This listener will handle all events that begin with `foo.`.

You may use the `Event::firing` method to determine exactly which event was fired:

```text
Event::listen('foo.*', function($param)
{
    if (Event::firing() == 'foo.bar')
    {
        //
    }
});
```

## Using Classes As Listeners

In some cases, you may wish to use a class to handle an event rather than a Closure. Class event listeners will be resolved out of the [Laravel IoC container](https://github.com/bryantyan/laravel4.2docs/tree/f12ffb53f9f16c3968c58e9dd508247dc98deb70/docs/ioc/README.md), providing you the full power of dependency injection on your listeners.

#### Registering A Class Listener

```text
Event::listen('auth.login', 'LoginHandler');
```

#### Defining An Event Listener Class

By default, the `handle` method on the `LoginHandler` class will be called:

```text
class LoginHandler {

    public function handle($data)
    {
        //
    }

}
```

#### Specifying Which Method To Subscribe

If you do not wish to use the default `handle` method, you may specify the method that should be subscribed:

```text
Event::listen('auth.login', 'LoginHandler@onLogin');
```

## Queued Events

#### Registering A Queued Event

Using the `queue` and `flush` methods, you may "queue" an event for firing, but not fire it immediately:

```text
Event::queue('foo', array($user));
```

#### Registering An Event Flusher

```text
Event::flusher('foo', function($user)
{
    //
});
```

Finally, you may run the "flusher" and flush all queued events using the `flush` method:

```text
Event::flush('foo');
```

## Event Subscribers

#### Defining An Event Subscriber

Event subscribers are classes that may subscribe to multiple events from within the class itself. Subscribers should define a `subscribe` method, which will be passed an event dispatcher instance:

```text
class UserEventHandler {

    /**
     * Handle user login events.
     */
    public function onUserLogin($event)
    {
        //
    }

    /**
     * Handle user logout events.
     */
    public function onUserLogout($event)
    {
        //
    }

    /**
     * Register the listeners for the subscriber.
     *
     * @param  Illuminate\Events\Dispatcher  $events
     * @return array
     */
    public function subscribe($events)
    {
        $events->listen('auth.login', 'UserEventHandler@onUserLogin');

        $events->listen('auth.logout', 'UserEventHandler@onUserLogout');
    }

}
```

#### Registering An Event Subscriber

Once the subscriber has been defined, it may be registered with the `Event` class.

```text
$subscriber = new UserEventHandler;

Event::subscribe($subscriber);
```

You may also use the [Laravel IoC container](https://github.com/bryantyan/laravel4.2docs/tree/f12ffb53f9f16c3968c58e9dd508247dc98deb70/docs/ioc/README.md) to resolve your subscriber. To do so, simply pass the name of your subscriber to the `subscribe` method:

```text
Event::subscribe('UserEventHandler');
```

