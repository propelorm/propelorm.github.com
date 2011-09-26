---
layout: documentation
title: Working With Symfony2
---

# Working With Symfony2 #

The [PropelBundle](http://www.github.com/propelorm/PropelBundle) eases the integration of Propel in Symfony2.

It currently supports:

* Generation of model classes based on an XML schema (not YAML) placed under `BundleName/Resources/*schema.xml`.
* Insertion of SQL statements.
* Runtime autoloading of Propel and generated classes.
* Propel runtime initialization through the XML configuration.
* Migrations [Propel 1.6](../documentation/10-migrations.html).
* Reverse engineering from [existing database](working-with-existing-databases.html).
* Integration to the Symfony2 Profiler.
* Load SQL and XML fixtures.
* Create/Drop databases.
* Dump data into XML and SQL.


## Installation

Clone this bundle in the `vendor/bundles/Propel` directory:

    git submodule add https://github.com/propelorm/PropelBundle.git vendor/bundles/Propel/PropelBundle

Checkout Propel and Phing in the `vendor` directory:

    svn checkout http://svn.github.com/propelorm/Propel.git vendor/propel

    svn checkout http://svn.phing.info/tags/2.4.6/ vendor/phing

Instead of using svn, you can clone the unofficial Git repositories:

    git submodule add https://github.com/Xosofox/phing vendor/phing

    git submodule add https://github.com/propelorm/Propel.git vendor/propel

Register this bundle in the `AppKernel` class:

{% highlight php %}
<?php

public function registerBundles()
{
    $bundles = array(
        ...

        // PropelBundle
        new Propel\PropelBundle\PropelBundle(),
        // register your bundles
        new Sensio\HelloBundle\HelloBundle(),
    );

    ...
}
{% endhighlight %}

Don't forget to register the PropelBundle namespace in `app/autoload.php`:

{% highlight php %}
<?php

$loader->registerNamespaces(array(
    ...

    'Propel' => __DIR__.'/../vendor/bundles',
));
{% endhighlight %}

To finish, add the following configuration `app/config/propel.ini`:

{% highlight ini %}
# Enable full use of the DateTime class.
# Setting this to true means that getter methods for date/time/timestamp
# columns will return a DateTime object when the default format is empty.
propel.useDateTimeClass = true

# Specify a custom DateTime subclass that you wish to have Propel use
# for temporal values.
propel.dateTimeClass = DateTime

# These are the default formats that will be used when fetching values from
# temporal columns in Propel. You can always specify these when calling the
# methods directly, but for methods like getByName() it is nice to change
# the defaults.
# To have these methods return DateTime objects instead, you should set these
# to empty values
propel.defaultTimeStampFormat = Y-m-d H:i:s
propel.defaultTimeFormat =
propel.defaultDateFormat =
{% endhighlight %}

## Sample Configuration

### Project configuration

{% highlight yaml %}
# in app/config/config.yml
propel:
    path:       "%kernel.root_dir%/../vendor/propel"
    phing_path: "%kernel.root_dir%/../vendor/phing"
#    logging:   %kernel.debug%
#    build_properties:
#        xxxxx.xxxxx: xxxxxx
#        xxxxx.xxxxx: xxxxxx

# in app/config/config*.yml
propel:
    dbal:
        driver:               mysql
        user:                 root
        password:             null
        dsn:                  mysql:host=localhost;dbname=test;charset=UTF8
        options:              {}
        attributes:           {}
#        default_connection:       default
#        connections:
#           default:
#               driver:             mysql
#               user:               root
#               password:           null
#               dsn:                mysql:host=localhost;dbname=test
#               options:
#                   ATTR_PERSISTENT: false
#               attributes:
#                   ATTR_EMULATE_PREPARES: true
#               settings:
#                   charset:        { value: UTF8 }
#                   queries:        { query: 'INSERT INTO BAR ('hey', 'there')' }
{% endhighlight %}

`options`, `attributes` and `settings` are parts of the runtime configuration. See [Runtime Configuration File](../reference/runtime-configuration) documentation for more explanation.


### Build properties

You can define _build properties_ by creating a `propel.ini` file in `app/config` and put build properties (see [Build properties Reference](../reference/buildtime-configuration)).

{% highlight ini %}
# in app/config/propel.ini
xxxx.xxxx.xxxx = XXXX
{% endhighlight %}

But you can follow the Symfony2 way by adding build properties in `app/config/config.yml`:

{% highlight yaml %}
# in app/config/config.yml
propel:
    build_properties:
        xxxxx.xxxx.xxxxx:   XXXX
        xxxxx.xxxx.xxxxx:   XXXX
        ...
{% endhighlight %}


### Sample Schema

Place the following schema in `src/Sensio/HelloBundle/Resources/config/schema.xml` :

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<database name="default" namespace="Sensio\HelloBundle\Model" defaultIdMethod="native">

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
{% endhighlight %}


## Commands

### Build Process

Call the application console with the `propel:build` command:

    php app/console propel:build [--classes] [--sql] [--insert-sql]


### Insert SQL

Call the application console with the `propel:insert-sql` command:

    php app/console propel:insert-sql [--force]

Note that the `--force` option is needed to actually execute the SQL statements.


### Use The Model Classes

Use the Model classes as any other class in Symfony2. Just use the correct namespace, and Symfony2 will autoload them:

{% highlight php %}
<?php

class HelloController extends Controller
{
    public function indexAction($name)
    {
        $author = new \Sensio\HelloBundle\Model\Author();
        $author->setFirstName($name);
        $author->save();

        return $this->render('HelloBundle:Hello:index.html.twig', array('name' => $name, 'author' => $author));
    }
}
{% endhighlight %}


### Migrations

Generates SQL diff between the XML schemas and the current database structure:

    php app/console propel:migration:generate-diff

Executes the migrations:

    php app/console propel:migration:migrate

Executes the next migration up:

    php app/console propel:migration:migrate --up

Executes the previous migration down:

    php app/console propel:migration:migrate --down

Lists the migrations yet to be executed:

    php app/console propel:migration:status


### Working with existing databases

Run the following command to generate an XML schema from your `default` database:

    php app/console propel:reverse

You can define which connection to use:

    php app/console propel:reverse --connection=default


### Fixtures

You can load your own fixtures by using the following command:

    php app/console propel:fixtures:load [-d|--dir[="..."]] [--xml] [--sql] [--yml] [--connection[="..."]]

As usual, `--connection` allows to specify a connection.

The `--dir` option allows to specify a directory containing the fixtures (default is: `app/propel/fixtures/`).
Note that the `--dir` expects a relative path from the root dir (which is `app/`).

The `--xml` parameter allows you to load only XML fixtures.
The `--sql` parameter allows you to load only SQL fixtures.
The `--yml` parameter allows you to load only YAML fixtures.

You can mix `--xml`, `--yml` and `--sql` parameters to load XML, YAML and SQL fixtures.
If none of this parameter are set all files YAML, XML and SQL in the directory will be load.

A valid _XML fixtures file_ is:

{% highlight xml %}
<Fixtures>
    <Object Namespace="Awesome">
        <o1 Title="My title" MyFoo="bar" />
    </Object>
    <Related Namespace="Awesome">
        <r1 ObjectId="o1" Description="Hello world !" />
    </Related>
</Fixtures>
{% endhighlight %}

A valid _YAML fixtures file_ is:

{% highlight yaml %}
\Awesome\Object:
     o1:
         Title: My title
         MyFoo: bar

 \Awesome\Related:
     r1:
         ObjectId: o1
         Description: Hello world !
{% endhighlight %}

You can dump data into YAML fixtures file by using this command:

    php app/console propel:fixtures:dump [--connection[="..."]]

Dumped files will be written in the fixtures directory: `app/propel/fixtures/` with the following name: `fixtures_99999.yml` where `99999`
is a timestamp.
Once done, you will be able to load this files by using the `propel:fixtures:load` command.


### Graphviz

You can generate **Graphviz** file for your project by using the following command line:

    php app/console propel:graphviz

It will write files in `app/propel/graph/`.


### Database

You can create a **database**:

    php app/console propel:database:create [--connection[=""]]

As usual, `--connection` allows to specify a connection.


You can drop a **database**:

    php app/console propel:database:drop [--connection[=""]] [--force]

As usual, `--connection` allows to specify a connection.

Note that the `--force` option is needed to actually execute the SQL statements.
