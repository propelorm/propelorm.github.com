---
layout: documentation
title: Using Propel With MSSQL Server
configuration: true
---

# Using Propel With MSSQL Server #

Propel has support for Sybase ASE and MSSQL server 2005 and above. There are several different options available for PDO drivers in both Windows and Linux.

## Windows ##

Windows has 4 different driver implementations that could be used. In order of support: `pdo_sqlsrv`, `pdo_sybase`, `pdo_mssql`, and `pdo_odbc`.

`pdo_dblib` can be built against either FreeTDS (`pdo_sybase`) or MS SQL Server (`pdo_mssql`) dblib implementations. The driver is not a complete PDO driver implementation and lacks support for transactions or driver attributes.

### pdo_sqlsrv ###

This is a driver [released in August 2010 by Microsoft](http://blogs.msdn.com/b/sqlphp/archive/2010/08/04/microsoft-drivers-for-php-for-sql-server-2-0-released.aspx) for interfacing with MS SQL Server. This is a very complete PDO driver implementation and will provide the best results when using Propel on the Windows platform. It does not return blobs as a resource right now but this feature will hopefully be added in a future release. There is also a bug with setting blob values to null that Propel has a workaround for.

Sample dsn's for pdo_sqlsrv:

```xml
<dsn>sqlsrv:server=localhost\SQLEXPRESS;Database=propel</dsn>
<dsn>sqlsrv:server=localhost\SQLEXPRESS,1433;Database=propel</dsn>
<dsn>sqlsrv:server=localhost,1433;Database=propel</dsn>
```

Sample configuration file for pdo_sqlsrv:

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
              adapter: mssql
              classname: Propel\Runtime\Connection\ConnectionWrapper
              dsn: sqlsrv:server=localhost,1433;Database=propel
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
                    'adapter'    => 'mssql',
                    'classname'  => 'Propel\Runtime\Connection\ConnectionWrapper',
                    'dsn'        => 'sqlsrv:server=localhost,1433;Database=propel',
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
                    "adapter": "mssql",
                    "classname": "Propel\Runtime\Connection\ConnectionWrapper",
                    "dsn": "sqlsrv:server=localhost,1433;Database=propel",
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
database.connections.bookstore.adapter    = mssql
database.connections.bookstore.classname  = Propel\Runtime\Connection\ConnectionWrapper
database.connections.bookstore.dsn        = sqlsrv:server=localhost,1433;Database=propel
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
                    <adapter>mssql</adapter>
                    <classname>Propel\Runtime\Connection\ConnectionWrapper</classname>
                    <dsn>sqlsrv:server=localhost,1433;Database=propel</dsn>
                    <user>my_db_user</user>
                    <password>s3cr3t</password>
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

### pdo_sybase ###

When built against FreeTDS dblib it will be called `pdo_sybase`. This requires properly setting up the FreeTDS `freetds.conf` and `locales.conf`. There is a workaround for the lack of transactions support in the `pdo_dblib` driver by using `MssqlDebugPDO` or `MssqlPropelPDO` classes.

c:\freetds.conf

```ini
[global]
  client charset = UTF-8
  tds version = 8.0
  text size = 20971520
```

c:\locales.conf

```ini
[default]
  date format = %Y-%m-%d %H:%M:%S.%z
```

Sample dsn's for pdo_sybase:

```xml
<dsn>sybase:host=localhost\SQLEXPRESS;dbname=propel</dsn>
<dsn>sybase:host=localhost\SQLEXPRESS:1433;dbname=propel</dsn>
<dsn>sybase:host=localhost:1433;dbname=propel</dsn>
```

Sample configuration file for pdo_sybase:

<div class="conftabs">
<ul>
<li><a href="#tabyaml-1">propel.yaml</a></li>
<li><a href="#tabphp-1">propel.php</a></li>
<li><a href="#tabjson-1">propel.json</a></li>
<li><a href="#tabini-1">propel.ini</a></li>
<li><a href="#tabxml-1">propel.xml</a></li>
</ul>
<div id="tabyaml-1">
{% highlight yaml %}
propel:
  database:
      connections:
          bookstore:
              adapter: mssql
              classname: Propel\Runtime\Connection\ConnectionWrapper
              dsn: sybase:host=localhost:1433;Database=propel
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
<div id="tabphp-1">
{% highlight php %}
<?php

return [
    'propel' => [
        'database' => [
            'connections' => [
                'bookstore' => [
                    'adapter'    => 'mssql',
                    'classname'  => 'Propel\Runtime\Connection\ConnectionWrapper',
                    'dsn'        => 'sybase:host=localhost:1433;Database=propel',
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
<div id="tabjson-1">
{% highlight json %}
{
    "propel": {
        "database": {
            "connections": {
                "bookstore": {
                    "adapter": "mssql",
                    "classname": "Propel\Runtime\Connection\ConnectionWrapper",
                    "dsn": "sybase:host=localhost:1433;Database=propel",
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
<div id="tabini-1">
{% highlight ini %}
[propel]
;
; Database section
;
database.connections.bookstore.adapter    = mssql
database.connections.bookstore.classname  = Propel\Runtime\Connection\ConnectionWrapper
database.connections.bookstore.dsn        = sybase:host=localhost:1433;Database=propel
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
<div id="tabxml-1">
{% highlight xml %}
<?xml version="1.0" encoding="ISO-8859-1" standalone="no"?>
<config>
    <propel>
        <database>
            <connections>
                <connection id="bookstore">
                    <adapter>mssql</adapter>
                    <classname>Propel\Runtime\Connection\ConnectionWrapper</classname>
                    <dsn>sybase:host=localhost:1433;Database=propel</dsn>
                    <user>my_db_user</user>
                    <password>s3cr3t</password>
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

### pdo_mssql ###

When built against MS SQL Server dblib the driver will be called `pdo_mssql`. It is not recommended to use the `pdo_mssql` driver because it strips blobs of single quotes when retrieving from the database and will not return blobs or clobs longer that 8192 characters. The dsn differs from `pdo_sybase` in that it uses a comma between the server and port number instead of a colon and mssql instead of sybase for the driver name.

Sample dsn's for `pdo_mssql`:

```xml
<dsn>mssql:host=localhost\SQLEXPRESS;dbname=propel</dsn>
<dsn>mssql:host=localhost\SQLEXPRESS,1433;dbname=propel</dsn>
<dsn>mssql:host=localhost,1433;dbname=propel</dsn>
```

### pdo_odbc ###

Currently `pdo_odbc` cannot be used to access MSSQL with propel because of a [long standing bug](http://connect.microsoft.com/SQLServer/feedback/details/521409/odbc-client-mssql-does-not-work-with-bound-parameters-in-subquery) with the MS SQL Server ODBC Client. Last update on 8/3/2010 was that it would be resolved in a future release of the SQL Server Native Access Client. This bug is related to two php bugs ([Bug #44643](http://bugs.php.net/bug.php?id=44643) and [Bug #36561](http://bugs.php.net/bug.php?id=36561))

## Linux ##

Linux has 2 driver implementations that could be used: `pdo_dblib`, and `pdo_obdc`.

### pdo_dblib ###

`pdo_dblib` is built against the FreeTDS dblib implementation. The driver is not a complete PDO driver implementation and lacks support for transactions or driver attributes. This requires properly setting up the FreeTDS `freetds.conf` and `locales.conf`. There is a workaround for the lack of transactions support in the `pdo_dblib` driver by using `MssqlDebugPDO` or `MssqlPropelPDO` classes.

Redhat: `/etc/freetds.conf`
Ubuntu: `/etc/freetds/freetds.conf`

```ini
[global]
  client charset = UTF-8
  tds version = 8.0
  text size = 20971520
```

Redhat: `/etc/locales.conf`
Ubuntu: `/etc/freetds/locales.conf`

```ini
[default]
  date format = %Y-%m-%d %H:%M:%S.%z
```

Sample dsn's for `pdo_dblib`:

```xml
<dsn>dblib:host=localhost\SQLEXPRESS;dbname=propel</dsn>
<dsn>dblib:host=localhost\SQLEXPRESS:1433;dbname=propel</dsn>
<dsn>dblib:host=localhost:1433;dbname=propel</dsn>
```

Sample configuration file for `pdo_dblib`:

<div class="conftabs">
<ul>
<li><a href="#tabyaml-2">propel.yaml</a></li>
<li><a href="#tabphp-2">propel.php</a></li>
<li><a href="#tabjson-2">propel.json</a></li>
<li><a href="#tabini-2">propel.ini</a></li>
<li><a href="#tabxml-2">propel.xml</a></li>
</ul>
<div id="tabyaml-2">
{% highlight yaml %}
propel:
  database:
      connections:
          bookstore:
              adapter: mssql
              classname: Propel\Runtime\Connection\ConnectionWrapper
              dsn: dblib:host=localhost:1433;dbname=propel
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
<div id="tabphp-2">
{% highlight php %}
<?php

return [
    'propel' => [
        'database' => [
            'connections' => [
                'bookstore' => [
                    'adapter'    => 'mssql',
                    'classname'  => 'Propel\Runtime\Connection\ConnectionWrapper',
                    'dsn'        => 'dblib:host=localhost:1433;dbname=propel',
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
<div id="tabjson-2">
{% highlight json %}
{
    "propel": {
        "database": {
            "connections": {
                "bookstore": {
                    "adapter": "mssql",
                    "classname": "Propel\Runtime\Connection\ConnectionWrapper",
                    "dsn": "dblib:host=localhost:1433;dbname=propel",
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
<div id="tabini-2">
{% highlight ini %}
[propel]
;
; Database section
;
database.connections.bookstore.adapter    = mssql
database.connections.bookstore.classname  = Propel\Runtime\Connection\ConnectionWrapper
database.connections.bookstore.dsn        = dblib:host=localhost:1433;dbname=propel
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
<div id="tabxml-2">
{% highlight xml %}
<?xml version="1.0" encoding="ISO-8859-1" standalone="no"?>
<config>
    <propel>
        <database>
            <connections>
                <connection id="bookstore">
                    <adapter>mssql</adapter>
                    <classname>Propel\Runtime\Connection\ConnectionWrapper</classname>
                    <dsn>dblib:host=localhost:1433;dbname=propel</dsn>
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

### pdo_odbc ###

`pdo_odbc` using UnixODBC and FreeTDS. This should be supported in propel but with ubuntu 10.04 and php 5.2.x any statement binding causes apache to segfault so I have not been able to test it further. If anyone has any additional experience with this please post information to the propel development group. If you would like to experiment there are some instructions you can follow [here](http://kitserve.org.uk/content/accessing-microsoft-sql-server-php-ubuntu-using-pdo-odbc-and-freetds) for getting it setup on ubuntu.
