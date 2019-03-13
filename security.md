# Security

* [Configuration](security.md#configuration)
* [Storing Passwords](security.md#storing-passwords)
* [Authenticating Users](security.md#authenticating-users)
* [Manually Logging In Users](security.md#manually)
* [Protecting Routes](security.md#protecting-routes)
* [HTTP Basic Authentication](security.md#http-basic-authentication)
* [Password Reminders & Reset](security.md#password-reminders-and-reset)
* [Encryption](security.md#encryption)
* [Authentication Drivers](security.md#authentication-drivers)

## Configuration

Laravel aims to make implementing authentication very simple. In fact, almost everything is configured for you out of the box. The authentication configuration file is located at `app/config/auth.php`, which contains several well documented options for tweaking the behavior of the authentication facilities.

By default, Laravel includes a `User` model in your `app/models` directory which may be used with the default Eloquent authentication driver. Please remember when building the Schema for this Model to ensure that the password field is a minimum of 60 characters.

If your application is not using Eloquent, you may use the `database` authentication driver which uses the Laravel query builder.

> **Note:** Before getting started, make sure that your `users` \(or equivalent\) table contains a nullable, string `remember_token` column of 100 characters. This column will be used to store a token for "remember me" sessions being maintained by your application.

## Storing Passwords

The Laravel `Hash` class provides secure Bcrypt hashing:

#### Hashing A Password Using Bcrypt

```text
$password = Hash::make('secret');
```

#### Verifying A Password Against A Hash

```text
if (Hash::check('secret', $hashedPassword))
{
    // The passwords match...
}
```

#### Checking If A Password Needs To Be Rehashed

```text
if (Hash::needsRehash($hashed))
{
    $hashed = Hash::make('secret');
}
```

## Authenticating Users

To log a user into your application, you may use the `Auth::attempt` method.

```text
if (Auth::attempt(array('email' => $email, 'password' => $password)))
{
    return Redirect::intended('dashboard');
}
```

Take note that `email` is not a required option, it is merely used for example. You should use whatever column name corresponds to a "username" in your database. The `Redirect::intended` function will redirect the user to the URL they were trying to access before being caught by the authentication filter. A fallback URI may be given to this method in case the intended destination is not available.

When the `attempt` method is called, the `auth.attempt` [event](https://github.com/bryantyan/laravel4.2docs/tree/f12ffb53f9f16c3968c58e9dd508247dc98deb70/docs/events/README.md) will be fired. If the authentication attempt is successful and the user is logged in, the `auth.login` event will be fired as well.

#### Determining If A User Is Authenticated

To determine if the user is already logged into your application, you may use the `check` method:

```text
if (Auth::check())
{
    // The user is logged in...
}
```

#### Authenticating A User And "Remembering" Them

If you would like to provide "remember me" functionality in your application, you may pass `true` as the second argument to the `attempt` method, which will keep the user authenticated indefinitely \(or until they manually logout\). Of course, your `users` table must include the string `remember_token` column, which will be used to store the "remember me" token.

```text
if (Auth::attempt(array('email' => $email, 'password' => $password), true))
{
    // The user is being remembered...
}
```

**Note:** If the `attempt` method returns `true`, the user is considered logged into the application.

#### Determining If User Authed Via Remember

If you are "remembering" user logins, you may use the `viaRemember` method to determine if the user was authenticated using the "remember me" cookie:

```text
if (Auth::viaRemember())
{
    //
}
```

#### Authenticating A User With Conditions

You also may add extra conditions to the authenticating query:

```text
if (Auth::attempt(array('email' => $email, 'password' => $password, 'active' => 1)))
{
    // The user is active, not suspended, and exists.
}
```

> **Note:** For added protection against session fixation, the user's session ID will automatically be regenerated after authenticating.

#### Accessing The Logged In User

Once a user is authenticated, you may access the User model / record:

```text
$email = Auth::user()->email;
```

To retrieve the authenticated user's ID, you may use the `id` method:

```text
$id = Auth::id();
```

To simply log a user into the application by their ID, use the `loginUsingId` method:

```text
Auth::loginUsingId(1);
```

#### Validating User Credentials Without Login

The `validate` method allows you to validate a user's credentials without actually logging them into the application:

```text
if (Auth::validate($credentials))
{
    //
}
```

#### Logging A User In For A Single Request

You may also use the `once` method to log a user into the application for a single request. No sessions or cookies will be utilized.

```text
if (Auth::once($credentials))
{
    //
}
```

#### Logging A User Out Of The Application

```text
Auth::logout();
```

## Manually Logging In Users

If you need to log an existing user instance into your application, you may simply call the `login` method with the instance:

```text
$user = User::find(1);

Auth::login($user);
```

This is equivalent to logging in a user via credentials using the `attempt` method.

## Protecting Routes

Route filters may be used to allow only authenticated users to access a given route. Laravel provides the `auth` filter by default, and it is defined in `app/filters.php`.

#### Protecting A Route

```text
Route::get('profile', array('before' => 'auth', function()
{
    // Only authenticated users may enter...
}));
```

### CSRF Protection

Laravel provides an easy method of protecting your application from cross-site request forgeries.

#### Inserting CSRF Token Into Form

#### Validate The Submitted CSRF Token

```text
Route::post('register', array('before' => 'csrf', function()
{
    return 'You gave a valid CSRF token!';
}));
```

## HTTP Basic Authentication

HTTP Basic Authentication provides a quick way to authenticate users of your application without setting up a dedicated "login" page. To get started, attach the `auth.basic` filter to your route:

#### Protecting A Route With HTTP Basic

```text
Route::get('profile', array('before' => 'auth.basic', function()
{
    // Only authenticated users may enter...
}));
```

By default, the `basic` filter will use the `email` column on the user record when authenticating. If you wish to use another column you may pass the column name as the first parameter to the `basic` method in your `app/filters.php` file:

```text
Route::filter('auth.basic', function()
{
    return Auth::basic('username');
});
```

#### Setting Up A Stateless HTTP Basic Filter

You may also use HTTP Basic Authentication without setting a user identifier cookie in the session, which is particularly useful for API authentication. To do so, define a filter that returns the `onceBasic` method:

```text
Route::filter('basic.once', function()
{
    return Auth::onceBasic();
});
```

If you are using PHP FastCGI, HTTP Basic authentication will not work correctly by default. The following lines should be added to your `.htaccess` file:

```text
RewriteCond %{HTTP:Authorization} ^(.+)$
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
```

## Password Reminders & Reset

### Model & Table

Most web applications provide a way for users to reset their forgotten passwords. Rather than forcing you to re-implement this on each application, Laravel provides convenient methods for sending password reminders and performing password resets. To get started, verify that your `User` model implements the `Illuminate\Auth\Reminders\RemindableInterface` contract. Of course, the `User` model included with the framework already implements this interface, and uses the `Illuminate\Auth\Reminders\RemindableTrait` to include the methods needed to implement the interface.

#### Implementing The RemindableInterface

```text
use Illuminate\Auth\Reminders\RemindableTrait;
use Illuminate\Auth\Reminders\RemindableInterface;

class User extends Eloquent implements RemindableInterface {

    use RemindableTrait;

}
```

#### Generating The Reminder Table Migration

Next, a table must be created to store the password reset tokens. To generate a migration for this table, simply execute the `auth:reminders-table` Artisan command:

```text
php artisan auth:reminders-table

php artisan migrate
```

### Password Reminder Controller

Now we're ready to generate the password reminder controller. To automatically generate a controller, you may use the `auth:reminders-controller` Artisan command, which will create a `RemindersController.php` file in your `app/controllers` directory.

```text
php artisan auth:reminders-controller
```

The generated controller will already have a `getRemind` method that handles showing your password reminder form. All you need to do is create a `password.remind` [view](https://github.com/bryantyan/laravel4.2docs/tree/f12ffb53f9f16c3968c58e9dd508247dc98deb70/docs/responses/README.md#views). This view should have a basic form with an `email` field. The form should POST to the `RemindersController@postRemind` action.

A simple form on the `password.remind` view might look like this:

In addition to `getRemind`, the generated controller will already have a `postRemind` method that handles sending the password reminder e-mails to your users. This method expects the `email` field to be present in the `POST` variables. If the reminder e-mail is successfully sent to the user, a `status` message will be flashed to the session. If the reminder fails, an `error` message will be flashed instead.

Within the `postRemind` controller method you may modify the message instance before it is sent to the user:

```text
Password::remind(Input::only('email'), function($message)
{
    $message->subject('Password Reminder');
});
```

Your user will receive an e-mail with a link that points to the `getReset` method of the controller. The password reminder token, which is used to identify a given password reminder attempt, will also be passed to the controller method. The action is already configured to return a `password.reset` view which you should build. The `token` will be passed to the view, and you should place this token in a hidden form field named `token`. In addition to the `token`, your password reset form should contain `email`, `password`, and `password_confirmation` fields. The form should POST to the `RemindersController@postReset` method.

A simple form on the `password.reset` view might look like this:

Finally, the `postReset` method is responsible for actually changing the password in storage. In this controller action, the Closure passed to the `Password::reset` method sets the `password` attribute on the `User` and calls the `save` method. Of course, this Closure is assuming your `User` model is an [Eloquent model](https://github.com/bryantyan/laravel4.2docs/tree/f12ffb53f9f16c3968c58e9dd508247dc98deb70/docs/eloquent/README.md); however, you are free to change this Closure as needed to be compatible with your application's database storage system.

If the password is successfully reset, the user will be redirected to the root of your application. Again, you are free to change this redirect URL. If the password reset fails, the user will be redirect back to the reset form, and an `error` message will be flashed to the session.

### Password Validation

By default, the `Password::reset` method will verify that the passwords match and are &gt;= six characters. You may customize these rules using the `Password::validator` method, which accepts a Closure. Within this Closure, you may do any password validation you wish. Note that you are not required to verify that the passwords match, as this will be done automatically by the framework.

```text
Password::validator(function($credentials)
{
    return strlen($credentials['password']) >= 6;
});
```

> **Note:** By default, password reset tokens expire after one hour. You may change this via the `reminder.expire` option of your `app/config/auth.php` file.

## Encryption

Laravel provides facilities for strong AES encryption via the mcrypt PHP extension:

#### Encrypting A Value

```text
$encrypted = Crypt::encrypt('secret');
```

> **Note:** Be sure to set a 16, 24, or 32 character random string in the `key` option of the `app/config/app.php` file. Otherwise, encrypted values will not be secure.

#### Decrypting A Value

```text
$decrypted = Crypt::decrypt($encryptedValue);
```

#### Setting The Cipher & Mode

You may also set the cipher and mode used by the encrypter:

```text
Crypt::setMode('ctr');

Crypt::setCipher($cipher);
```

## Authentication Drivers

Laravel offers the `database` and `eloquent` authentication drivers out of the box. For more information about adding additional authentication drivers, check out the [Authentication extension documentation](https://github.com/bryantyan/laravel4.2docs/tree/f12ffb53f9f16c3968c58e9dd508247dc98deb70/docs/extending/README.md#authentication).

