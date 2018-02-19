---
layout: documentation
title: Working With Symfony2 - Introduction
---

# Working With Symfony2 - Introduction #

If you are interested to work with Propel2 with Symfony2, you should consider
using the [PropelBundle](https://github.com/propelorm/PropelBundle).

## Installation ##

The recommended way to install this bundle is to rely on
[Composer](http://getcomposer.org/):

``` json
"require": {
    "propel/propel": "2.0.*@dev",
    "propel/propel-bundle": "~2.0@dev"
}
```

The second step is to register this bundle in the AppKernel class:

``` php
<?php

public function registerBundles()
{
    $bundles = array(
        // ...
        new Propel\Bundle\PropelBundle\PropelBundle(),
    );

    // ...
}
```

You are almost ready, the next steps are:

* to [configure propel](#configuration) ;
* to [write an XML schema](#xml-schema).

Now, you can build your model classes, and SQL by running the following command:

```bash
$ php app/console propel:build [--classes] [--sql] [--insert-sql]
```

To insert SQL statements, use the propel:sql:insert command:

```bash
$ php app/console propel:sql:insert
```

Congratulations! You're done; just use the Model classes as any other class in
Symfony2:

``` php
<?php

class HelloController extends Controller
{
    public function indexAction($name)
    {
        $author = new \Acme\DemoBundle\Model\Author();
        $author->setFirstName($name);
        $author->save();

        return $this->render('AcmeDemoBundle:Hello:index.html.twig', array(
            'name' => $name, 'author' => $author)
        );
    }
}
```

## Bundle Inheritance ##

The *PropelBundle* makes use of the bundle inheritance. Currently only schema
inheritance is provided.

### Schema Inheritance ###

You can override the defined schema of a bundle from within its child bundle.
To make use of the inheritance you only need to drop a schema file in the
`Resources/config` folder of the child bundle.

Each file can be overridden without interfering with other schema files. If
you want to remove parts of a schema, you only need to add an empty schema
file.

## Configuration ##

### Basic configuration ###

If you have just one database connection, your configuration will look like as
following:

``` yaml
# app/config/config*.yml
propel:
    database:
        connections:
            default:
                adapter:    mysql
                user:       sandbox
                password:   null
                dsn:        mysql:host=localhost;dbname=sandbox;charset=UTF8
```

The recommended way to fill in these information is to use parameters:

``` yaml
# app/config/config*.yml
# define the parameters in app/config/parameters.yml
propel:
    database:
        connections:
            default:
                adapter:    %database_driver%
                user:       %database_user%
                password:   %database_password%
                dsn:        %database_driver%:host=%database_host%;dbname=%database_name%;charset=UTF8
```

### Configure Multiple Connections ###

If you have more than one connection the configuration will look like:

``` yaml
# app/config/config*.yml
propel:
    database:
        connections:
            default:
                adapter:    mysql
                user:       sandbox
                password:   null
                dsn:        mysql:host=localhost;dbname=sandbox;charset=UTF8
            other:
                adapter:    mysql
                user:       other
                password:   null
                dsn:        mysql:host=localhost;dbname=other;charset=UTF8
    runtime:
        defaultConnection: default
        connections:       [ default, other ]
    generator:
        defaultConnection: default
        connections:       [ default, other ]
```

### Attributes, Options, Settings ###

``` yaml
# app/config/config*.yml
# define the parameters in app/config/parameters.yml
propel:
    database:
        connections:
            default:
                # ...
                options:
                    ATTR_PERSISTENT: false
                attributes:
                    ATTR_EMULATE_PREPARES: true
                settings:
                    charset:        { value: UTF8 }
                    queries:        { query: 'INSERT INTO BAR ('hey', 'there')' }
```

See the [configuration reference](/documentation/reference/configuration-file.html)
for further details.

## XML Schema ##

Place the following schema in `src/Acme/DemoBundle/Resources/config/schema.xml`:

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<database name="default" namespace="Acme\DemoBundle\Model" defaultIdMethod="native">

    <table name="book">
        <column name="id" type="integer" required="true" primaryKey="true" autoIncrement="true" />
        <column name="title" type="varchar" primaryString="1" size="100" />
        <column name="ISBN" type="varchar" size="20" />
        <column name="author_id" type="integer" />
        <foreign-key foreignTable="author">
            <reference local="author_id" foreign="id" />
        </foreign-key>
    </table>

    <table name="author">
        <column name="id" type="integer" required="true" primaryKey="true" autoIncrement="true" />
        <column name="first_name" type="varchar" size="100" />
        <column name="last_name" type="varchar" size="100" />
    </table>

</database>
```

If you are working with an existing database, please check the [related
section](#working-with-existing-databases).

## The Commands ##

The *PropelBundle* provides a lot of commands to manage migrations, database/table
manipulations, and so on.

### Database Manipulations ###

You can **create a database**:

```bash
$ php app/console propel:database:create [--connection[=""]]
```

As usual, `--connection` allows you to specify a connection.

You can **drop a database**:

```bash
$ php app/console propel:database:drop [--connection[=""]] [--force]
```

Note that the `--force` option is needed to actually execute the SQL statements.

### Form Types ###

You can generate stub classes based on your `schema.xml` in a given bundle:

```bash
$ php app/console propel:form:generate [-f|--force] bundle [models1] ... [modelsN]
```

It will write Form Type classes in `src/YourVendor/YourBundle/Form/Type`.

You can choose which Form Type to build by specifying Model names:

```bash
$ php app/console propel:form:generate @AcmeDemoBundle Book Author
```

### Graphviz ###

You can generate **Graphviz** file for your project by using the following command
line:

```bash
$ php app/console propel:graphviz:generate
```

It will write files in `./generated-graphviz`.

### Migrations ###

Generates SQL diff between the XML schemas and the current database structure:

```bash
$ php app/console propel:migration:generate-diff
```

Executes the migrations:

```bash
$ php app/console propel:migration:migrate
```

Executes the next migration up:

```bash
$ php app/console propel:migration:migrate --up
```

Executes the previous migration down:

```bash
$ php app/console propel:migration:migrate --down
```

Lists the pending migrations:

```bash
$ php app/console propel:migration:status
```

### Table Manipulations ###

You can drop one or several tables:

```bash
$ php app/console propel:table:drop [--force] [--connection[="..."]] [table1] ... [tableN]
```

The table arguments define which tables will be deleted, by default all tables
are deleted.

Note that the `--force` option is needed to actually execute the deletion.

### Working with existing databases ###

Run the following command to generate an XML schema from your default database:

```bash
$ php app/console propel:reverse
```

You can define which connection to use:

```bash
$ php app/console propel:reverse --connection=default
```

This will create your schema file under `./generated-schemas`. You need to
move/copy it to the corresponding bundle config directory. For example: `src/Acme/DemoBundle/Resources/config/`.

## The Fixtures ##

Fixtures are data you usually write to populate your database during the
development, or static content like menus, labels, etc. you need by default in
your production database.

### Loading Fixtures ###

The following command is designed to load fixtures:

```bash
$ php app/console propel:fixtures:load [-d|--dir[="..."]] [--xml] [--sql] [--yml] [--connection[="..."]] [bundle]
```

As you can see, there are many options to allow you to easily load fixtures.

As usual, `--connection` allows to specify a connection. The `--dir` option allows
to specify a directory containing the fixtures (default is: `app/propel/fixtures/`).
Note that the `--dir` expects a relative path from the root dir (which is `app/`).

The `--xml` parameter allows you to load only XML fixtures. The `--sql` parameter
allows you to load only SQL fixtures. The `--yml` parameter allows you to load
only YAML fixtures.

You can mix `--xml`, `--yml` and `--sql` parameters to load XML, YAML and SQL
fixtures at the same time. If none of these parameters are set, all YAML, XML
and SQL files in the directory will be loaded.

You can pass a bundle name to load fixtures from it. A bundle's name starts
with *@* like *@AcmeDemoBundle*.

```bash
$ php app/console propel:fixtures:load @AcmeDemoBundle
```

### XML Fixtures ###

A valid XML fixtures file is:

``` xml
<Fixtures>
    <Object Namespace="Awesome">
        <o1 Title="My title" MyFoo="bar" />
    </Object>
    <Related Namespace="Awesome">
        <r1 ObjectId="o1" Description="Hello world !" />
    </Related>
</Fixtures>
```

### YAML Fixtures ###

A valid YAML fixtures file is:

```
Awesome\Object:
     o1:
         Title: My title
         MyFoo: bar

Awesome\Related:
     r1:
         ObjectId: o1
         Description: Hello world !

Awesome\Tag:
    t1:
        name: Foo
    t2:
        name: Baz

Awesome\Post:
    p1:
        title: A Post with tags (N-N relation)
        tags: [ t1, t2 ]
```

#### Using Faker in YAML Fixtures ####

If you use [Faker](https://github.com/fzaninotto/Faker) with its [Symfony2](https://github.com/willdurand/BazingaFakerBundle)
integration, then the PropelBundle offers a facility to use the Faker
generator in your YAML files:

``` yaml
Acme\DemoBundle\Model\Book:
    Book1:
        name:        "Awesome Feature"
        description: <?php $faker('text', 500); ?>
```

The aim of this feature is to be able to mix both real and fake data in the same
file. Fake data is interesting to quickly add data to your application, but most
of the time you need to rely on real data. Integrating Faker within your YAML
files allows you to write strong fixtures efficiently.

## Dumping data ##

You can dump data from your database into YAML fixtures file by using this
command:

```bash
$ php app/console propel:fixtures:dump [--connection[="..."]]
```

Dumped files will be written in the fixtures directory: `app/propel/fixtures/`
with the following name: `fixtures_99999.yml` where 99999 is a timestamp.

Once done, you will be able to load these files by using the
`propel:fixtures:load` command.

## The PropelParamConverter ##

You can use the PropelParamConverter with the [SensioFrameworkExtraBundle](http://github.com/sensio/SensioFrameworkExtraBundle).
You just need to put the right Annotation on top of your controller:

``` php
<?php

use Sensio\Bundle\FrameworkExtraBundle\Configuration\ParamConverter;
use BlogBundle\Model\Post;

/**
 * @ParamConverter("post", class="BlogBundle\Model\Post")
 */
public function myAction(Post $post)
{
}
```

Your request needs to have an id parameter or any field as parameter (e.g. slug,
title).

The *Annotation* is optional if your parameter is typed you could only have this:

``` php
<?php

use BlogBundle\Model\Post;

public function myAction(Post $post)
{
}
```

**New** with last version of *SensioFrameworkExtraBundle*, you can omit the class
parameter if your controller parameter is typed, this is useful when you need to
set extra options.

``` php
<?php

use BlogBundle\Model\Post;

/**
 * @ParamConverter("post")
 */
public function myAction(Post $post)
{
}
```

### Exclude some parameters ###

You can exclude some attributes from being used by the converter:

If you have a route like `/my-route/{slug}/{name}/edit/{id}` you can exclude name
and slug by setting the option "*exclude*":

``` php
<?php

/**
 * @ParamConverter("post", class="BlogBundle\Model\Post", options={"exclude"={"name", "slug"}})
 */
public function myAction(Post $post)
{
}
```

### Custom mapping ###

You can map route parameters directly to model columns to be used for filtering.

If you have a route like `/my-route/{postUniqueName}/{AuthorId}` the *mapping* option
overwrite any other automatic mapping.

``` php
<?php

/**
 * @ParamConverter("post", class="BlogBundle\Model\Post", options={"mapping"={"postUniqueName":"name"}})
 * @ParamConverter("author", class="BlogBundle\Model\Author", options={"mapping"={"AuthorId":"id"}})
 */
public function myAction(Post $post, $author)
{
}
```

### Hydrate related object ###

You could hydrate related object with the "*with*" option:

``` php
<?php

/**
 * @ParamConverter("post", class="BlogBundle\Model\Post", options={"with"={"Comments"}})
 */
public function myAction(Post $post)
{
}
```

You can set multiple with `"with"={"Comments", "Author", "RelatedPosts"}`.

The default join is an "*inner join*" but you can configure it to be a left join
or right join instead:

``` php
<?php

/**
 * @ParamConverter("post", class="BlogBundle\Model\Post", options={"with"={ {"Comments", "left join" } }})
 */
public function myAction(Post $post)
{
}
```

Accepted parameters for join:

* left, LEFT, left join, LEFT JOIN, left_join, LEFT_JOIN
* right, RIGHT, right join, RIGHT JOIN, right_join, RIGHT_JOIN
* inner, INNER, inner join, INNER JOIN, inner_join, INNER_JOIN

## What's Next? ##

Now you are ready to use Propel with Symfony2. If you are interested, you can
also checkout these cookbooks:

* [Mastering Symfony2 Forms With Propel](mastering-symfony2-forms-with-propel.html)
* [The Symfony2 Security Component And Propel](the-symfony2-security-component-and-propel.html)
