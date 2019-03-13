# Forms & HTML

* [Opening A Form](html.md#opening-a-form)
* [CSRF Protection](html.md#csrf-protection)
* [Form Model Binding](html.md#form-model-binding)
* [Labels](html.md#labels)
* [Text, Text Area, Password & Hidden Fields](html.md#text)
* [Checkboxes and Radio Buttons](html.md#checkboxes-and-radio-buttons)
* [File Input](html.md#file-input)
* [Drop-Down Lists](html.md#drop-down-lists)
* [Buttons](html.md#buttons)
* [Custom Macros](html.md#custom-macros)
* [Generating URLs](html.md#generating-urls)

## Opening A Form

### Opening A Form

```text
{{ Form::open(array('url' => 'foo/bar')) }}
    //
{{ Form::close() }}
```

By default, a `POST` method will be assumed; however, you are free to specify another method:

```text
echo Form::open(array('url' => 'foo/bar', 'method' => 'put'))
```

> **Note:** Since HTML forms only support `POST` and `GET`, `PUT` and `DELETE` methods will be spoofed by automatically adding a `_method` hidden field to your form.

You may also open forms that point to named routes or controller actions:

```text
echo Form::open(array('route' => 'route.name'))

echo Form::open(array('action' => 'Controller@method'))
```

You may pass in route parameters as well:

```text
echo Form::open(array('route' => array('route.name', $user->id)))

echo Form::open(array('action' => array('Controller@method', $user->id)))
```

If your form is going to accept file uploads, add a `files` option to your array:

```text
echo Form::open(array('url' => 'foo/bar', 'files' => true))
```

## CSRF Protection

### Adding The CSRF Token To A Form

Laravel provides an easy method of protecting your application from cross-site request forgeries. First, a random token is placed in your user's session. Don't sweat it, this is done automatically. The CSRF token will be added to your forms as a hidden field automatically. However, if you wish to generate the HTML for the hidden field, you may use the `token` method:

```text
echo Form::token();
```

### Attaching The CSRF Filter To A Route

```text
Route::post('profile', array('before' => 'csrf', function()
{
    //
}));
```

## Form Model Binding

### Opening A Model Form

Often, you will want to populate a form based on the contents of a model. To do so, use the `Form::model` method:

```text
echo Form::model($user, array('route' => array('user.update', $user->id)))
```

Now, when you generate a form element, like a text input, the model's value matching the field's name will automatically be set as the field value. So, for example, for a text input named `email`, the user model's `email` attribute would be set as the value. However, there's more! If there is an item in the Session flash data matching the input name, that will take precedence over the model's value. So, the priority looks like this:

1. Session Flash Data \(Old Input\)
2. Explicitly Passed Value
3. Model Attribute Data

This allows you to quickly build forms that not only bind to model values, but easily re-populate if there is a validation error on the server!

> **Note:** When using `Form::model`, be sure to close your form with `Form::close`!

## Labels

### Generating A Label Element

```text
echo Form::label('email', 'E-Mail Address');
```

### Specifying Extra HTML Attributes

```text
echo Form::label('email', 'E-Mail Address', array('class' => 'awesome'));
```

> **Note:** After creating a label, any form element you create with a name matching the label name will automatically receive an ID matching the label name as well.

## Text, Text Area, Password & Hidden Fields

### Generating A Text Input

```text
echo Form::text('username');
```

### Specifying A Default Value

```text
echo Form::text('email', 'example@gmail.com');
```

> **Note:** The _hidden_ and _textarea_ methods have the same signature as the _text_ method.

### Generating A Password Input

```text
echo Form::password('password');
```

### Generating Other Inputs

```text
echo Form::email($name, $value = null, $attributes = array());
echo Form::file($name, $attributes = array());
```

## Checkboxes and Radio Buttons

### Generating A Checkbox Or Radio Input

```text
echo Form::checkbox('name', 'value');

echo Form::radio('name', 'value');
```

### Generating A Checkbox Or Radio Input That Is Checked

```text
echo Form::checkbox('name', 'value', true);

echo Form::radio('name', 'value', true);
```

## File Input

### Generating A File Input

```text
echo Form::file('image');
```

> **Note:** The form must have been opened with the `files` option set to `true`.

## Drop-Down Lists

### Generating A Drop-Down List

```text
echo Form::select('size', array('L' => 'Large', 'S' => 'Small'));
```

### Generating A Drop-Down List With Selected Default

```text
echo Form::select('size', array('L' => 'Large', 'S' => 'Small'), 'S');
```

### Generating A Grouped List

```text
echo Form::select('animal', array(
    'Cats' => array('leopard' => 'Leopard'),
    'Dogs' => array('spaniel' => 'Spaniel'),
));
```

### Generating A Drop-Down List With A Range

```text
echo Form::selectRange('number', 10, 20);
```

### Generating A List With Month Names

```text
echo Form::selectMonth('month');
```

## Buttons

### Generating A Submit Button

```text
echo Form::submit('Click Me!');
```

> **Note:** Need to create a button element? Try the _button_ method. It has the same signature as _submit_.

## Custom Macros

### Registering A Form Macro

It's easy to define your own custom Form class helpers called "macros". Here's how it works. First, simply register the macro with a given name and a Closure:

```text
Form::macro('myField', function()
{
    return '<input type="awesome">';
});
```

Now you can call your macro using its name:

### Calling A Custom Form Macro

```text
echo Form::myField();
```

## Generating URLs

For more information on generating URL's, check out the documentation on [helpers](https://github.com/bryantyan/laravel4.2docs/tree/f12ffb53f9f16c3968c58e9dd508247dc98deb70/docs/helpers/README.md#urls).

