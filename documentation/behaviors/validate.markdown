---
layout: documentation
title: Validate Behavior
---

# Validate Behavior #

The `validate` behavior provides validating capabilities to ActiveRecord objects.
Using this behavior, you can perform validation of an ActiveRecord and its related objects, checking if properties meet certain conditions.

This behavior is based on [Symfony2 Validator Component](http://symfony.com/doc/current/book/validation.html).
We recommend reading Symfony2 Validator Component documentation, in particular [Validator Constraints](http://symfony.com/doc/current/reference/constraints.html) chapter, before starting to use this behavior.

## Basic Usage ##

In the `schema.xml`, use the `<behavior>` tag to add the `validate` behavior and then add validation rules via the `<parameter>` tag to the table.

```xml
<table name="author" description="Author Table">
  <column name="id" required="true" primaryKey="true" autoIncrement="true" type="INTEGER" description="Author Id" />
  <column name="first_name" required="true" type="VARCHAR" size="128" description="First Name" />
  <column name="last_name" required="true" type="VARCHAR" size="128" description="Last Name" />
  <column name="email" type="VARCHAR" size="128" description="E-Mail Address" />

  <behavior name="validate">
    <parameter name="rule1" value="{column: first_name, validator: NotNull}" />
    <parameter name="rule2" value="{column: first_name, validator: Length, options: {max: 128}}" />
    <parameter name="rule3" value="{column: last_name, validator: NotNull}" />
    <parameter name="rule4" value="{column: last_name, validator: Length, options: {max: 128}}" />
    <parameter name="rule5" value="{column: email, validator: Email}" />
  </behavior>
</table>
```

Let's see the properties of `<parameter>` tag:

* The `name` of each parameter doesn't relate to a column, just make sure it is unique.
* The `value` of a parameter is an array in YAML format, in which we need to specify 3 values:
  * `column`: the column to validate
  * `validator`: the [Validator Constraint](http://symfony.com/doc/current/reference/constraints.html) you are using
  * `options`: (optional) an array of optional values to pass to the validator constraint class, according to its reference documentation



After rebuilding your model, the ActiveRecord object now exposes two additional public methods:
* `validate()`: this method performs validation on the ActiveRecord object itself and on all related objects. If the validation is successful it returns true, otherwise false.
* `getValidationFailures()`: this method returns a [ConstraintViolationList](http://api.symfony.com/2.0/Symfony/Component/Validator/ConstraintViolationList.html) object. If validate() is false, it returns a list of [ConstraintViolation](http://api.symfony.com/2.0/Symfony/Component/Validator/ConstraintViolation.html) objects, if validate() is true, it returns an empty `ConstraintViolationList` object.


Now you are ready to perform validations:

```php
<?php

$author = new Author();
$author->setLastName('Wilde');
$author->setFirstName('Oscar');
$author->setEmail('oscar.wilde@gmail.com');

if (!$author->validate()) {
    foreach ($author->getValidationFailures() as $failure) {
        echo "Property ".$failure->getPropertyPath().": ".$failure->getMessage()."\n";
    }
}
else {
   echo "Everything's all right!";
}

```



## Related objects validation ##


When using the ActiveRecord `validate()` method, we perform validation on the object itself and on all related objects. As an incredibly powerful function, we need to know what it does to avoid unpleasant surprises.


The `validate()` method follows these steps:

1.   search the 1-n related objects by foreign key
2.   if validate behavior is configured on it, call its `validate()` method
3.   performs validation on itself
4.   search the n-1 related objects
5.   if validate behavior is configured on them, call their `validate()` method



Let's see it in action, with an example.

Consider the following model:

```xml
<database name="bookstore">
    <table name="book">
        <column name="id" required="true" primaryKey="true" autoIncrement="true" type="INTEGER"/>
        <column name="title" type="VARCHAR" required="true" />
        <column name="isbn" type="VARCHAR" size="24" />
        <column name="price" required="false" type="FLOAT" />
        <column name="publisher_id" required="false" type="INTEGER" />
        <column name="author_id" required="false" type="INTEGER" />
        <foreign-key foreignTable="publisher" onDelete="setnull">
            <reference local="publisher_id" foreign="id" />
        </foreign-key>
        <foreign-key foreignTable="author" onDelete="setnull" onUpdate="cascade">
            <reference local="author_id" foreign="id" />
        </foreign-key>
        <behavior name="validate">
            <parameter name="rule1" value="{column: title, validator: NotNull}" />
        </behavior>
    </table>

    <table name="publisher">
        <column name="id" required="true" primaryKey="true" autoIncrement="true" type="INTEGER" />
        <column name="name" required="true" type="VARCHAR" size="128" />
        <column name="website" type="VARCHAR" />
        <behavior name="validate">
            <parameter name="rule1" value="{column: name, validator: NotNull}" />
            <parameter name="rule2" value="{column: website, validator: Url}" />
        </behavior>
    </table>

    <table name="author">
        <column name="id" required="true" primaryKey="true" autoIncrement="true" type="INTEGER" />
        <column name="first_name" required="true" type="VARCHAR" size="128" />
        <column name="last_name" required="true" type="VARCHAR" size="128" />
        <column name="email" type="VARCHAR" size="128" />
        <behavior name="validate">
            <parameter name="rule1" value="{column: first_name, validator: NotNull}" />
            <parameter name="rule2" value="{column: first_name, validator: Length, options: {max: 128}}" />
            <parameter name="rule3" value="{column: last_name, validator: NotNull}" />
            <parameter name="rule4" value="{column: last_name, validator: Length, options: {max: 128}}" />
            <parameter name="rule5" value="{column: email, validator: Email}" />
        </behavior>
    </table>

    <table name="reader">
        <column name="id" required="true" primaryKey="true" autoIncrement="true" type="INTEGER" />
        <column name="first_name" required="true" type="VARCHAR" size="128" />
        <column name="last_name" required="true" type="VARCHAR" size="128" />
        <column name="email" type="VARCHAR" size="128" />
        <behavior name="validate">
            <parameter name="rule1" value="{column: first_name, validator: NotNull}" />
            <parameter name="rule2" value="{column: first_name, validator: Length, options: {min: 4}}" />
            <parameter name="rule3" value="{column: last_name, validator: NotNull}" />
            <parameter name="rule4" value="{column: last_name, validator: Length, options: {max: 128}}" />
            <parameter name="rule5" value="{column: email, validator: Email}" />
        </behavior>
    </table>

    <table name="reader_book" isCrossRef="true">
         <column name="reader_id" type="INTEGER" primaryKey="true"/>
         <column name="book_id" type="INTEGER" primaryKey="true"/>
         <foreign-key foreignTable="reader">
              <reference local="reader_id" foreign="id"/>
         </foreign-key>
         <foreign-key foreignTable="book">
              <reference local="book_id" foreign="id"/>
         </foreign-key>
     </table>

</database>
```

And consider to perform a validation on a book object:

```php
<?php

$book = new Book();

// some operations by which we add to the book object some related objects:
// we add a publisher object, an author object and some reader objects

$book->validate();
```


The steps of validation are the following:

1.    search the author and publisher objects, related to our book
2.    author and publisher objects have the validate behavior tag in its schema definition, so `$author->validate()` and `$publisher->validate()` are called
3.    perform validation on the *book* object itself
4.    search all reader objects associated to this book object, using the `reader_book` table
5.    the reader_book table has *no* validate behavior tag so no other validation will be performed


In this case, no reader object will be validated because the cross reference table has no validate behavior, even if reader table has the validate behavior properly configured. No error message will be raised, because the behavior gives you the possibility to configure validations on a table but not on related ones. It's your choice.

Continuing with the previous example, you can also perform validations on reader objects, but we need to configure the behavior on the `reader_book` table as well:

```xml
<!-- previous schema -->

<table name="reader_book" isCrossRef="true">
         <column name="reader_id" type="INTEGER" primaryKey="true"/>
         <column name="book_id" type="INTEGER" primaryKey="true"/>
         <foreign-key foreignTable="reader">
              <reference local="reader_id" foreign="id"/>
         </foreign-key>
         <foreign-key foreignTable="book">
              <reference local="book_id" foreign="id"/>
         </foreign-key>
         <behavior name="validate">
            <parameter name="rule1" value="{column: reader_id, validator: NotNull}" />
            <parameter name="rule2" value="{column: book_id, validator: NotNull}" />
            <parameter name="rule3" value="{column: reader_id, validator: Type, options: {type: integer}}" />
            <parameter name="rule4" value="{column: book_id, validator: Type, options: {type: integer}}" />
        </behavior>
     </table>
```

And now the validation process will be the following:

1.    search the author and publisher objects
2.    author and publisher objects have the validate behavior tag in its schema definition, so `$author->validate()` and `$publisher->validate()` are called
3.    perform validation on $book itself
4.    search all readers associated to this book object, by using the `reader_book` table
5.    the `reader_book` table now has the behavior, so `$reader_book->validate()` is called
6.    inside `$reader_book->validate()` all related reader objects will be searched and validated

>**Tip**If you configure the behavior on all related objects, every object will *ALWAYS* be validated, no matter if you call the `validate()` method of one or the other.



## Parameter tag: name ##

Inside the `<parameter>` tag, you define the `name` property.
This property can be a value of your choice, but this name should be *unique*. If we define more rules with the same name, only the last one will be considered.

In the following example, only the third and the fourth rules will be considered: the first two rules are overwritten by the third one.

```xml
<!-- your schema -->

   <column name="reader_id" type="INTEGER" primaryKey="true"/>
   <column name="book_id" type="INTEGER" primaryKey="true"/>
   <behavior name="validate">
       <parameter name="rule1" value="{column: reader_id, validator: NotNull}" />
       <parameter name="rule1" value="{column: book_id, validator: NotNull}" />
       <parameter name="rule1" value="{column: reader_id, validator: Type, options: {type: integer}}" />
       <parameter name="rule2" value="{column: book_id, validator: Type, options: {type: integer}}" />
    </behavior>

<!-- end of your schema -->
```


## Parameter tag: value ##

As we mentioned earlier, the `value` property contains a string, representing an array in YAML format. We've chosen this format because, in the YAML array definition, there is no special xml character, so we don't need to escape anything and there's no need to change standard Propel xsd and xsl files.
The `options` key, inside the value array, is an array too and it can contain other arrays (i.e. see [Choice constraint](http://symfony.com/doc/current/reference/constraints/Choice.html), in which the `choices` option is also an array) and with YAML there's no problem.

There is only one case we suggest to be careful.
Any respectable validation library (also the Symfony Validator Component) allows validations against regular expressions, by using the constraint [Regex](http://symfony.com/doc/current/reference/constraints/Regex.html).
As you can see in the Regex constraint documentation, the `options` parameter contains a `pattern` key, defining the pattern for validation.

Usually however, a regular expression pattern contains a lot of special and escape characters so, in YAML definition, we need to include the pattern string in a pair of double-quotes (").

In the following example, we add a constraint to validate ISBN. It's very complicated to check if an ISBN is valid, but an initial check could be to disallow every character that's not a digit or minus, using the pattern  `/[^\d-]+/`:

```xml
<!-- ATTENTION PLEASE: THIS EXAMPLE DOES NOT WORK -->

<!-- your schema -->
  <behavior name="validate">
      .......
      <parameter name="rule1" value="{column: isbn, validator: Regex, options: {pattern: "/[^\d-]+/", match: false, message: Please enter a valid ISBN}}" />
  </behavior>

<!-- end of your schema -->
```

Remember that inside an xml string the double-quote characters should be escaped, so replace them with `&quot;`:


```xml
<!-- THIS EXAMPLE WORKS FINE -->

<!-- your schema -->
  <behavior name="validate">
      .......
      <parameter name="rule1" value="{column: isbn, validator: Regex, options: {pattern: &quot;/[^\d-]+/&quot;, match: false, message: Please enter a valid ISBN }}" />
  </behavior>

<!-- end of your schema -->
```


## Automatic validation ##

You can automatically validate an ActiveRecord before saving it into your database, thanks to the `preSave()` hook (see [behaviors documentation](/documentation/06-behaviors.html)).
For example, let's suppose we wish to add automatic validation capability to our `Book` class. Open `Book.php`, in your model path, and add the following code:

```php
<?php
// Code of your Book class.
// Remember use statement to set properly ConnectionInterface namespace

public function preSave(ConnectionInterface $con = null)
{
    return $this->validate();
}
```

If validation failed, `preSave()` returns false and the saving operation stops. No error is raised but the `save()` method of your ActiveRecord returns the integer `0`, because no object was affected. So, we can check the returned value of the `save()` method to see what has happened and to get any error messages:

```php
<?php
// your app code

$author = AuthorQuery::create()->findPk(1);
$publisher = PublisherQuery::create()->findPk(1);

$book = new Book();
$book->setAuthor($author);
$book->setPublisher($publisher);
$book->setTitle('The country house');
$book->setPrice(10,00);

$ret = $book->save();

// if $ret <= 0 means no affected rows, that is validation failed or no object to persist
if ($ret <= 0) {
    $failures = $book->getValidationFailures();

    // count($failures) > 0 means that we have ConstraintViolation objects and validation failed
    if (count($failures) > 0) {
        foreach ($failures as $failure) {
            echo $failure->getPropertyPath()." validation failed: ".$failure->getMessage();
        }
    }
}
```

## Supported constraints ##

The behavior supports all Symfony Validator Constraints (see [Symfony documentation] (http://symfony.com/doc/current/reference/constraints.html) for details), except `UniqueEntity` which is not compatible with Propel.
Propel has its own unique validator: the `Unique` constraint.
This constraint checks if a certain value is already stored in the database.

You can use it in the same way:

```xml
<!-- your schema -->
  <behavior name="validate">
      <parameter name="rule1" value="{column: column_name, validator: Unique}" />
  </behavior>
```

And if you want to specify an error message:

```xml
<!-- your schema -->
  <behavior name="validate">
      <parameter name="rule1" value="{column: column_name, validator: Unique, options: {message: Your message here}}" />
  </behavior>
```

>**Tip** Do you store date-times as strings? Use the `Date`, `Time` and `DateTime` constraints to prevent invalid PHP date-times from raising exceptions before any validations are performed.


## Custom validation constraints ##

Together, Propel and the Symfony 2 Validator component come with many bundled constraints which should cover most validation needs.

For cases that the inbuilt constraints do not cover, you will need to create your own. Fortunately, this is very easy and can be considered a two-step process:

1.    Write your custom constraint: please refer to [this document](http://symfony.com/doc/current/cookbook/validation/custom_constraint.html) and see the example below.
2.    Set up Propel to work with it: simply adjust your autoload class or function, to correctly map `Propel\Runtime\Validator\Constraints` namespace to the directory in which your constraint scripts reside.

>**Tip** Propel expects to find custom constraints under the `Propel\Runtime\Validator\Constraints` namespace.


For example, let's suppose we want to write a custom constraint, called *PropelDomain*, that checks if an url belongs to *propelorm.org* domain.
Let's put our files in a subdir of our project root, called `/myConstraints`, and we'll manage the dependencies of our project with [Composer](http://getcomposer.org).

Under the `/myConstraints` dir, let's create a subdir `Propel/Runtime/Validator/Constraints`, in which we'll put the two following scripts:

`PropelDomain.php`

```php
<?php
// /myConstraints/Propel/Runtime/Validator/Constraints/PropelDomain.php

namespace Propel\Runtime\Validator\Constraints;

use Symfony\Component\Validator\Constraint;


class PropelDomain extends Constraint
{
    public $message = 'This url does not belong to propelorm.org domain';
    public $column = '';
}
```

`PropelDomainValidator.php`

```php
<?php
 // /myConstraints/Propel/Runtime/Validator/Constraints/PropelDomainValidator.php

 namespace Propel\Runtime\Validator\Constraints;

 use Symfony\Component\Validator\Constraint;
 use Symfony\Component\Validator\ConstraintValidator;

 class PropelDomainValidator extends ConstraintValidator
 {
     public function isValid($value, Constraint $constraint)
     {
         if ('propelorm.org' === strstr($value, 'propelorm.org')) {
             return false;
         } else {
             $this->setMessage($constraint->message);

             return true;
         }
     }
 }
```

Now, open the `composer.json` file in your project root and add the namespace `Propel\Runtime\Validator\Constraints` to the autoload directive:

```json
"autoload": {
        "psr-0": {
            "Propel\\Runtime\\Validator\\Constraints": "myConstraints/"
        }
    }
```

Done! Now you can use your custom validator constraint in your `schema.xml` file, as usual:

```xml

<!-- your schema -->
  <behavior name="validate">
      <parameter name="rule1" value="{column: website, validator: PropelDomain, options: {message: Your custom message}}" />
  </behavior>

<!-- end of your schema -->
```

**Note**: if you think your custom constraint is generic enough for community use, please submit it to the Propel team
to include it in our bundled constraints (see <http://www.redotheweb.com/2011/11/13/open-source-is-a-gift.html>).

## Inside Symfony2 ##

This behavior adds to ActiveRecord objects the static `loadValidatorMetadata()` method, which contains all validation rules. So, inside your Symfony projects, you can perform the "usual" Symfony validations:

```php
<?php

// Symfony 2

use Symfony\Component\HttpFoundation\Response;
use YourVendor\YourBundle\Model\Author;
// ...

public function indexAction()
{
    $author = new Author();
    // ... do something to the $author object

    $validator = $this->get('validator');
    $errors = $validator->validate($author);

    if (count($errors) > 0) {
        return new Response(print_r($errors, true));
    } else {
        return new Response('The author is valid! Yes!');
    }
}
```

But if you also want automatic validation of related objects, you can use the ActiveRecord `validate()` method, passing to it an instance of the registered validator object:

```php
<?php

// Symfony 2

use Symfony\Component\HttpFoundation\Response;
use YouVendor\YourBundle\Model\Author;
// ...

public function indexAction()
{
    $author = new Author();
    // ... do something to the $author object

    $validator = $this->get('validator');
    if (!$author->validate($validator)) {
        $errors = $author->getValidationFailures();

        return new Response(print_r($errors, true));

    }
    else {
        return new Response('The author is valid! Yes!');
    }
}
```


## Inside Silex ##

Using the behavior inside a Silex project is very similar to Symfony:

```php
<?php

// Silex

// ...

$app->post('/authors/new', function () use ($app) {
    $post = new Author();
    $author->setLastName($app['request']->get('lastname'));
    $author->setFirstName($app['request']->get('firstname'));
    $author->setEmail($app['request']->get('email'));

    $violations = $app['validator']->validate($author);

    return $violations;

}
```

and if you also want automatic validation of related objects:

```php
<?php

// Silex

// ...

$app->post('/authors/new', function () use ($app) {
    $post = new Author();
    $author->setLastName($app['request']->get('lastname'));
    $author->setFirstName($app['request']->get('firstname'));
    $author->setEmail($app['request']->get('email'));

    $author->validate($app['validator']))
    $violations = $author->getValidationFailures();

    return $violations;

}
```

## Properties and methods added to ActiveRecord ##

The behavior adds the following properties to your ActiveRecord:

*   `alreadyInValidation`:  this *protected* property is a flag to prevent an endless validation loop in case this object is referenced by another we're already performing a validation on.
*   `validationFailures`:   this *protected* property contains the ConstraintViolationList object.


The behavior adds the following methods to your ActiveRecord:

*   `validate`:  this *public* method validates the object and all objects related to it.
*   `getValidationFailures`:  this *public* method gets the ConstraintViolationList object, which contains all ConstraintViolation objects resulted from the last call to `validate()` method.
*   `loadValidatorMetadata`:  this *public static* method contains all the Constraint objects.
