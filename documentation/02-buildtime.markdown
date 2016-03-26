---
layout: documentation
title: The Build Time
configuration: true
---

# The Build Time #

The initial step in every Propel project is the "build". During build time, a developer describes the structure of the datamodel in a XML file called the "schema".

From this schema, Propel generates PHP classes, called "model classes", made of object-oriented PHP code optimized for a given RDMBS. The model classes are the primary interface to find and manipulate data in the database in Propel.

The XML schema can also be used to generate SQL code to setup your database.
Alternatively, you can generate the schema from an existing database (see the
[Working with existing databases guide](/documentation/cookbook/working-with-existing-databases.html) for more details).

During build time, a developer also defines the connection settings for communicating with the database.

To illustrate Propel's build abilities, this chapter uses the data structure of a bookstore as an example. It is made of three tables: a `book` table, with a foreign key to two other tables, `author` and `publisher`.

Create a `bookstore` directory then setup Propel into it as covered in the previous
chapter. This will be the root of the bookstore project.

There's actually two ways to setup the build configuration ; the hard one (manually)
and the easy one (through the command line). This chapter explains both. Consider
that the command line approach is just here to simplify the process. You will
learn more about the Propel structure and API setting up your project the hard way.

## The Hard Way ##

### Describing Your Database as XML Schema ###

