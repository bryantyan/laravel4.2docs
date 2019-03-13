# Migrations & Seeding

* [Introduction](migrations.md#introduction)
* [Creating Migrations](migrations.md#creating-migrations)
* [Running Migrations](migrations.md#running-migrations)
* [Rolling Back Migrations](migrations.md#rolling-back-migrations)
* [Database Seeding](migrations.md#database-seeding)

## Introduction

Migrations are a type of version control for your database. They allow a team to modify the database schema and stay up to date on the current schema state. Migrations are typically paired with the [Schema Builder](https://github.com/bryantyan/laravel4.2docs/tree/f12ffb53f9f16c3968c58e9dd508247dc98deb70/docs/schema/README.md) to easily manage your application's scheme.

## Creating Migrations

To create a migration, you may use the `migrate:make` command on the Artisan CLI:

```text
php artisan migrate:make create_users_table
```

The migration will be placed in your `app/database/migrations` folder, and will contain a timestamp which allows the framework to determine the order of the migrations.

You may also specify a `--path` option when creating the migration. The path should be relative to the root directory of your installation:

```text
php artisan migrate:make foo --path=app/migrations
```

The `--table` and `--create` options may also be used to indicate the name of the table, and whether the migration will be creating a new table:

```text
php artisan migrate:make add_votes_to_user_table --table=users

php artisan migrate:make create_users_table --create=users
```

## Running Migrations

#### Running All Outstanding Migrations

```text
php artisan migrate
```

#### Running All Outstanding Migrations For A Path

```text
php artisan migrate --path=app/foo/migrations
```

#### Running All Outstanding Migrations For A Package

```text
php artisan migrate --package=vendor/package
```

> **Note:** If you receive a "class not found" error when running migrations, try running the `composer dump-autoload` command.

### Forcing Migrations In Production

Some migration operations are destructive, meaning they may cause you to lose data. In order to protect you from running these commands against your production database, you will prompted for confirmation before these commands are executed. To force thse commands to run without a prompt, use the `--force` flag:

```text
php artisan migrate --force
```

## Rolling Back Migrations

#### Rollback The Last Migration Operation

```text
php artisan migrate:rollback
```

#### Rollback all migrations

```text
php artisan migrate:reset
```

#### Rollback all migrations and run them all again

```text
php artisan migrate:refresh

php artisan migrate:refresh --seed
```

## Database Seeding

Laravel also includes a simple way to seed your database with test data using seed classes. All seed classes are stored in `app/database/seeds`. Seed classes may have any name you wish, but probably should follow some sensible convention, such as `UserTableSeeder`, etc. By default, a `DatabaseSeeder` class is defined for you. From this class, you may use the `call` method to run other seed classes, allowing you to control the seeding order.

#### Example Database Seed Class

```text
class DatabaseSeeder extends Seeder {

    public function run()
    {
        $this->call('UserTableSeeder');

        $this->command->info('User table seeded!');
    }

}

class UserTableSeeder extends Seeder {

    public function run()
    {
        DB::table('users')->delete();

        User::create(array('email' => 'foo@bar.com'));
    }

}
```

To seed your database, you may use the `db:seed` command on the Artisan CLI:

```text
php artisan db:seed
```

By default, the `db:seed` command runs the `DatabaseSeeder` class, which may be used to call other seed classes. However, you may use the `--class` option to specify a specific seeder class to run individually:

```text
php artisan db:seed --class=UserTableSeeder
```

You may also seed your database using the `migrate:refresh` command, which will also rollback and re-run all of your migrations:

```text
php artisan migrate:refresh --seed
```

