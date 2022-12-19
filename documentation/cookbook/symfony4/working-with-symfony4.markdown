---
layout: documentation
title: Working With Symfony4
---

# Working With Symfony4 #

If you are interested to work with Propel2 with Symfony, you should consider
using the [PropelBundle](https://github.com/propelorm/PropelBundle).

## Installation ##

The recommended way to install this bundle is to rely on
[Composer](http://getcomposer.org/):
```bash
composer require propel/propel: ~2.0@beta
composer require propel/propel-bundle: ~4.0@dev
```
or, if you prefer to manually edit your `composer.json` file: 

``` json
"require": {
    "propel/propel": "~2.0@beta",
    "propel/propel-bundle": "~4.0@dev"
}
```
Symfony Flex automatically creates a recipe and enables the PropelBundle.

## Configuration ##

Symfony team recommends to use the `.env` file to store passwords and accessing data, so let's open this file and add some environment variables, useful for Propel:

```
###> Propel configuration ###
DB_USER=your_username
DB_PWD=your_password
DB_HOST=your_host_usually_localhost
DB_NAME=your_db_name
###> Propel configuration ###
```

Now we're ready to configure Propel. Let's create the `config/packages/propel.yml` file and add the following minimal configuration:

```yaml
propel:
    database:
        connections:
            default:
                adapter:    mysql
                user:       "%env(DB_USER)%"
                password:   "%env(DB_PWD)%"
                dsn:        "mysql:host=%env(DB_HOST)%;dbname=%env(DB_NAME)%;charset=UTF8"
```

That's all! Propel now works in your Symfony4 application!

## Schema.xml and model directory ##

Propel can auto-discover your schema file: create an `src/propel`directory an put there your `schema.xml` file.

To autoload your model classes without any effort, thanks to the predefined Symfony autoload, generate your model into `App` namespace.

```xml
<database name="default" namespace="App\Model">
.............
.............
</database>
```

## What's next? ##

Read the [Symfony 2 recipe](/documentation/cookbook/symfony2/index.html) for all other informations about commands, forms and security. 