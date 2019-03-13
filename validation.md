# Validation

* [Basic Usage](validation.md#basic-usage)
* [Working With Error Messages](validation.md#working-with-error-messages)
* [Error Messages & Views](validation.md#error-messages-and-views)
* [Available Validation Rules](validation.md#available-validation-rules)
* [Conditionally Adding Rules](validation.md#conditionally-adding-rules)
* [Custom Error Messages](validation.md#custom-error-messages)
* [Custom Validation Rules](validation.md#custom-validation-rules)

## Basic Usage

Laravel ships with a simple, convenient facility for validating data and retrieving validation error messages via the `Validation` class.

#### Basic Validation Example

```text
$validator = Validator::make(
    array('name' => 'Dayle'),
    array('name' => 'required|min:5')
);
```

The first argument passed to the `make` method is the data under validation. The second argument is the validation rules that should be applied to the data.

#### Using Arrays To Specify Rules

Multiple rules may be delimited using either a "pipe" character, or as separate elements of an array.

```text
$validator = Validator::make(
    array('name' => 'Dayle'),
    array('name' => array('required', 'min:5'))
);
```

#### Validating Multiple Fields

```text
$validator = Validator::make(
    array(
        'name' => 'Dayle',
        'password' => 'lamepassword',
        'email' => 'email@example.com'
    ),
    array(
        'name' => 'required',
        'password' => 'required|min:8',
        'email' => 'required|email|unique:users'
    )
);
```

Once a `Validator` instance has been created, the `fails` \(or `passes`\) method may be used to perform the validation.

```text
if ($validator->fails())
{
    // The given data did not pass validation
}
```

If validation has failed, you may retrieve the error messages from the validator.

```text
$messages = $validator->messages();
```

You may also access an array of the failed validation rules, without messages. To do so, use the `failed` method:

```text
$failed = $validator->failed();
```

#### Validating Files

The `Validator` class provides several rules for validating files, such as `size`, `mimes`, and others. When validating files, you may simply pass them into the validator with your other data.

## Working With Error Messages

After calling the `messages` method on a `Validator` instance, you will receive a `MessageBag` instance, which has a variety of convenient methods for working with error messages.

#### Retrieving The First Error Message For A Field

```text
echo $messages->first('email');
```

#### Retrieving All Error Messages For A Field

```text
foreach ($messages->get('email') as $message)
{
    //
}
```

#### Retrieving All Error Messages For All Fields

```text
foreach ($messages->all() as $message)
{
    //
}
```

#### Determining If Messages Exist For A Field

```text
if ($messages->has('email'))
{
    //
}
```

#### Retrieving An Error Message With A Format

```text
echo $messages->first('email', '<p>:message</p>');
```

> **Note:** By default, messages are formatted using Bootstrap compatible syntax.

#### Retrieving All Error Messages With A Format

```text
foreach ($messages->all('<li>:message</li>') as $message)
{
    //
}
```

## Error Messages & Views

Once you have performed validation, you will need an easy way to get the error messages back to your views. This is conveniently handled by Laravel. Consider the following routes as an example:

```text
Route::get('register', function()
{
    return View::make('user.register');
});

Route::post('register', function()
{
    $rules = array(...);

    $validator = Validator::make(Input::all(), $rules);

    if ($validator->fails())
    {
        return Redirect::to('register')->withErrors($validator);
    }
});
```

Note that when validation fails, we pass the `Validator` instance to the Redirect using the `withErrors` method. This method will flash the error messages to the session so that they are available on the next request.

However, notice that we do not have to explicitly bind the error messages to the view in our GET route. This is because Laravel will always check for errors in the session data, and automatically bind them to the view if they are available. **So, it is important to note that an** `$errors` **variable will always be available in all of your views, on every request**, allowing you to conveniently assume the `$errors` variable is always defined and can be safely used. The `$errors` variable will be an instance of `MessageBag`.

So, after redirection, you may utilize the automatically bound `$errors` variable in your view:

```text
<?php echo $errors->first('email'); ?>
```

### Named Error Bags

If you have multiple forms on a single page, you may wish to name the `MessageBag` of errors. This will allow you to retrieve the error messages for a specific form. Simply pass a name as the second argument to `withErrors`:

```text
return Redirect::to('register')->withErrors($validator, 'login');
```

You may then access the named `MessageBag` instance from the `$errors` variable:

```text
<?php echo $errors->login->first('email'); ?>
```

## Available Validation Rules

Below is a list of all available validation rules and their function:

* [Accepted](validation.md#rule-accepted)
* [Active URL](validation.md#rule-active-url)
* [After \(Date\)](validation.md#rule-after)
* [Alpha](validation.md#rule-alpha)
* [Alpha Dash](validation.md#rule-alpha-dash)
* [Alpha Numeric](validation.md#rule-alpha-num)
* [Array](validation.md#rule-array)
* [Before \(Date\)](validation.md#rule-before)
* [Between](validation.md#rule-between)
* [Confirmed](validation.md#rule-confirmed)
* [Date](validation.md#rule-date)
* [Date Format](validation.md#rule-date-format)
* [Different](validation.md#rule-different)
* [Digits](validation.md#rule-digits)
* [Digits Between](validation.md#rule-digits-between)
* [E-Mail](validation.md#rule-email)
* [Exists \(Database\)](validation.md#rule-exists)
* [Image \(File\)](validation.md#rule-image)
* [In](validation.md#rule-in)
* [Integer](validation.md#rule-integer)
* [IP Address](validation.md#rule-ip)
* [Max](validation.md#rule-max)
* [MIME Types](validation.md#rule-mimes)
* [Min](validation.md#rule-min)
* [Not In](validation.md#rule-not-in)
* [Numeric](validation.md#rule-numeric)
* [Regular Expression](validation.md#rule-regex)
* [Required](validation.md#rule-required)
* [Required If](validation.md#rule-required-if)
* [Required With](validation.md#rule-required-with)
* [Required With All](validation.md#rule-required-with-all)
* [Required Without](validation.md#rule-required-without)
* [Required Without All](validation.md#rule-required-without-all)
* [Same](validation.md#rule-same)
* [Size](validation.md#rule-size)
* [Unique \(Database\)](validation.md#rule-unique)
* [URL](validation.md#rule-url)

#### accepted

The field under validation must be _yes_, _on_, or _1_. This is useful for validating "Terms of Service" acceptance.

#### active\_url

The field under validation must be a valid URL according to the `checkdnsrr` PHP function.

#### after:_date_

The field under validation must be a value after a given date. The dates will be passed into the PHP `strtotime` function.

#### alpha

The field under validation must be entirely alphabetic characters.

#### alpha\_dash

The field under validation may have alpha-numeric characters, as well as dashes and underscores.

#### alpha\_num

The field under validation must be entirely alpha-numeric characters.

#### array

The field under validation must be of type array.

#### before:_date_

The field under validation must be a value preceding the given date. The dates will be passed into the PHP `strtotime` function.

#### between:_min_,_max_

The field under validation must have a size between the given _min_ and _max_. Strings, numerics, and files are evaluated in the same fashion as the `size` rule.

#### confirmed

The field under validation must have a matching field of `foo_confirmation`. For example, if the field under validation is `password`, a matching `password_confirmation` field must be present in the input.

#### date

The field under validation must be a valid date according to the `strtotime` PHP function.

#### date_format:\_format_

The field under validation must match the _format_ defined according to the `date_parse_from_format` PHP function.

#### different:_field_

The given _field_ must be different than the field under validation.

#### digits:_value_

The field under validation must be _numeric_ and must have an exact length of _value_.

#### digits_between:\_min_,_max_

The field under validation must have a length between the given _min_ and _max_.

#### email

The field under validation must be formatted as an e-mail address.

#### exists:_table_,_column_

The field under validation must exist on a given database table.

#### Basic Usage Of Exists Rule

```text
'state' => 'exists:states'
```

#### Specifying A Custom Column Name

```text
'state' => 'exists:states,abbreviation'
```

You may also specify more conditions that will be added as "where" clauses to the query:

```text
'email' => 'exists:staff,email,account_id,1'
```

Passing `NULL` as a "where" clause value will add a check for a `NULL` database value:

```text
'email' => 'exists:staff,email,deleted_at,NULL'
```

#### image

The file under validation must be an image \(jpeg, png, bmp, or gif\)

#### in:_foo_,_bar_,...

The field under validation must be included in the given list of values.

#### integer

The field under validation must have an integer value.

#### ip

The field under validation must be formatted as an IP address.

#### max:_value_

The field under validation must be less than or equal to a maximum _value_. Strings, numerics, and files are evaluated in the same fashion as the `size` rule.

#### mimes:_foo_,_bar_,...

The file under validation must have a MIME type corresponding to one of the listed extensions.

#### Basic Usage Of MIME Rule

```text
'photo' => 'mimes:jpeg,bmp,png'
```

#### min:_value_

The field under validation must have a minimum _value_. Strings, numerics, and files are evaluated in the same fashion as the `size` rule.

#### not_in:\_foo_,_bar_,...

The field under validation must not be included in the given list of values.

#### numeric

The field under validation must have a numeric value.

#### regex:_pattern_

The field under validation must match the given regular expression.

**Note:** When using the `regex` pattern, it may be necessary to specify rules in an array instead of using pipe delimiters, especially if the regular expression contains a pipe character.

#### required

The field under validation must be present in the input data.

#### required\_if:_field_,_value_

The field under validation must be present if the _field_ field is equal to _value_.

#### required_with:\_foo_,_bar_,...

The field under validation must be present _only if_ any of the other specified fields are present.

#### required_with\_all:\_foo_,_bar_,...

The field under validation must be present _only if_ all of the other specified fields are present.

#### required_without:\_foo_,_bar_,...

The field under validation must be present _only when_ any of the other specified fields are not present.

#### required_without\_all:\_foo_,_bar_,...

The field under validation must be present _only when_ the all of the other specified fields are not present.

#### same:_field_

The given _field_ must match the field under validation.

#### size:_value_

The field under validation must have a size matching the given _value_. For string data, _value_ corresponds to the number of characters. For numeric data, _value_ corresponds to a given integer value. For files, _size_ corresponds to the file size in kilobytes.

#### unique:_table_,_column_,_except_,_idColumn_

The field under validation must be unique on a given database table. If the `column` option is not specified, the field name will be used.

#### Basic Usage Of Unique Rule

```text
'email' => 'unique:users'
```

#### Specifying A Custom Column Name

```text
'email' => 'unique:users,email_address'
```

#### Forcing A Unique Rule To Ignore A Given ID

```text
'email' => 'unique:users,email_address,10'
```

#### Adding Additional Where Clauses

You may also specify more conditions that will be added as "where" clauses to the query:

```text
'email' => 'unique:users,email_address,NULL,id,account_id,1'
```

In the rule above, only rows with an `account_id` of `1` would be included in the unique check.

#### url

The field under validation must be formatted as an URL.

> **Note:** This function uses PHP's `filter_var` method.

## Conditionally Adding Rules

In some situations, you may wish to run validation checks against a field **only** if that field is present in the input array. To quickly accomplish this, add the `sometimes` rule to your rule list:

```text
$v = Validator::make($data, array(
    'email' => 'sometimes|required|email',
));
```

In the example above, the `email` field will only be validated if it is present in the `$data` array.

#### Complex Conditional Validation

Sometimes you may wish to require a given field only if another field has a greater value than 100. Or you may need two fields to have a given value only when another field is present. Adding these validation rules doesn't have to be a pain. First, create a `Validator` instance with your _static rules_ that never change:

```text
$v = Validator::make($data, array(
    'email' => 'required|email',
    'games' => 'required|numeric',
));
```

Let's assume our web application is for game collectors. If a game collector registers with our application and they own more than 100 games, we want them to explain why they own so many games. For example, perhaps they run a game re-sell shop, or maybe they just enjoy collecting. To conditionally add this requirement, we can use the `sometimes` method on the `Validator` instance.

```text
$v->sometimes('reason', 'required|max:500', function($input)
{
    return $input->games >= 100;
});
```

The first argument passed to the `sometimes` method is the name of the field we are conditionally validating. The second argument is the rules we want to add. If the `Closure` passed as the third argument returns `true`, the rules will be added. This method makes it a breeze to build complex conditional validations. You may even add conditional validations for several fields at once:

```text
$v->sometimes(array('reason', 'cost'), 'required', function($input)
{
    return $input->games >= 100;
});
```

> **Note:** The `$input` parameter passed to your `Closure` will be an instance of `Illuminate\Support\Fluent` and may be used as an object to access your input and files.

## Custom Error Messages

If needed, you may use custom error messages for validation instead of the defaults. There are several ways to specify custom messages.

#### Passing Custom Messages Into Validator

```text
$messages = array(
    'required' => 'The :attribute field is required.',
);

$validator = Validator::make($input, $rules, $messages);
```

> _Note:_ The `:attribute` place-holder will be replaced by the actual name of the field under validation. You may also utilize other place-holders in validation messages.

#### Other Validation Place-Holders

```text
$messages = array(
    'same'    => 'The :attribute and :other must match.',
    'size'    => 'The :attribute must be exactly :size.',
    'between' => 'The :attribute must be between :min - :max.',
    'in'      => 'The :attribute must be one of the following types: :values',
);
```

#### Specifying A Custom Message For A Given Attribute

Sometimes you may wish to specify a custom error messages only for a specific field:

```text
$messages = array(
    'email.required' => 'We need to know your e-mail address!',
);
```

#### Specifying Custom Messages In Language Files

In some cases, you may wish to specify your custom messages in a language file instead of passing them directly to the `Validator`. To do so, add your messages to `custom` array in the `app/lang/xx/validation.php` language file.

```text
'custom' => array(
    'email' => array(
        'required' => 'We need to know your e-mail address!',
    ),
),
```

## Custom Validation Rules

#### Registering A Custom Validation Rule

Laravel provides a variety of helpful validation rules; however, you may wish to specify some of your own. One method of registering custom validation rules is using the `Validator::extend` method:

```text
Validator::extend('foo', function($attribute, $value, $parameters)
{
    return $value == 'foo';
});
```

The custom validator Closure receives three arguments: the name of the `$attribute` being validated, the `$value` of the attribute, and an array of `$parameters` passed to the rule.

You may also pass a class and method to the `extend` method instead of a Closure:

```text
Validator::extend('foo', 'FooValidator@validate');
```

Note that you will also need to define an error message for your custom rules. You can do so either using an inline custom message array or by adding an entry in the validation language file.

#### Extending The Validator Class

Instead of using Closure callbacks to extend the Validator, you may also extend the Validator class itself. To do so, write a Validator class that extends `Illuminate\Validation\Validator`. You may add validation methods to the class by prefixing them with `validate`:

```text
<?php

class CustomValidator extends Illuminate\Validation\Validator {

    public function validateFoo($attribute, $value, $parameters)
    {
        return $value == 'foo';
    }

}
```

#### Registering A Custom Validator Resolver

Next, you need to register your custom Validator extension:

```text
Validator::resolver(function($translator, $data, $rules, $messages)
{
    return new CustomValidator($translator, $data, $rules, $messages);
});
```

When creating a custom validation rule, you may sometimes need to define custom place-holder replacements for error messages. You may do so by creating a custom Validator as described above, and adding a `replaceXXX` function to the validator.

```text
protected function replaceFoo($message, $attribute, $rule, $parameters)
{
    return str_replace(':foo', $parameters[0], $message);
}
```

If you would like to add a custom message "replacer" without extending the `Validator` class, you may use the `Validator::replacer` method:

```text
Validator::replacer('rule', function($message, $attribute, $rule, $parameters)
{
    //
});
```

