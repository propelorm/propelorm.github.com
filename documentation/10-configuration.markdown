---
layout: documentation
title: Configuration
---
# Configuring Propel #

Propel asks you to define some data to work properly, for instance: connection parameters, working directories, flags to take decisions and so on.
You can pass these data via a *configuration file*.

*Note* : Configuration files are used to generate a configuration class (`generated-conf/config.php`). Any changes made to the configuration file need to be pushed to this generated configuration file through the `vendor/bin/propel config:convert` command.

Propel configuration file simply describes an associative array of properties, with a well defined hierarchy. See [Configuration reference](/documentation/reference/configuration-file.html) for the complete list.

Propel configuration sub-system is based on [Symfony Config Component](http://symfony.com/doc/current/components/config/index.html).

## Naming conventions ##

The name of the configuration file is `propel`, with one of the supported extensions (see [Supported formats paragraph](#supported-formats)). E.g. `propel.yaml` or `propel.json`.

> **Note** <br /> Throughout this document, when we'll write `.ext` we intend "one of the supported extensions".

Propel looks for its configuration file in the current directory or in a sub-directory named *conf* or *config*.
Alternatively, you can choose a directory of your choice and pass it to the
command line as the `--config-dir` parameter. For instance:

```bash
# Propel looks for its configuration file in the current directory or
# in a subdir named 'conf' or 'config'
vendor/bin/propel sql:build

# Propel looks for its configuration file in /my/awesome/path directory
vendor/bin/propel model:build --config-dir="/my/awesome/path"
```

When you're part of a team, you could want to define a common configuration file and commit it into your VCS. But, of course, there can be some properties you don't want to share, e.g. database passwords.
Propel helps you and looks for a `propel.ext.dist` file too, merging its properties with `propel.ext` ones.
So you can define shared configuration properties in `propel.ext.dist`, committing it in your VCS, and keep `propel.ext` as private.
The properties loaded from `propel.ext` overwrite the ones with the same name, loaded from `propel.ext.dist`.

## Supported formats ##

Propel supports the following formats for its configuration file:

1. **PHP**  - valid php file names are *propel.php* and *propel.inc*
2. **INI**  - valid ini file names are *propel.ini* and *propel.properties*
3. **YAML** - valid yaml file names are *propel.yaml* and *propel.yml*
4. **XML**  - valid xml file name is *propel.xml*
5. **JSON** - valid json file name is *propel.json*


### PHP ###

A php configuration file is expected to be named `propel.php` or `propel.inc` (and [`propel.php.dist` or `propel.inc.dist`](#naming-conventions)).
It must return an associative array of properties.
Here it is an example of php configuration file:

```php
<?php

return [
    'propel' => [
        'database' => [
            'connections' => [
                'mysource' => [
                    'adapter'    => 'mysql',
                    'classname'  => 'Propel\Runtime\Connection\DebugPDO',
                    'dsn'        => 'mysql:host=localhost;dbname=mydb',
                    'user'       => 'root',
                    'password'   => '',
                    'attributes' => []
                ],
                'yoursource' => [
                    'adapter'    => 'mysql',
                    'classname'  => 'Propel\Runtime\Connection\DebugPDO',
                    'dsn'        => 'mysql:host=localhost;dbname=yourdb',
                    'user'       => 'root',
                    'password'   => '',
                    'attributes' => []
                ]
            ]
        ],
        'runtime' => [
            'defaultConnection' => 'mysource',
            'connections' => ['mysource', 'yoursource']
        ],
        'generator' => [
            'defaultConnection' => 'yoursource',
            'connections' => ['yoursource']
        ]
    ]
];
```


### INI ###

An ini configuration file is expected to be named `propel.ini` or `propel.properties`.

Propel utilizes the [`parse_ini_file`](http://www.php.net/parse_ini_file) function to parse ini configuration files; it supports the standard ini format with sections and a *"nested"* format, inspired of [Zend\Config\IniReader](http://framework.zend.com/manual/2.3/en/modules/zend.config.reader.html) class. In this way, you can define an ini hierarchy via a string, where each level is separated by a dot. For instance:

```ini
; Classical way
[first_section]
foo = bar
bar = baz

;Nested way
first_section.foo = bar
first_section.bar = baz
```

You can define also a deeper hierarchy, with more than two levels:

```ini
section_1.sub_section_1.sub_sub_section.foo = bar
section_1.sub_section_2.sub_sub_section.bar = baz
```

and this will get converted into the following array:

```php
<?php

[
    'section_1' => [
        'sub_section_1' => [
            'sub_sub_section' => [
                'foo' => 'bar'
            ]
        ],
        'sub_section_2' => [
            'sub_sub_section' => [
                'bar' => 'baz'
            ]
        ]
    ]
]
```

Of course, you can mix both syntaxes, as you can see in the following *propel.ini* sample:

```ini
[propel]

; Database section
;
; Define 'mysource' connection
database.connections.mysource.adapter    = mysql
database.connections.mysource.classname  = Propel\Runtime\Connection\DebugPDO
database.connections.mysource.dsn        = mysql:host=localhost;dbname=mydb
database.connections.mysource.user       = root
database.connections.mysource.password   =
database.connections.mysource.attributes =
;
; Define 'yoursource' connection
database.connections.yoursource.adapter    = mysql
database.connections.yoursource.classname  = Propel\Runtime\Connection\DebugPDO
database.connections.yoursource.dsn        = mysql:host=localhost;dbname=yourdb
database.connections.yoursource.user       = root
database.connections.yoursource.password   =
database.connections.yoursource.attributes =

;
; Runtime section
;
runtime.defaultConnection = mysource
;
;Available runtime connections
runtime.connections[0]    = mysource
runtime.connections[1]    = yoursource

;
; Generator section
;
generator.defaultConnection = yoursource
generator.connections[0] = yoursource
```


### YAML ###

An yaml configuration file is expected to be named `propel.yaml` or `propel.yml`.

To parse yaml configuration files, Propel uses [Symfony Yaml Component](http://symfony.com/doc/current/components/yaml/index.html).

```yaml
# An example of Propel configuration file in yaml

propel:
  database:
      connections:
          mysource:
              adapter: mysql
              classname: Propel\Runtime\Connection\DebugPDO
              dsn: mysql:host=localhost;dbname=mydb
              user: root
              password:
              attributes:
          yoursource:
              adapter: mysql
              classname: Propel\Runtime\Connection\DebugPDO
              dsn: mysql:host=localhost;dbname=yourdb
              user: root
              password:
              attributes:
  runtime:
      defaultConnection: mysource
      connections:
          - mysource
          - yoursource
  generator:
      defaultConnection: yoursource
      connections:
          - yoursource
```

You can find an example of complete Propel configuration file in yaml, in [this gist](https://gist.github.com/cristianoc72/9060420).


### XML ###

An xml configuration file is expected to be named `propel.xml`.

Here it is the previous example of configuration file, written in xml:

```xml
<?xml version="1.0" encoding="ISO-8859-1" standalone="no"?>
<config>
    <propel>
        <database>
            <connections>
                <connection id="mysource">
                    <adapter>mysql</adapter>
                    <classname>Propel\Runtime\Connection\DebugPDO</classname>
                    <dsn>mysql:host=localhost;dbname=mydb</dsn>
                    <user>root</user>
                    <password>''</password>
                    <attributes></attributes>
                </connection>
                <connection id="yoursource">
                    <adapter>mysql</adapter>
                    <classname>Propel\Runtime\Connection\DebugPDO</classname>
                    <dsn>mysql:host=localhost;dbname=yourdb</dsn>
                    <user>root</user>
                    <password>''</password>
                    <attributes></attributes>
                </connection>
            </connections>
        </database>
        <runtime>
            <defaultConnection>mysource</defaultConnection>
            <connection>mysource</connection>
            <connection>yoursource</connection>
        </runtime>
        <generator>
            <defaultConnection>yoursource</defaultConnection>
            <connection>yoursource</connection>
        </generator>
    </propel>
</config>
```


### JSON ###

A json configuration file is expected to be named `propel.json`.

Propel uses [`json_decode`](http://www.php.net/manual/en/function.json-decode.php) function to parse json configuration files.

And the following is the same example in json format:

```json
{
    "propel": {
        "database": {
            "connections": {
                "mysource": {
                    "adapter": "mysql",
                    "classname": "Propel\\Runtime\\Connection\\DebugPDO",
                    "dsn": "mysql:host=localhost;dbname=mydb",
                    "user": "root",
                    "password": "",
                    "attributes": []
                },
                "yoursource": {
                    "adapter": "mysql",
                    "classname": "Propel\\Runtime\\Connection\\DebugPDO",
                    "dsn": "mysql:host=localhost;dbname=yourdb",
                    "user": "root",
                    "password": "",
                    "attributes": []
                },
            }
        },
        "runtime": {
            "defaultConnection": "mysource",
            "connections": ["mysource", "yoursource"]
        },
        "generator": {
            "defaultConnection": "yoursource",
            "connections": ["yoursource"]
        }
    }
}
```


## Using parameters ##

In Propel configuration file (any format) you can also define some *parameters*.

A parameter is a previously defined property, put between `%` special character. When Propel 's found a parameter, it simply replaces its placeholder with the previously defined value. In example:

```yaml
propel:
  general:
    project: MyProject

  paths:
    projectDir: /home/%project%
```

It becomes:

```yaml
propel:
  general:
    project: MyProject

  paths:
    projectDir: /home/MyProject
```

You can escape the special character `%` by doubling it:

```yaml
propel:
  general:
    project: 100%%
```

`project` property now contains the string `'100%'`.

### Special parameters: environment variables ###

The parameter `env` is used to specify an environment variable. Many hosts give services or credentials via environment variables and you can use them in your configuration file via `env.variable` syntax.
In example, let's suppose to have the following environment variables:

```php
<?php

$_ENV['host']   = '192.168.0.54'; //Database host name
$_ENV['dbName'] = 'myDB'; //Database name
```

In your configuration file you can write:

```yaml
propel:
  database:
      connections:
          default:
              adapter: mysql
              classname: Propel\Runtime\Connection\DebugPDO
              dsn: mysql:host=%env.host%;dbname=%env.dbName%
```

and it becomes:

```yaml
propel:
  database:
      connections:
          default:
              adapter: mysql
              classname: Propel\Runtime\Connection\DebugPDO
              dsn: mysql:host=192.168.0.54;dbname=myDB
```

## Precedence among configuration properties ##

Propel receives configuration properties from four ways: `propel.ext` file, `propel.ext.dist` file, *console* (via command line options), default values (defined in `Propel\Common\Config\PropelConfiguration` class).
Some properties are often overwritten by others with the same name, so we need to define a precedence relation:

1. Default values are overwritten by `propel.ext.dist` values
2. `propel.ext.dist` values are overwritten by `propel.ext` values
3. `propel.ext` values are overwritten by *console* values


## Internals ##

The heart of Propel configuration system is `Propel\Common\Config\ConfigurationManager` class.
This class loads the configuration file, replace parameters and processes it, by using the *TreeBuilder* defined in `Propel\Common\Config\PropelConfiguration` class.

`ConfigurationManager` class exposes three public methods to access the configuration properties:

- `ConfigurationManager::get()` method returns the whole array of properties.
- `ConfigurationManager::getSection($section)` method returns all the properties defined in `$section` section. **Note**: *$section* is a first level only section.
- `ConfigurationManager::getConfigProperty($name)` returns the value of the property named `$name`.

`ConfigurationManager::get()` returns the whole array of configuration properties, and you can access them via the usual array syntax:

```php
<?php

$manager = new Propel\Common\Config\ConfigurationManager();

//Get the array of runtime configured connections
$connections = $manager->get()['runtime']['connections'];
```

`ConfigurationManager::getSection()` is useful when you want to access only to a specific first-level section of properties, i.e. *runtime* or *generator* or *migrations*:

```php
<?php

$manager = new Propel\Common\Config\ConfigurationManager();

$migrationProperties = $manager->getSection('migration');

//getSection is simply a shortcut, so I can write also:
$migrationProperties = $manager->get()['migration'];
```

`ConfigurationManager::getConfigProperty($name)` returns the value of a given property. The hierarchy of this property is expressed by a dotted level syntax. For example:

```php
<?php

$manager = new Propel\Common\Config\ConfigurationManager();

// I want to retrieve 'generator->objectModel->builders->object' property
$tableName = $manager->getConfigProperty('generator.objectModel.builders.object');
```

The main difference between `get` (or `getSection`) and `getConfigProperty` is error handling. If the property, you want to get, doesn't exists the first method returns a PHP *E_NOTICE* `Notice: undefined index....`, while the second returns `null`.
And, of course, `get` is as efficient as an array access, while `getConfigProperty` performs some more checks.

So, we suggest to use `get` method when you are absolutely sure that the property, you want to retrieve, exists and contains a value.

```php
<?php

function getUcaseValue($name)
{
    $manager = new Propel\Common\Config\ConfigurationManager();

    // I don't know if a property $name exists
    return strtoupper($manager->getConfigProperty($name));
}

//Exists! Return '2.0.0-DEV'
echo getUcaseValue('general.version');

//Doesn't exist: return null without any other error message
echo getUcaseValue('mickey.mouse');
```

```php
<?php

function getUcaseGeneralValue($name)
{
    $manager = new Propel\Common\Config\ConfigurationManager();

    // I don't know if a property $name exists, but I use get() method all the same
    return strtoupper($manager->get()['general'][$name]);
}

//Exists! Return '2.0.0-DEV'
echo getUcaseGeneralValue('version');

//Doesn't exist:
//raises an E_NOTICE 'Notice: Undefined index Barbapapa in .... line ....'
echo getUcaseGeneralValue('Barbapapa');
```
