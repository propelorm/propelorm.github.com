---
layout: documentation
title: What's new in Propel 2.0?
---

# What's new in Propel 2.0? #

Propel 2 represents an evolution of our favorite orm: it follows PSR-0, PSR-1, PRS-2, PSR-3 Php standards, it has a more rational architecture and some enhancements which improve its features and make your coding easier and more enjoyable. But it presents some backward compatibility breaks.
Migrating your projects from Propel 1.x to Propel 2 is easy, but it needs some work and a bit of attention.

## Quick update reference ##

This paragraph is written for all those programmers who hasn't time to go into details, by now. It exposes only those changes about Propel public api, without considering internal changes. Of course, we suggest to read in deep this chapter later.

Firstly, install Propel 2 via [Composer](http://getcomposer.org/), so that you have an autoload function ready to use, Propel classes namespace already set up and you can easily adjust your model namespaces, via `composer.json`.


|New feature|Explanation |Action to do|More details|
|:----------|------------|-----------|------------|
|New console|Propel command line has new commands with brand new syntax.|Run `propel` command without arguments and get a look at the new command list.|See paragraph [New console](#new-console).|
|PSR-0 standard|Both Propel and generated model classes follow PSR-0 standard. All classes have their namespace. Propel hasn't its autoload function anymore.|1) Use a PSR-0 autoload function to auto-load Propel classes and generated model. You can use the Composer one or your own. 2) Rebuild your model with namespaces feature enabled and adjust your autoload to correctly map your model classes namespace.|See paragraph [Namespaces](#psr-0-and-namespaces).|
|Removed Peer classes|All Peer classes were removed. Now Active Query is the first class citizen to perform queries. All methods, previously contained in Peer classes, now have an equivalent in Query classes and all constants were moved from Peer to TableMap classes.|Review your code to remove peer methods and substitute them with the ActiveQuery equivalent ones (i.e. `BookPeer::count()` now is `BookQuery::create()->count()`).|See paragraph [From Peer to ActiveQuery](#from-peer-to-activequery).|
|Removed validators|To properly follow the *Separation of Concerns* best practice, the validator system has been removed from Propel core.|Remove all references to `<validator>` tag in your schema.xml and rebuild your model. Use an external library to validate your data (i.e. [Zend Validator](http://framework.zend.com/manual/2.2/en/modules/zend.validator.html), [Symfony Validator](http://symfony.com/doc/current/book/validation.html), [Valitron](https://github.com/vlucas/valitron) or [Respect](https://github.com/Respect/Validation)). If you prefer to still define your validators inside your `schema.xml`, you can use the new `Validate behavior`.|See http://propelorm.org/behaviors/validate.html|
|Propel\Runtime\Propel class refactor|Some `Propel` class methods have been renamed and now we have a `ServiceContainer` class.|If you don't explicit call `Propel` class methods in your code, simply rebuild your model. Otherwise, review your code to find all those methods that now are included in ServiceContainer or have been renamed.|See paragraph [Service Container](#propel-class-refactor-and-service-container)|
|Configuration|Propel now comes with a new configuration system, which supports more formats (php, yaml, json, ini, xml). Now all configuration properties are in a single file.|Remove `build.properties`, `runtime-conf.xml`, `buildtime-conf.xml` and write all properties in `propel.ext` file ('.ext' means one of the supported extensions of your choice)|Strongly recommended to read [Configuration](10-configuration.html)|


## New console ##

Propel 2 comes with a brand new console, based on [Symfony Console Component](http://symfony.com/doc/current/components/console/index.html).
As usual, you can see a full list of available commands simply running from your command line:

```bash
vendor/bin/propel
```

Here is an extract of the output:

```bash
config
  config:convert-xml   Transform the XML configuration to PHP code leveraging the ServiceContainer

database
  database:reverse     Reverse-engineer a XML schema file based on given database

graphviz
  graphviz:generate    Generate Graphviz files (.dot)

migration
  migration:diff       Generate diff classes
  migration:down       Execute migrations down
  migration:migrate    Execute all pending migrations
  migration:status     Get migration status
  migration:up         Execute migrations up

model
  model:build          Build the model classes based on Propel XML schemas

sql
  sql:build            Build SQL files
  sql:insert           Insert SQL statements

test
  test:prepare         Prepare the Propel test suite by building fixtures
```

If you've run `bin/propel`, you can see some other commands, not included in the previous panel, listed in your shell output. These commands are still here for compatibility with Propel 1.x, but they're deprecated and (probably) they'll be removed in future versions.

The Phing tasks that you used at buildtime are now refactored to Commands. They work the same, but their name has changed:

| Old task      | New command       |
|---------------|-------------------|
| build-model   | model:build       |
| build-sql     | sql:build         |
| insert-sql    | sql:insert        |
| convert-conf  | config:convert-xml|
| diff          | migration:diff    |
| status        | migration:status  |
| migrate       | migration:migrate |
| up            | migration:up      |
| down          | migration:down    |
| reverse       | database:reverse  |


## PSR-0 and namespaces ##

Propel 2 follows the [PSR-0](http://www.php-fig.org/psr/0/) standard, both for its internal and generated classes.

If you've never worked with namespace before, please read the [PHP official documentation](http://www.php.net/manual/en/language.namespaces.php) about it.

Propel directory structure has changed to adhere to the standard and class namespaces map this. Propel has no more an internal autoload function and you can use any PSR-0 autoload you prefer. If you install Propel via Composer, a PSR-0 autoloader is automatically provided and configured and you can use it simply including the relative file:

```php
<?php

require_once 'vendor/autoload.php';
```

You should adjust your schema.xml to work with namespaces, according to Propel documentation (see http://www.propelorm.org/cookbook/namespaces.html) and rebuild your model.
Now you are able to easily autoload your model classes:

```php
<?php

use My\Model\Namespace\ModelClass;

$class = new ModelClass();
```

## From Peer to ActiveQuery ##

In earlier versions, Propel provided database queries via some generated Peer classes.
Starting from version 1.5, a new object oriented query API, called [Active Query](/documentation/reference/model-criteria.html), was introduced. This feature was immediately a first class citizen in Propel world, near the good old Peer classes.
Now, Propel 2 leaves Peers, which are no longer available, and Active Query generated classes are the only Propel query api (of course, it's still possible to directly execute SQL code).

If you've never used Active Query before, but only Peer classes, please read this [blog post](http://propelorm.org/blog/2010/08/03/refactoring-to-propel-1-5-from-peer-classes-to-query-classes.html).

All Peer methods are moved into Query classes, if they didn't already have an equivalent.
All instance pool references, previously contained in Peer, now have its own Instance Pool Trait.
All Peer constants, about tables structure, now are in TableMap classes.

Please, refer to [Propel API](http://api.propelorm.org/2.0-master/) and read the generated Query and TableMap classes, for other details.


## Adapter, Connection and Statement classes ##

Propel 2 introduces an its own abstraction layer, based on PDO interface, to overcome some PDO limitations. Besides, thanks to this feature, now it's possible to write a database adapter based on a non-PDO extension.

Propel abstraction layer is based on the following classes:

*  `Propel\Runtime\Connection\ConnectionInterface`: Propel connection object, based on PHP builtin PDO class
*  `Propel\Runtime\Connection\StatementInterface`: Propel statement object, dased on PHP builtin PDOStatement interface
*  `Propel\Runtime\Adapter\AdapterInterface`: Propel interface for specific DBMS adapters

All PDO connection references, now are `ConnectionInterface` onces and all PDOStatement objects now are `StatementInterface`.


## Propel class refactor and Service Container ##

`ServiceContainer` class includes all methods, properties and constants relate to "shared" services: connections, adapters, data sources, database map, logger and profiler.
Some ServiceContainer informations was previously included in `Propel` class:

| Replace...                 | With...                                       |
|----------------------------|-----------------------------------------------|
| `Propel::CONNECTION_WRITE` | `ServiceContainerInterface::CONNECTION_WRITE` |
| `Propel::CONNECTION_READ`  | `ServiceContainerInterface::CONNECTION_READ`  |


Some static methods from the `Propel` class have been renamed. Therefore, you must replace the following occurrences in your code:

    Replace...                                              With...

    Propel::getDB($name)                                    Propel::getAdapter($name)
    Propel::getConnection($name, Propel::CONNECTION_READ)   Propel::getReadConnection($name)
    Propel::getConnection($name, Propel::CONNECTION_WRITE)  Propel::getWriteConnection($name)
    Propel::getDefaultDB()                                  Propel::getDefaultDatasource()


The generated model is automatically updated once you rebuild your model.

> **Tip**<br>Internally, `Propel::getAdapter()` proxies to `Propel::getServiceContainer()->getAdapter()`.
> The `Propel` class was refactored to keep only one static class and to be more extensible.
> It remains the easy entry point to all the necessary services provided by Propel.<br>

Also, `Propel::setConnection()` was removed.

The instruction to initialize Propel

```php
<?php

Propel::init($configFilePath);
```
now is deprecated and should be replaced with

```php
<?php

include $configFilePath;
```

`$configFilePath` is the path to the PHP configuration generated by the `config:convert` command.

## Configuration ##

The way of configuring Propel is changed. Now there's only one configuration file and more formats are supported. You can write your configuration file in plain php, ini, json, yaml or xml format. Read the [Configuration chapter](10-configuration.html) of the documetation and [configuration reference](/documentation/reference/configuration-file.html) chapter for a complete explanation of configuration properties.

## Migrations and reverse engineering ##

In addition to *Mysql*, now migrations system supports *SQLite* and *PostgreSQL*.
The schema reverse engineering now supports also *SQLite*, so you can generate a `schema.xml` file from an existing SQLite database.

## Other Refactors ##

All the refactors, described in this paragraph, affect your project only if you
extended one of the modified classes. In all the other cases, just rebuild your
model.

#### Builders renamed ####

The classes used by Propel internally to build the object model were renamed.
This affects your project if you extended one of these classes.

| Replace...                       | With...                     |
|----------------------------------|-----------------------------|
| OMBuilder.php                    | AbstractOMBuilder.php       |
| ObjectBuilder.php                | AbstractObjectBuilder.php   |
| PeerBuilder.php                  |  __removed__                |
| PHP5ExtensionObjectBuilder.php   | ExtensionObjectBuilder.php  |
| PHP5ExtensionPeerBuilder.php     | __removed__                 |
| PHP5InterfaceBuilder.php         |InterfaceBuilder.php         |
| PHP5MultiExtendObjectBuilder.php | MultiExtendObjectBuilder.php|
| PHP5ObjectBuilder.php            | ObjectBuilder.php           |
| PHP5PeerBuilder.php              | __removed__                 |
| PHP5TableMapBuilder.php          | TableMapBuilder.php         |

#### Base classes reorganized ####

Base classes are generated in a `Base` directory, and base classes are no more prefixed by `Base`.
Parameters `basePrefix` and `namespaceOm` have been removed

#### Model\Database methods renamed ####

Before:

```php
<?php

    public function setAppData(AppData $parent)
```

After:

```php
<?php

    public function setParentSchema(Schema $parent)
```

Before:

```php
<?php

    public function getAppData()
```

After:

```php
<?php

    public function getParentSchema()
```

#### Model\AppData class has been renamed to Schema ####

The `AppData::joinAppDatas` method has been renamed to `Schema::joinSchemas`.

Before:

```php
<?php

    public function joinAppDatas($ads)
```

After:

```php
<?php

    public function joinSchemas($schemas)
```

#### Builder\Util\XmlToAppData class has been renamed to SchemaReader ####

Both `SchemaReader::parseString` and `SchemaReader::parseFile` methods return
a `Propel\Generator\Model\Schema` object instead of a `Propel\Generator\Model\AppData` object.

#### Util\SchemaValidator constructor signature changed ####

Before:

```php
<?php

    public function __construct(AppData $appData)
```

After:

```php
<?php

    public function __construct(Schema $schema)
```

#### ActiveQuery\Criteria::addOr refactor ####

`Propel\Runtime\ActiveQuery\Criteria::addOr` operates only on existing conditions, where prior it would `OR` to a condition to on the same column (if it existed).

E.g.

```php
<?php

$criteria->add('column1', 'value');
$criteria->add('column2', 'value');
$criteria->addOr('column1', 'value'); // this used to be OR`ed to the first add (matched by name)
```

All base object methods have been merged in generated Base Object classes. This could break, behaviors that call `parent` methods

#### Om\Persistent Interface replaced by Om\ActiveRecordInterface ####

All methods from `Propel\Runtime\Om\Persistent` Interface have been removed.
All Base Object classes now implements `Propel\Runtime\Om\ActiveRecordInterface` this could be use to identify a Propel Object.

#### Util\PropelModelPager::count() ####

`Propel\Runtime\Util\PropelModelPager::count()` now return the number of items in the collection.
To get the old behavior use `Propel\Runtime\Util\PropelModelPager::getNbResults()`.

#### But also... ####

The following methods have been removed (under the `Propel` namespace):

* `Generator\Model\Table::containsColumn`
* `Generator\Model\VendorInfo::addParameter`
* `Generator\Model\Column::setTypeFromString`
* `Generator\Model\Column::getPropelType`
* `Generator\Model\Column::getDefaultSetting`
* `Generator\Model\Column::makeList`
* `Generator\Model\ForeignKey::getLocalColumnNames`
* `Generator\Model\ForeignKey::getForeignColumnNames`
* `Generator\Model\Table::printPrimaryKey`
* `Generator\Model\Table::printList`
* `Generator\Model\PropelTypes::getCreoleTypes`
* `Runtime\Om\BaseObject`

The following methods and classes have been renamed (under the `Propel` namespace):

| Replace....                               | With...                                     |
| ----------------------------------------- | ------------------------------------------- |
| `Generator\Model\ScopedElement`           | `Generator\Model\ScopedMappingModel`        |
| `Generator\Model\XmlElement`              | `Generator\Model\MappingModel`              |
| `Generator\Model\XmlElement::loadFromXml` | `Generator\Model\MappingModel::loadMapping` |
| `Generator\Model\Column::printSize`       | `Generator\Model\Column::getSizeDefinition` |
| `Generator\Model\Column::getStudlyPhpName`| `Generator\Model\Column::getCamelCaseName`  |
| `Generator\Model\Domain::printSize`       | `Generator\Model\Domain::getSizeDefinition` |
