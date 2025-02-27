# Unit Testing

* [Introduction](testing.md#introduction)
* [Defining & Running Tests](testing.md#defining-and-running-tests)
* [Test Environment](testing.md#test-environment)
* [Calling Routes From Tests](testing.md#calling-routes-from-tests)
* [Mocking Facades](testing.md#mocking-facades)
* [Framework Assertions](testing.md#framework-assertions)
* [Helper Methods](testing.md#helper-methods)
* [Refreshing The Application](testing.md#refreshing-the-application)

## Introduction

Laravel is built with unit testing in mind. In fact, support for testing with PHPUnit is included out of the box, and a `phpunit.xml` file is already setup for your application. In addition to PHPUnit, Laravel also utilizes the Symfony HttpKernel, DomCrawler, and BrowserKit components to allow you to inspect and manipulate your views while testing, allowing to simulate a web browser.

An example test file is provided in the `app/tests` directory. After installing a new Laravel application, simply run `phpunit` on the command line to run your tests.

## Defining & Running Tests

To create a test case, simply create a new test file in the `app/tests` directory. The test class should extend `TestCase`. You may then define test methods as you normally would when using PHPUnit.

#### An Example Test Class

```text
class FooTest extends TestCase {

    public function testSomethingIsTrue()
    {
        $this->assertTrue(true);
    }

}
```

You may run all of the tests for your application by executing the `phpunit` command from your terminal.

> **Note:** If you define your own `setUp` method, be sure to call `parent::setUp`.

## Test Environment

When running unit tests, Laravel will automatically set the configuration environment to `testing`. Also, Laravel includes configuration files for `session` and `cache` in the test environment. Both of these drivers are set to `array` while in the test environment, meaning no session or cache data will be persisted while testing. You are free to create other testing environment configurations as necessary.

## Calling Routes From Tests

#### Calling A Route From A Test

You may easily call one of your routes for a test using the `call` method:

```text
$response = $this->call('GET', 'user/profile');

$response = $this->call($method, $uri, $parameters, $files, $server, $content);
```

You may then inspect the `Illuminate\Http\Response` object:

```text
$this->assertEquals('Hello World', $response->getContent());
```

#### Calling A Controller From A Test

You may also call a controller from a test:

```text
$response = $this->action('GET', 'HomeController@index');

$response = $this->action('GET', 'UserController@profile', array('user' => 1));
```

The `getContent` method will return the evaluated string contents of the response. If your route returns a `View`, you may access it using the `original` property:

```text
$view = $response->original;

$this->assertEquals('John', $view['name']);
```

To call a HTTPS route, you may use the `callSecure` method:

```text
$response = $this->callSecure('GET', 'foo/bar');
```

> **Note:** Route filters are disabled when in the testing environment. To enable them, add `Route::enableFilters()` to your test.

### DOM Crawler

You may also call a route and receive a DOM Crawler instance that you may use to inspect the content:

```text
$crawler = $this->client->request('GET', '/');

$this->assertTrue($this->client->getResponse()->isOk());

$this->assertCount(1, $crawler->filter('h1:contains("Hello World!")'));
```

For more information on how to use the crawler, refer to its [official documentation](http://symfony.com/doc/master/components/dom_crawler.html).

## Mocking Facades

When testing, you may often want to mock a call to a Laravel static facade. For example, consider the following controller action:

```text
public function getIndex()
{
    Event::fire('foo', array('name' => 'Dayle'));

    return 'All done!';
}
```

We can mock the call to the `Event` class by using the `shouldReceive` method on the facade, which will return an instance of a [Mockery](https://github.com/padraic/mockery) mock.

#### Mocking A Facade

```text
public function testGetIndex()
{
    Event::shouldReceive('fire')->once()->with('foo', array('name' => 'Dayle'));

    $this->call('GET', '/');
}
```

> **Note:** You should not mock the `Request` facade. Instead, pass the input you desire into the `call` method when running your test.

## Framework Assertions

Laravel ships with several `assert` methods to make testing a little easier:

#### Asserting Responses Are OK

```text
public function testMethod()
{
    $this->call('GET', '/');

    $this->assertResponseOk();
}
```

#### Asserting Response Statuses

```text
$this->assertResponseStatus(403);
```

#### Asserting Responses Are Redirects

```text
$this->assertRedirectedTo('foo');

$this->assertRedirectedToRoute('route.name');

$this->assertRedirectedToAction('Controller@method');
```

#### Asserting A View Has Some Data

```text
public function testMethod()
{
    $this->call('GET', '/');

    $this->assertViewHas('name');
    $this->assertViewHas('age', $value);
}
```

#### Asserting The Session Has Some Data

```text
public function testMethod()
{
    $this->call('GET', '/');

    $this->assertSessionHas('name');
    $this->assertSessionHas('age', $value);
}
```

#### Asserting The Session Has Errors

```text
public function testMethod()
{
    $this->call('GET', '/');

    $this->assertSessionHasErrors();

    // Asserting the session has errors for a given key...
    $this->assertSessionHasErrors('name');

    // Asserting the session has errors for several keys...
    $this->assertSessionHasErrors(array('name', 'age'));
}
```

#### Asserting Old Input Has Some Data

```text
public function testMethod()
{
    $this->call('GET', '/');

    $this->assertHasOldInput();
}
```

## Helper Methods

The `TestCase` class contains several helper methods to make testing your application easier.

#### Setting And Flushing Sessions From Tests

```text
$this->session(['foo' => 'bar']);

$this->flushSession();
```

#### Setting The Currently Authenticated User

You may set the currently authenticated user using the `be` method:

```text
$user = new User(array('name' => 'John'));

$this->be($user);
```

You may re-seed your database from a test using the `seed` method:

#### Re-Seeding Database From Tests

```text
$this->seed();

$this->seed($connection);
```

More information on creating seeds may be found in the [migrations and seeding](https://github.com/bryantyan/laravel4.2docs/tree/f12ffb53f9f16c3968c58e9dd508247dc98deb70/docs/migrations/README.md#database-seeding) section of the documentation.

## Refreshing The Application

As you may already know, you can access your Laravel `Application` / IoC Container via `$this->app` from any test method. This Application instance is refreshed for each test class. If you wish to manually force the Application to be refreshed for a given method, you may use the `refreshApplication` method from your test method. This will reset any extra bindings, such as mocks, that have been placed in the IoC container since the test case started running.