Propel generates PHP classes based on a _relational_ description of your data
model. This "schema" uses XML to describe tables, columns and relationships. The
schema syntax closely follows the actual structure of the database (you have
to describe any excluded tables inside your
[configuration file](/documentation/reference/configuration-file.html#exclude-tables)).

#### Database Connection Name ####

Create a file called `schema.xml` in the new `bookstore/` directory.

The root tag of the XML schema is the `<database>` tag:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<database name="bookstore" defaultIdMethod="native">
  <!-- table definitions go here -->
</database>
```

The `name` attribute defines the name of the connection that Propel uses for the
tables in this schema. It is not necessarily the name of the actual database. In
fact, Propel uses some configuration properties to link a connection name with real
connection settings (like the database name, user and password).

The `defaultIdMethod` attribute indicates that the tables in this schema use the database's "native" auto-increment/sequence features to handle id columns that are set to auto-increment.

> **Tip** You can define several schemas for a single project. Just make sure
> that each of the schema filenames end with `schema.xml`.

#### Tables And Columns ####

Within the `<database>` tag, Propel expects a `<table>` tag for each table:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<database name="bookstore" defaultIdMethod="native">
  <table name="book" phpName="Book">
    <!-- column and foreign key definitions go here -->
  </table>
  <table name="author" phpName="Author">
    <!-- column and foreign key definitions go here -->
  </table>
  <table name="publisher" phpName="Publisher">
    <!-- column and foreign key definitions go here -->
  </table>
</database>
```

This time, the `name` attributes are the real table names. The `phpName` is the name that Propel will use for the generated PHP class. By default, Propel uses a CamelCase version of the table name as its phpName - that means that you could omit the `phpName` attribute in the example above.

Within each set of `<table>` tags, define the columns that belong to that table:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<database name="bookstore" defaultIdMethod="native">
  <table name="book" phpName="Book">
    <column name="id" type="integer" required="true" primaryKey="true" autoIncrement="true"/>
    <column name="title" type="varchar" size="255" required="true" />
    <column name="isbn" type="varchar" size="24" required="true" phpName="ISBN"/>
    <column name="publisher_id" type="integer" required="true"/>
    <column name="author_id" type="integer" required="true"/>
  </table>
  <table name="author" phpName="Author">
    <column name="id" type="integer" required="true" primaryKey="true" autoIncrement="true"/>
    <column name="first_name" type="varchar" size="128" required="true"/>
    <column name="last_name" type="varchar" size="128" required="true"/>
  </table>
  <table name="publisher" phpName="Publisher">
   <column name="id" type="integer" required="true" primaryKey="true" autoIncrement="true" />
   <column name="name" type="varchar" size="128" required="true" />
  </table>
</database>
```

Each column has a `name` (the one used by the database), and an optional `phpName` attribute. Once again, the Propel default behavior is to use a CamelCase version of the `name` as `phpName` when not specified.

Each column also requires a `type`. The XML schema is database agnostic, so the
column types and attributes are probably not exactly the same as the ones you
use in your own database. But Propel knows how to map the schema types with SQL
types for many database vendors. Check the [schema reference][schema] for more
details on each column type.

As for the other column attributes, `required`, `primaryKey`, and `autoIncrement`, they mean exactly what their names imply.

> **Tip** If you specify a `namespace` attribute in a `<table>` element, the
> generated PHP classes for this table will use this namespace.

#### Foreign Keys ####

A table can have several `<foreign-key>` tags, describing foreign keys to foreign tables. Each `<foreign-key>` tag consists of one or more mappings between a local column and a foreign column.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<database name="bookstore" defaultIdMethod="native">
  <table name="book" phpName="Book">
    <column name="id" type="integer" required="true" primaryKey="true" autoIncrement="true"/>
    <column name="title" type="varchar" size="255" required="true" />
    <column name="isbn" type="varchar" size="24" required="true" phpName="ISBN"/>
    <column name="publisher_id" type="integer" required="true"/>
    <column name="author_id" type="integer" required="true"/>
    <foreign-key foreignTable="publisher" phpName="Publisher" refPhpName="Book">
      <reference local="publisher_id" foreign="id"/>
    </foreign-key>
    <foreign-key foreignTable="author">
      <reference local="author_id" foreign="id"/>
    </foreign-key>
  </table>
  <table name="author" phpName="Author">
    <column name="id" type="integer" required="true" primaryKey="true" autoIncrement="true"/>
    <column name="first_name" type="varchar" size="128" required="true"/>
    <column name="last_name" type="varchar" size="128" required="true"/>
  </table>
  <table name="publisher" phpName="Publisher">
   <column name="id" type="integer" required="true" primaryKey="true" autoIncrement="true" />
   <column name="name" type="varchar" size="128" required="true" />
  </table>
</database>
```

A foreign key represents a relationship. Just like a table or a column, a relationship has a `phpName`. By default, Propel uses the `phpName` of the foreign table as the `phpName` of the relation. The `refPhpName` defines the name of the relation as seen from the foreign table.

There are many more attributes and elements available to describe a datamodel.
Propel's documentation provides a complete [reference of the schema syntax][schema],
together with a [DTD][DTD] and a [XSD][XSD] schema for its validation.

### Building The Model ###

#### Setting Up Configuration ####

The build process is highly customizable. Whether you need the generated classes to inherit one of your classes rather than Propel's base classes, or to enable/disable some methods in the generated classes, pretty much every customization is possible. Of course, Propel provides sensible defaults, so that you actually need to define only two settings for the build process to start: the RDBMS you are going to use, and a name for your project.

Propel expects the configuration to be stored in a file called `propel.ext`, where `.ext` stands for "one of the supported extensions" and stored at the same level as the `schema.xml`, or in a subdirectory named `conf` or `config`. See the [configuration chapter](10-configuration.html) of the documentation, for a detailed description.
Here's a minimal example configuration file for MySQL, for each supported format

<div class="conftabs">
<ul>
<li><a href="#tabyaml">propel.yaml</a></li>
<li><a href="#tabphp">propel.php</a></li>
<li><a href="#tabjson">propel.json</a></li>
<li><a href="#tabini">propel.ini</a></li>
<li><a href="#tabxml">propel.xml</a></li>
</ul>
<div id="tabyaml">
{% highlight yaml %}
propel:
  database:
      connections:
          bookstore:
              adapter: mysql
              classname: Propel\Runtime\Connection\ConnectionWrapper
              dsn: "mysql:host=localhost;dbname=my_db_name"
              user: my_db_user
              password: s3cr3t
              attributes:
  runtime:
      defaultConnection: bookstore
      connections:
          - bookstore
  generator:
      defaultConnection: bookstore
      connections:
          - bookstore
{% endhighlight %}
</div>
<div id="tabphp">
{% highlight php %}
<?php

return [
    'propel' => [
        'database' => [
            'connections' => [
                'bookstore' => [
                    'adapter'    => 'mysql',
                    'classname'  => 'Propel\Runtime\Connection\ConnectionWrapper',
                    'dsn'        => 'mysql:host=localhost;dbname=my_db_name',
                    'user'       => 'my_db_user',
                    'password'   => 's3cr3t',
                    'attributes' => []
                ]
            ]
        ],
        'runtime' => [
            'defaultConnection' => 'bookstore',
            'connections' => ['bookstore']
        ],
        'generator' => [
            'defaultConnection' => 'bookstore',
            'connections' => ['bookstore']
        ]
    ]
];
{% endhighlight %}
</div>
<div id="tabjson">
{% highlight json %}
{
    "propel": {
        "database": {
            "connections": {
                "bookstore": {
                    "adapter": "mysql",
                    "classname": "Propel\Runtime\Connection\ConnectionWrapper",
                    "dsn": "mysql:host=localhost;dbname=my_db_name",
                    "user": "my_db_user",
                    "password": "s3cr3t",
                    "attributes": []
                }
            }
        },
        "runtime": {
            "defaultConnection": "bookstore",
            "connections": ["bookstore"]
        },
        "generator": {
            "defaultConnection": "bookstore",
            "connections": ["bookstore"]
        }
    }
}
{% endhighlight %}
</div>
<div id="tabini">
{% highlight ini %}
[propel]
;
; Database section
;
database.connections.bookstore.adapter    = mysql
database.connections.bookstore.classname  = Propel\Runtime\Connection\ConnectionWrapper
database.connections.bookstore.dsn        = mysql:host=localhost;dbname=my_db_name
database.connections.bookstore.user       = my_db_user
database.connections.bookstore.password   = s3cr3t
database.connections.bookstore.attributes =

;
; Runtime section
;
runtime.defaultConnection = bookstore
runtime.connections[0]    = bookstore

;
; Generator section
;
generator.defaultConnection = bookstore
generator.connections[0] = bookstore
{% endhighlight %}
</div>
<div id="tabxml">
{% highlight xml %}
<?xml version="1.0" encoding="ISO-8859-1" standalone="no"?>
<config>
    <propel>
        <database>
            <connections>
                <connection id="bookstore">
                    <adapter>mysql</adapter>
                    <classname>Propel\Runtime\Connection\ConnectionWrapper</classname>
                    <dsn>mysql:host=localhost;dbname=my_db_name</dsn>
                    <user>my_db_user</user>
                    <password>s3cr3t</password>
                    <attributes></attributes>
                </connection>
            </connections>
        </database>
        <runtime>
            <defaultConnection>bookstore</defaultConnection>
            <connection>bookstore</connection>
        </runtime>
        <generator>
            <defaultConnection>bookstore</defaultConnection>
            <connection>bookstore</connection>
        </generator>
    </propel>
</config>
{% endhighlight %}
</div>
</div>

Use your own database vendor driver, chosen among pgsql, mysql, sqlite, mssql
and oracle.

You can learn more about the available build settings and their possible values in
the [configuration reference](/documentation/reference/configuration-file.html).

#### Setup UTF-8 ####

If you want to save anything in UTF-8 in your database then depending on your database you have to set some extra configuration values.

<div class="conftabs">
<ul>
<li><a href="#tabMysql">MySQL</a></li>
<li><a href="#tabPgsql">PostgreSQL</a></li>
<li><a href="#tabSqlite">SQLite</a></li>
<li><a href="#tabMssql">MSSQL</a></li>
<li><a href="#tabOracle">Oracle</a></li>
</ul>
<div id="tabMysql">
{% highlight yaml %}
propel:
  database:
    connections:
      default:
        adapter: mysql
        settings:
          charset: utf8mb4
          queries:
            utf8: "SET NAMES utf8mb4 COLLATE utf8mb4_unicode_ci, COLLATION_CONNECTION = utf8mb4_unicode_ci, COLLATION_DATABASE = utf8mb4_unicode_ci, COLLATION_SERVER = utf8mb4_unicode_ci"
{% endhighlight %}
</div>
<div id="tabPgsql">
{% highlight yaml %}
propel:
  database:
    connections:
      default:
        adapter: pgsql
        settings:
          charset: utf8
          queries:
            utf8: "SET NAMES 'UTF8'"
{% endhighlight %}
</div>
<div id="tabSqlite">
{% highlight yaml %}
propel:
  database:
    connections:
      default:
        adapter: sqlite
        settings:
          charset: utf8
{% endhighlight %}
</div>
<div id="tabMssql">
{% highlight yaml %}
propel:
  database:
    connections:
      default:
        adapter: mssql
        settings:
          charset: UTF-8
{% endhighlight %}
</div>
<div id="tabOracle">
{% highlight yaml %}
propel:
  database:
    connections:
      default:
        adapter: oracle
        settings:
          charset: UTF8
{% endhighlight %}
</div>
</div>

#### Using the `propel` Script To Build The SQL Code ####

As seen in the previous chapter, Propel ships with a script that allows you to
realize different actions such as schema generation.

Open a terminal inside your project's directory (here `bookstore/`), where you
saved the two previous files (`schema.xml` and `propel.ext`). Then use the `propel`
script to generate the SQL code of your schema:

```bash
$ propel sql:build
```

#### Generate Model Classes ####

Now that your database is ready, we are going to generate our model files. These files are just classes that allows you to interact easily with your different tables. To generate these tables, just run:

```bash
$ propel model:build
```

Propel will generate a new `generated-classes` folder containing all the stuff you need to interact with your different tables.

For every table in the database, Propel creates 3 PHP classes:

* a _model_ class (e.g. `Book`), which represents a row in the database;
* a _tablemap_ class (e.g. `Map\BookTableMap`), offering static constants and methods mostly for compatibility with previous Propel versions;
* a _query_ class (e.g. `BookQuery`), used to operate on a table to retrieve and update rows

Propel uses the `phpName` attribute of each table as the base for the PHP class names.

All these classes are empty, but they inherit from `Base` classes that you will find under the `Base/` directory in the `generated-classes` one:

```php
<?php

/**
 * Skeleton subclass for representing a row from the 'book' table.
 */
class Book extends BaseBook
{

}
```

These empty classes are called _stub_ classes. This is where you will add your own model code. These classes are generated only once by Propel ; on the other hand, the _base_ classes they extend are overwritten every time you call the `model:build` command, and that happens a lot in the course of a project, because the schema evolves with your needs.


Propel also generates one `TableMap` class for each table under the `Map/`
directory. You will probably never use the map classes directly, but Propel needs
them to get metadata information about the table structure at runtime.

> **Tip** Never add any code of your own to the classes generated by Propel in the
> `Map/` directory; this code would be lost next time you call the `model:build`
> command.

Basically, all that means is that despite the fact that Propel generates _five_ classes for each table, you should only care about two of them: the model class and the query class.


> **Warning** After generating the classes, you have to autoload them. For example,
> with composer this can be achieved with:
>
> ```json
> {
>   ...
>   "autoload": {
>     "classmap": ["generated-classes/"]
>   }
> }
> ```
>
> and then executing the command `composer dump-autoload`. See also
> [namespaces recipe](../cookbook/namespaces.html) if you prefer to autoload
> your model classes following PSR-0.

### Runtime Connection Settings ###

The database and PHP classes are now ready to be used. But they don't know yet
how to communicate with each other at runtime. You must tell Propel which database
connection settings should be used to finish the setup.

For performance reasons, Propel prefers to use a PHP version of the connection
settings rather than read it from the configuration file every time. So you must
use the `propel` script one last time to build the PHP version of the runtime
configuration:

```bash
$ propel config:convert
```

The `config:convert` command reads the `runtime` section of the configuration file
and generates the relative PHP script. The resulting file can be found under
`generated-conf/config.php`.

You can now setup Propel with the following script:

```php
<?php

// setup the autoloading
require_once '/path/to/vendor/autoload.php';

// setup Propel
require_once '/generated-conf/config.php';
```

It's also a good practice to add a logger to the service container, so that Propel
can log warnings and errors. You can do so by adding the following code to the
setup script:

```php
<?php

use Monolog\Logger;
use Monolog\Handler\StreamHandler;

$defaultLogger = new Logger('defaultLogger');
$defaultLogger->pushHandler(new StreamHandler('/var/log/propel.log', Logger::WARNING));

$serviceContainer->setLogger('defaultLogger', $defaultLogger);
```

> **Tip** You may wish to write the setup code in a standalone script that is
included at the beginning of your PHP files.

Now you are ready to start using your model classes!

#### Create the Database Schema ####

Now that your project is fully set up, you have to create the generated schema
in your database.

Before inserting it, you should create a database, let's say we want to call it
`bookstore`. If you are using MySQL for instance, just run:

```bash
$ mysqladmin -u root -p create bookstore
```

Then insert the SQL into your database:

```bash
$ propel sql:insert
```

You should normally have your tables created. Propel will also generate a
`generated-sql` folder containning the SQL files of your schema ; useful if you
are using a SCM, you can so compare the different versions of your schema.

Each time you will update your schema, you should run `sql:build` and `sql:insert`.

Depending on which RDBMS you are using, it may be normal to see some errors
(e.g. "unable to DROP...") when you first run this command. This is because some
databases have no way of checking to see whether a database object exists before
attempting to DROP it (MySQL is a notable exception). It is safe to disregard
these errors, and you can always run the script a second time to make sure that
the errors are no longer present.

> **Warning** The `schema.sql` file will DROP any existing table before
creating them, which will effectively erase your database.

## The easy way ##

To ease the setup of your build configuration, Propel ships with an `init` command
that will guide you through this process. Just type, in a terminal:

```bash
$ propel init
```

Propel will ask first for your database driver and then your database credentials.
It will ask you again if it's not able to establish a connection to the database.

Then, Propel will ask you if you want to import an existing database into your
project and where to store the different Propel specific files like the schema.

Finally, you need to specify the format of you configuration file. YAML is a
good default but you can pick the format you want ; the documentation explains
each one.

The command will also output a summary of your current configuration that you can
modify if you want to. Propel is now ready to be used!

---
<span class="next">[Next: Basic CRUD &rarr;](03-basic-crud.html)</span>

[schema]: /documentation/reference/schema.html
[DTD]: https://github.com/propelorm/Propel2/blob/master/resources/dtd/database.dtd
[XSD]: https://github.com/propelorm/Propel2/blob/master/resources/xsd/database.xsd
