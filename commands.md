# Artisan Development

* [Introduction](commands.md#introduction)
* [Building A Command](commands.md#building-a-command)
* [Registering Commands](commands.md#registering-commands)
* [Calling Other Commands](commands.md#calling-other-commands)

## Introduction

In addition to the commands provided with Artisan, you may also build your own custom commands for working with your application. You may store your custom commands in the `app/commands` directory; however, you are free to choose your own storage location as long as your commands can be autoloaded based on your `composer.json` settings.

## Building A Command

### Generating The Class

To create a new command, you may use the `command:make` Artisan command, which will generate a command stub to help you get started:

#### Generate A New Command Class

```text
php artisan command:make FooCommand
```

By default, generated commands will be stored in the `app/commands` directory; however, you may specify custom path or namespace:

```text
php artisan command:make FooCommand --path=app/classes --namespace=Classes
```

When creating the command, the `--command` option may be used to assign the terminal command name:

```text
php artisan command:make AssignUsers --command=users:assign
```

### Writing The Command

Once your command is generated, you should fill out the `name` and `description` properties of the class, which will be used when displaying your command on the `list` screen.

The `fire` method will be called when your command is executed. You may place any command logic in this method.

### Arguments & Options

The `getArguments` and `getOptions` methods are where you may define any arguments or options your command receives. Both of these methods return an array of commands, which are described by a list of array options.

When defining `arguments`, the array definition values represent the following:

```text
array($name, $mode, $description, $defaultValue)
```

The argument `mode` may be any of the following: `InputArgument::REQUIRED` or `InputArgument::OPTIONAL`.

When defining `options`, the array definition values represent the following:

```text
array($name, $shortcut, $mode, $description, $defaultValue)
```

For options, the argument `mode` may be: `InputOption::VALUE_REQUIRED`, `InputOption::VALUE_OPTIONAL`, `InputOption::VALUE_IS_ARRAY`, `InputOption::VALUE_NONE`.

The `VALUE_IS_ARRAY` mode indicates that the switch may be used multiple times when calling the command:

```text
php artisan foo --option=bar --option=baz
```

The `VALUE_NONE` option indicates that the option is simply used as a "switch":

```text
php artisan foo --option
```

### Retrieving Input

While your command is executing, you will obviously need to access the values for the arguments and options accepted by your application. To do so, you may use the `argument` and `option` methods:

#### Retrieving The Value Of A Command Argument

```text
$value = $this->argument('name');
```

#### Retrieving All Arguments

```text
$arguments = $this->argument();
```

#### Retrieving The Value Of A Command Option

```text
$value = $this->option('name');
```

#### Retrieving All Options

```text
$options = $this->option();
```

### Writing Output

To send output to the console, you may use the `info`, `comment`, `question` and `error` methods. Each of these methods will use the appropriate ANSI colors for their purpose.

#### Sending Information To The Console

```text
$this->info('Display this on the screen');
```

#### Sending An Error Message To The Console

```text
$this->error('Something went wrong!');
```

### Asking Questions

You may also use the `ask` and `confirm` methods to prompt the user for input:

#### Asking The User For Input

```text
$name = $this->ask('What is your name?');
```

#### Asking The User For Secret Input

```text
$password = $this->secret('What is the password?');
```

#### Asking The User For Confirmation

```text
if ($this->confirm('Do you wish to continue? [yes|no]'))
{
    //
}
```

You may also specify a default value to the `confirm` method, which should be `true` or `false`:

```text
$this->confirm($question, true);
```

## Registering Commands

#### Registering An Artisan Command

Once your command is finished, you need to register it with Artisan so it will be available for use. This is typically done in the `app/start/artisan.php` file. Within this file, you may use the `Artisan::add` method to register the command:

```text
Artisan::add(new CustomCommand);
```

#### Registering A Command That Is In The IoC Container

If your command is registered in the application [IoC container](https://github.com/bryantyan/laravel4.2docs/tree/f12ffb53f9f16c3968c58e9dd508247dc98deb70/docs/ioc/README.md), you may use the `Artisan::resolve` method to make it available to Artisan:

```text
Artisan::resolve('binding.name');
```

## Calling Other Commands

Sometimes you may wish to call other commands from your command. You may do so using the `call` method:

```text
$this->call('command:name', array('argument' => 'foo', '--option' => 'bar'));
```

