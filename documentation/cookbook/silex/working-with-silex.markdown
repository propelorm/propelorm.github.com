---
layout: documentation
title: Working With Silex
---

# Working With Silex #

The [PropelServiceProvider](https://github.com/propelorm/PropelServiceProvider)
provides integration with [Propel](http://www.propelorm.org).

The *PropelServiceProvider* provides Silex integration with
[Propel](https://github.com/propelorm/Propel2).

Set up
------

If you want to use *PropelServiceProvider*, you will need:

  * Silex 1.x
  * PHP 5.5 or greater
  * Composer

To setup the project in your Silex application, you have to rely on composer ;
just add the following to your `composer.json` file:

``` json
"require": {
    "propel/propel-service-provider": "2.x"
}
```

Then register you model namespace in Composer autoload:

``` json
"autoload": {
     "psr-0": { "Your\\Model\\Namespace": "path/of/your/model" }
}
```

Then install Composer and all dependencies:

    wget http://getcomposer.org/composer.phar

    php composer.phar install


Parameters
----------

* **propel.config_file** (optional): The name of Propel configuration file
with full path. Default is `/full/project/path/generated-conf/config.php`.

> **Tip**<br>It's strongly recommended to use absolute paths for previous option.


Services
--------

No service is provided.

Propel configures and manages itself by using **static** methods and its own
service container, so no service is registered into Application. Actually, the
PropelServiceProvider class initializes Propel in a more "Silex-ian" way.


Registering
-----------

After you've installed *PropelServiceProvider* and its dependencies, you can
register PropelServiceProvider in your application:

``` php
<?php

$app->register(new Propel\Silex\PropelServiceProvider(), array(
    'propel.config_file' => __DIR__.'/path/to/myproject-conf.php'
));
```

Alternatively, if you built your model with default Propel generator options:

``` php
<?php

$app->register(new Propel\Silex\PropelServiceProvider());
```

We can consider "default" Propel generator options:

* Put `schema.xml` files into the main directory project
* Run `vendor/bin/propel model:build` command without specify any options about
directories and namespace package.

Usage
-----

You'll have to build the model by yourself. According to Propel documentation,
you'll need two files:

* `schema.xml` which contains your database schema;

* `runtime-conf.xml` which contains the database configuration.


Use the `vendor/bin/propel` script to create all files (SQL, configuration, Model
classes).

    vendor/bin/propel model:build
    vendor/bin/propel config:convert-xml
    vendor/bin/propel sql:build
    .................

