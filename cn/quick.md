# quick

## Laravel 快速入门

* [安装](quick.md#installation)
* [路由](quick.md#routing)
* [创建视图](quick.md#creating-a-view)
* [创建迁移](quick.md#creating-a-migration)
* [Eloquent ORM](quick.md#eloquent-orm)
* [显示数据](quick.md#displaying-data)

### 安装

#### 通过 Laravel Installer

首先，下载\[Laravel installer PHAR 归档\]。为了方便，重名文件为`laravel`并且移动到`/usr/local/bin`目录。一旦安装完成，使用简单的`laravel new`命令将会在特定目录下创建一个全新的laravel环境。例如，`laravel new blog`将创建`blog`目录，里面包含有所有laravel安装及依赖。该方法比通过Composer安装要快。

#### 通过 Composer

Laravel使用[Composer](http://getcomposer.org)执行安装和管理依赖。如果你还没有安装，从[安装 Composer](http://getcomposer.org/doc/00-intro.md)开始吧。

现在你可以在终端使用如下命令来安装Laravel:

```text
composer create-project laravel/laravel your-project-name --prefer-dist
```

该命令将下载以及安装全新的Laravel拷贝，存放在`your-project-name`目录。

如果你喜欢的话，你还可以手动拷贝[Laravel Github 仓库](https://github.com/laravel/laravel/archive/master.zip)来代替安装。接着在手动创建项目根目录下执行`composer install`命令。该命令将会下载并安装框架依赖。

#### 权限

安装完Laravel，你需要确保web服务器具有`app/storage`目录写入权限。查看[安装](https://github.com/bryantyan/laravel4.2docs/tree/f12ffb53f9f16c3968c58e9dd508247dc98deb70/docs/installation/README.md)文档了解配置细节。

#### Serving Laravel

Typically, you may use a web server such as Apache or Nginx to serve your Laravel applications. If you are on PHP 5.4+ and would like to use PHP's built-in development server, you may use the `serve` Artisan command:

```text
php artisan serve
```

#### Directory Structure

After installing the framework, take a glance around the project to familiarize yourself with the directory structure. The `app` directory contains folders such as `views`, `controllers`, and `models`. Most of your application's code will reside somewhere in this directory. You may also wish to explore the `app/config` directory and the configuration options that are available to you.

### Routing

To get started, let's create our first route. In Laravel, the simplest route is a route to a Closure. Pop open the `app/routes.php` file and add the following route to the bottom of the file:

```text
Route::get('users', function()
{
    return 'Users!';
});
```

Now, if you hit the `/users` route in your web browser, you should see `Users!` displayed as the response. Great! You've just created your first route.

Routes can also be attached to controller classes. For example:

```text
Route::get('users', 'UserController@getIndex');
```

This route informs the framework that requests to the `/users` route should call the `getIndex` method on the `UserController` class. For more information on controller routing, check out the [controller documentation](https://github.com/bryantyan/laravel4.2docs/tree/f12ffb53f9f16c3968c58e9dd508247dc98deb70/docs/controllers/README.md).

### Creating A View

Next, we'll create a simple view to display our user data. Views live in the `app/views` directory and contain the HTML of your application. We're going to place two new views in this directory: `layout.blade.php` and `users.blade.php`. First, let's create our `layout.blade.php` file:

## Laravel Quickstart

@yield\('content'\)

Next, we'll create our `users.blade.php` view:

```text
@extends('layout')

@section('content')
    Users!
@stop
```

Some of this syntax probably looks quite strange to you. That's because we're using Laravel's templating system: Blade. Blade is very fast, because it is simply a handful of regular expressions that are run against your templates to compile them to pure PHP. Blade provides powerful functionality like template inheritance, as well as some syntax sugar on typical PHP control structures such as `if` and `for`. Check out the [Blade documentation](https://github.com/bryantyan/laravel4.2docs/tree/f12ffb53f9f16c3968c58e9dd508247dc98deb70/docs/templates/README.md) for more details.

Now that we have our views, let's return it from our `/users` route. Instead of returning `Users!` from the route, return the view instead:

```text
Route::get('users', function()
{
    return View::make('users');
});
```

Wonderful! Now you have setup a simple view that extends a layout. Next, let's start working on our database layer.

### Creating A Migration

To create a table to hold our data, we'll use the Laravel migration system. Migrations let you expressively define modifications to your database, and easily share them with the rest of your team.

First, let's configure a database connection. You may configure all of your database connections from the `app/config/database.php` file. By default, Laravel is configured to use MySQL, and you will need to supply connection credentials within the database configuration file. If you wish, you may change the `driver` option to `sqlite` and it will use the SQLite database included in the `app/database` directory.

Next, to create the migration, we'll use the [Artisan CLI](https://github.com/bryantyan/laravel4.2docs/tree/f12ffb53f9f16c3968c58e9dd508247dc98deb70/docs/artisan/README.md). From the root of your project, run the following from your terminal:

```text
php artisan migrate:make create_users_table
```

Next, find the generated migration file in the `app/database/migrations` folder. This file contains a class with two methods: `up` and `down`. In the `up` method, you should make the desired changes to your database tables, and in the `down` method you simply reverse them.

Let's define a migration that looks like this:

```text
public function up()
{
    Schema::create('users', function($table)
    {
        $table->increments('id');
        $table->string('email')->unique();
        $table->string('name');
        $table->timestamps();
    });
}

public function down()
{
    Schema::drop('users');
}
```

Next, we can run our migrations from our terminal using the `migrate` command. Simply execute this command from the root of your project:

```text
php artisan migrate
```

If you wish to rollback a migration, you may issue the `migrate:rollback` command. Now that we have a database table, let's start pulling some data!

### Eloquent ORM

Laravel ships with a superb ORM: Eloquent. If you have used the Ruby on Rails framework, you will find Eloquent familiar, as it follows the ActiveRecord ORM style of database interaction.

First, let's define a model. An Eloquent model can be used to query an associated database table, as well as represent a given row within that table. Don't worry, it will all make sense soon! Models are typically stored in the `app/models` directory. Let's define a `User.php` model in that directory like so:

```text
class User extends Eloquent {}
```

Note that we do not have to tell Eloquent which table to use. Eloquent has a variety of conventions, one of which is to use the plural form of the model name as the model's database table. Convenient!

Using your preferred database administration tool, insert a few rows into your `users` table, and we'll use Eloquent to retrieve them and pass them to our view.

Now let's modify our `/users` route to look like this:

```text
Route::get('users', function()
{
    $users = User::all();

    return View::make('users')->with('users', $users);
});
```

Let's walk through this route. First, the `all` method on the `User` model will retrieve all of the rows in the `users` table. Next, we're passing these records to the view via the `with` method. The `with` method accepts a key and a value, and is used to make a piece of data available to a view.

Awesome. Now we're ready to display the users in our view!

### Displaying Data

Now that we have made the `users` available to our view, we can display them like so:

```text
@extends('layout')

@section('content')
    @foreach($users as $user)
        <p>{{ $user->name }}</p>
    @endforeach
@stop
```

You may be wondering where to find our `echo` statements. When using Blade, you may echo data by surrounding it with double curly braces. It's a cinch. Now, you should be able to hit the `/users` route and see the names of your users displayed in the response.

This is just the beginning. In this tutorial, you've seen the very basics of Laravel, but there are so many more exciting things to learn. Keep reading through the documentation and dig deeper into the powerful features available to you in [Eloquent](https://github.com/bryantyan/laravel4.2docs/tree/f12ffb53f9f16c3968c58e9dd508247dc98deb70/docs/eloquent/README.md) and [Blade](https://github.com/bryantyan/laravel4.2docs/tree/f12ffb53f9f16c3968c58e9dd508247dc98deb70/docs/templates/README.md). Or, maybe you're more interested in [Queues](https://github.com/bryantyan/laravel4.2docs/tree/f12ffb53f9f16c3968c58e9dd508247dc98deb70/docs/queues/README.md) and [Unit Testing](https://github.com/bryantyan/laravel4.2docs/tree/f12ffb53f9f16c3968c58e9dd508247dc98deb70/docs/testing/README.md). Then again, maybe you want to flex your architecture muscles with the [IoC Container](https://github.com/bryantyan/laravel4.2docs/tree/f12ffb53f9f16c3968c58e9dd508247dc98deb70/docs/ioc/README.md). The choice is yours!

