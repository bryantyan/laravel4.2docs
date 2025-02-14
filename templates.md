# Templates

* [Controller Layouts](templates.md#controller-layouts)
* [Blade Templating](templates.md#blade-templating)
* [Other Blade Control Structures](templates.md#other-blade-control-structures)
* [Extending Blade](templates.md#extending-blade)

## Controller Layouts

One method of using templates in Laravel is via controller layouts. By specifying the `layout` property on the controller, the view specified will be created for you and will be the assumed response that should be returned from actions.

### Defining A Layout On A Controller

```text
class UserController extends BaseController {

    /**
     * The layout that should be used for responses.
     */
    protected $layout = 'layouts.master';

    /**
     * Show the user profile.
     */
    public function showProfile()
    {
        $this->layout->content = View::make('user.profile');
    }

}
```

## Blade Templating

Blade is a simple, yet powerful templating engine provided with Laravel. Unlike controller layouts, Blade is driven by _template inheritance_ and _sections_. All Blade templates should use the `.blade.php` extension.

### Defining A Blade Layout

@section\('sidebar'\) This is the master sidebar. @show @yield\('content'\)

### Using A Blade Layout

```text
@extends('layouts.master')

@section('sidebar')
    @parent

    <p>This is appended to the master sidebar.</p>
@stop

@section('content')
    <p>This is my body content.</p>
@stop
```

Note that views which `extend` a Blade layout simply override sections from the layout. Content of the layout can be included in a child view using the `@parent` directive in a section, allowing you to append to the contents of a layout section such as a sidebar or footer.

Sometimes, such as when you are not sure if a section has been defined, you may wish to pass a default value to the `@yield` directive. You may pass the default value as the second argument:

```text
@yield('section', 'Default Content');
```

## Other Blade Control Structures

### Echoing Data

```text
Hello, {{{ $name }}}.

The current UNIX timestamp is {{{ time() }}}.
```

### Echoing Data After Checking For Existence

Sometimes you may wish to echo a variable, but you aren't sure if the variable has been set. Basically, you want to do this:

```text
{{{ isset($name) ? $name : 'Default' }}}
```

However, instead of writing a ternary statement, Blade allows you to use the following convenient short-cut:

```text
{{{ $name or 'Default' }}}
```

### Displaying Raw Text With Curly Braces

If you need to display a string that is wrapped in curly braces, you may escape the Blade behavior by prefixing your text with an `@` symbol:

```text
@{{ This will not be processed by Blade }}
```

Of course, all user supplied data should be escaped or purified. To escape the output, you may use the triple curly brace syntax:

```text
Hello, {{{ $name }}}.
```

If you don't want the data to be escaped, you may use double curly-braces:

```text
Hello, {{ $name }}.
```

> **Note:** Be very careful when echoing content that is supplied by users of your application. Always use the triple curly brace syntax to escape any HTML entities in the content.

### If Statements

```text
@if (count($records) === 1)
    I have one record!
@elseif (count($records) > 1)
    I have multiple records!
@else
    I don't have any records!
@endif

@unless (Auth::check())
    You are not signed in.
@endunless
```

### Loops

```text
@for ($i = 0; $i < 10; $i++)
    The current value is {{ $i }}
@endfor

@foreach ($users as $user)
    <p>This is user {{ $user->id }}</p>
@endforeach

@while (true)
    <p>I'm looping forever.</p>
@endwhile
```

### Including Sub-Views

```text
@include('view.name')
```

You may also pass an array of data to the included view:

```text
@include('view.name', array('some'=>'data'))
```

### Overwriting Sections

By default, sections are appended to any previous content that exists in the section. To overwrite a section entirely, you may use the `overwrite` statement:

```text
@extends('list.item.container')

@section('list.item.content')
    <p>This is an item of type {{ $item->type }}</p>
@overwrite
```

### Displaying Language Lines

```text
@lang('language.line')

@choice('language.line', 1);
```

### Comments

```text
{{-- This comment will not be in the rendered HTML --}}
```

## Extending Blade

Blade even allows you to define your own custom control structures. When a Blade file is compiled, each custom extension is called with the view contents, allowing you to do anything from simple `str_replace` manipulations to more complex regular expressions.

The Blade compiler comes with the helper methods `createMatcher` and `createPlainMatcher`, which generate the expression you need to build your own custom directives.

The `createPlainMatcher` method is used for directives with no arguments like `@endif` and `@stop`, while `createMatcher` is used for directives with arguments.

The following example creates a `@datetime($var)` directive which simply calls `->format()` on `$var`:

```text
Blade::extend(function($view, $compiler)
{
    $pattern = $compiler->createMatcher('datetime');

    return preg_replace($pattern, '$1<?php echo $2->format('m/d/Y H:i'); ?>', $view);
});
```

