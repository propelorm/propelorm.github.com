---
layout: documentation
title: Logging And Debugging
configuration: true
---

# Logging And Debugging #

Propel provides tools to monitor and debug your model. Whether you need to check the SQL code of slow queries, or to look for error messages previously thrown, Propel is your best friend for finding and fixing problems.

## Propel Logs ##

Propel uses [Monolog](https://github.com/Seldaek/monolog) to log errors, warnings, and debug information.

You can set the Propel loggers by configuration (in `runtime-conf.xml`), or directly in the service container.

### Setting a Logger Manually ###

By default, Propel uses a single logger called 'defaultLogger'. To enable logging, just set this logger using Propel's `ServiceContainer`:

```php
<?php
use Monolog\Logger;
use Monolog\Handler\StreamHandler;
$logger = new Logger('defaultLogger');
$logger->pushHandler(new StreamHandler('php://stderr'));
Propel::getServiceContainer()->setLogger('defaultLogger', $logger);
```

Propel can use specialized loggers for each connection. For instance, you may want to log the queries for a MySQL database in a file, and the errors of the Propel runtime in another file. To do so, just set another logger using the datasource name, as follows:

```php
<?php
use Monolog\Logger;
use Monolog\Handler\StreamHandler;
$defaultLogger = new Logger('defaultLogger');
$defaultLogger->pushHandler(new StreamHandler('/var/log/propel.log', Logger::WARNING));
Propel::getServiceContainer()->setLogger('defaultLogger', $defaultLogger);
$queryLogger = new Logger('bookstore');
$queryLogger->pushHandler(new StreamHandler('/var/log/propel_bookstore.log'));
Propel::getServiceContainer()->setLogger('bookstore', $queryLogger);
```

>**Tip**If you don't configure a logger for a particular connection, Propel falls back to the default logger.

### Logger Configuration ###

Alternatively, you can configure the logger to use via your configuration file, under the `propel.runtime.log` section. Configuration only allows one handler per logger, and only from a subset of handler types, but this is enough for most use cases.

Here is the way to define the same loggers as in the previous snippet using configuration:

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
  runtime:
      log:
          defaultLogger:
              type: stream
              path: /var/log/propel.log
              level: 300
          bookstore:
              type: stream
              path: /var/log/propel_bookstore.log
{% endhighlight %}
</div>
<div id="tabphp">
{% highlight php %}
<?php

return [
    'propel' => [
        'runtime' => [
            'log' => [
                'defaultLogger' => [
                    'type' => 'stream',
                    'path' => '/var/log/propel.log',
                    'level' => 300
                ],
                'bookstore' => [
                    'type' => 'stream',
                    'path' => '/var/log/propel_bookstore.log',
                ]
            ]
        ]
    ]
];
{% endhighlight %}
</div>
<div id="tabjson">
{% highlight json %}
{
    "propel": {
        "runtime": {
            "log": {
                "defaultLogger": {
                    "type": "stream",
                    "path": "/var/log/propel.log",
                    "level": 300
                },
                "bookstore": {
                    "type": "stream",
                    "path": "/var/log/propel_bookstore.log"
                }
            }
        }
    }
}
{% endhighlight %}
</div>
<div id="tabini">
{% highlight ini %}
[propel]

runtime.log.defaultLogger.type = "stream"
runtime.log.defaultLogger.path = "/var/log/propel.log"
runtime.log.defaultLogger.level = 300

runtime.log.bookstore.type = "stream"
runtime.log.bookstore.path = "/var/log/propel_bookstore.log"

{% endhighlight %}
</div>
<div id="tabxml">
{% highlight xml %}
<?xml version="1.0" encoding="ISO-8859-1" standalone="no"?>
<config>
    <propel>
        <runtime>
            <log name="defaultLogger">
                <type>stream</type>
                <path>/var/log/propel.log</path>
                <level>300</level>
            </log>
            <log name="bookstore">
                <type>stream</type>
                <path>/var/log/propel_bookstore.log</path>
            </log>
        </runtime>
    </propel>
</config>
{% endhighlight %}
</div>
</div>

The meaning of each of the `<log>` nested elements may vary, depending on which log handler you are using. Accepted handler types in configuration are `stream`, `rotating_file`, and `syslog`. Refer to the [Monolog](https://github.com/Seldaek/monolog) documentation for more details on log handlers configuration and options.

### Logging Messages ###

The service container offers a `getLogger()` method which, when called without parameter, returns the 'defaultLogger'. You can use then use the logger to add messages in the usual Monolog way.

```php
<?php
$logger = Propel::getServiceContainer()->getLogger();
$logger->addInfo('This is a message');
```

Alternatively, use the static `Propel::log()` method, passing a log level as parameter:

```php
<?php
$myObj = new MyObj();
$myObj->setName('foo');
Propel::log('uh-oh, something went wrong with ' . $myObj->getName(), Logger::ERROR);
```

You can also log your own messages from the generated model objects by using their `log()` method, implemented in `ActiveRecordInterface`:

```php
<?php
$myObj = new MyObj();
$myObj->log('uh-oh, something went wrong', Logger::ERROR);
```

The log messages will show up in the log handler defined in `runtime-conf.xml` (`propel.log` file by default) as follows:

```
[2011-12-12 00:29:31] defaultLogger.ERROR: uh-oh, something went wrong with foo
[2011-12-12 00:29:31] defaultLogger.ERROR: MyObj: uh-oh, something went wrong
```

>**Tip**All serious errors coming from the Propel core do not only issue a log message, they are also thrown as `PropelException`.

## Debugging Database Activity ##

By default, Propel uses the `Propel\Runtime\Connection\ConnectionWrapper` class for database connections. This class, which wraps around PHP's `PDO`, offers a debug mode to keep track of all the database activity, including all the executed queries.

### Enabling The Debug Mode ###

The debug mode is disabled by default, but you can enable it at runtime as follows:

```php
<?php
$con = Propel::getWriteConnection(MyObjTableMap::DATABASE_NAME);
$con->useDebug(true);
```

You can also disable the debug mode at runtime, by calling `PropelPDO::useDebug(false)`. Using this method, you can choose to enable the debug mode for only one particular query, or for all queries.

Alternatively, you can ask Propel to always enable the debug mode for a particular connection by using the `Propel\Runtime\Connection\DebugPDO` class instead of the default `ConnectionWrapper` class. This is accomplished in the configuration file, in the `classname` node of a given datasource connection (see the [configuration reference](/documentation/reference/configuration-file.html) for more details).

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
              adapter: mysql
              classname: Propel\Runtime\Connection\DebugPDO
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
                    'adapter'    => 'mysql',
                    'classname'  => 'Propel\Runtime\Connection\DebugPDO'
                ]
            ]
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
                    "adapter": "mysql",
                    "classname": "Propel\Runtime\Connection\DebugPDO",
                }
            }
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
database.connections.bookstore.adapter    = mysql
database.connections.bookstore.classname  = Propel\Runtime\Connection\DebugPDO
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
                    <adapter>mysql</adapter>
                    <classname>Propel\Runtime\Connection\DebugPDO</classname>
                </connection>
            </connections>
        </database>
    </propel>
</config>
{% endhighlight %}
</div>
</div>

>**Tip**You can use your own connection class there, but make sure that it implements `Propel\Runtime\Connection\ConnectionInterface`.

### Counting Queries ###

In debug mode, the connection class keeps track of the number of queries that are executed. Use `ConnectionWrapper::getQueryCount()` to retrieve this number:

```php
<?php
$con = Propel::getWriteConnection(MyObjTableMap::DATABASE_NAME);
$myObjs = MyObjQuery::create()->doSelect(new Criteria(), $con);
echo $con->getQueryCount();  // 1
```

>**Tip**You cannot use persistent connections if you want the query count to work. Actually, the debug mode in general requires that you don't use persistent connections in order for it to correctly log bound values and count executed statements.

### Retrieving The Latest Executed Query ###

For debugging purposes, you may need the SQL code of the latest executed query. It is available at runtime in debug mode using `ConnectionWrapper::getLastExecutedQuery()`, as follows:

```php
<?php
$con = Propel::getWriteConnection(MyObjTableMap::DATABASE_NAME);
$myObjs = MyObjTableMap::create()->doSelect(new Criteria(), $con);
echo $con->getLastExecutedQuery(); // 'SELECT * FROM my_obj';
```

>**Tip**You can also get a decent SQL representation of the criteria being used in a SELECT query by using the `Criteria->toString()` method.

Propel also keeps track of the queries executed directly on the connection object, and displays the bound values correctly.

```php
<?php
$con = Propel::getWriteConnection(MyObjTableMap::DATABASE_NAME);
$stmt = $con->prepare('SELECT * FROM my_obj WHERE name = :p1');
$stmt->bindValue(':p1', 'foo');
$stmt->execute();
echo $con->getLastExecutedQuery(); // 'SELECT * FROM my_obj where name = "foo"';
```

>**Tip**The debug mode is intended for development use only. Do not use it in production environment, it logs too much information for a production server, and adds a small overhead to the database queries.

## Full Query Logging ##

If you use both the debug mode and a logger, then Propel logs automatically all executed queries in the provided log handler:

```
Oct 04 00:00:18 propel-bookstore [debug] INSERT INTO publisher (`ID`,`NAME`) VALUES (NULL,'William Morrow')
Oct 04 00:00:18 propel-bookstore [debug] INSERT INTO author (`ID`,`FIRST_NAME`,`LAST_NAME`) VALUES (NULL,'J.K.','Rowling')
Oct 04 00:00:18 propel-bookstore [debug] INSERT INTO book (`ID`,`TITLE`,`ISBN`,`PRICE`,`PUBLISHER_ID`,`AUTHOR_ID`) VALUES (NULL,'Harry Potter and the Order of the Phoenix','043935806X',10.99,53,58)
Oct 04 00:00:18 propel-bookstore [debug] INSERT INTO review (`ID`,`REVIEWED_BY`,`REVIEW_DATE`,`RECOMMENDED`,`BOOK_ID`) VALUES (NULL,'Washington Post','2009-10-04',1,52)
...
Oct 04 00:00:18 propel-bookstore [debug] SELECT bookstore_employee_account.EMPLOYEE_ID, bookstore_employee_account.LOGIN FROM `bookstore_employee_account` WHERE bookstore_employee_account.EMPLOYEE_ID=25
```

By default, Propel logs all SQL queries, together with the date of the query and the name of the connection.

### Using a Custom Logger per Connection ###

To log SQL queries for a connection, Propel first looks for a logger named after the connection itself, and falls back to the default logger if no custom logger is defined for the connection.

Using the following config, Propel will log SQL queries from the `bookstore` datasource into a `propel_bookstore.log` file, and the SQL queries for all other datasources into a `propel.log` file.

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
  runtime:
      log:
          defaultLogger:
              type: stream
              path: /var/log/propel.log
              level: 300
          bookstore:
              type: stream
              path: /var/log/propel_bookstore.log
{% endhighlight %}
</div>
<div id="tabphp-2">
{% highlight php %}
<?php

return [
    'propel' => [
        'runtime' => [
            'log' => [
                'defaultLogger' => [
                    'type' => 'stream',
                    'path' => '/var/log/propel.log',
                    'level' => 300
                ],
                'bookstore' => [
                    'type' => 'stream',
                    'path' => '/var/log/propel_bookstore.log',
                ]
            ]
        ]
    ]
];
{% endhighlight %}
</div>
<div id="tabjson-2">
{% highlight json %}
{
    "propel": {
        "runtime": {
            "log": {
                "defaultLogger": {
                    "type": "stream",
                    "path": "/var/log/propel.log",
                    "level": 300
                },
                "bookstore": {
                    "type": "stream",
                    "path": "/var/log/propel_bookstore.log"
                }
            }
        }
    }
}
{% endhighlight %}
</div>
<div id="tabini-2">
{% highlight ini %}
[propel]

runtime.log.defaultLogger.type = "stream"
runtime.log.defaultLogger.path = "/var/log/propel.log"
runtime.log.defaultLogger.level = 300

runtime.log.bookstore.type = "stream"
runtime.log.bookstore.path = "/var/log/propel_bookstore.log"

{% endhighlight %}
</div>
<div id="tabxml-2">
{% highlight xml %}
<?xml version="1.0" encoding="ISO-8859-1" standalone="no"?>
<config>
    <propel>
        <runtime>
            <log name="defaultLogger">
                <type>stream</type>
                <path>/var/log/propel.log</path>
                <level>300</level>
            </log>
            <log name="bookstore">
                <type>stream</type>
                <path>/var/log/propel_bookstore.log</path>
            </log>
        </runtime>
    </propel>
</config>
{% endhighlight %}
</div>
</div>

This allows you to define a different logger per connection, for instance to have different log files for each database, or to log only the queries from a MySQL database to a file while the ones from an Oracle database go into Syslog.

### Logging More Events ###

By default, the full query logger logs only executed SQL queries. But the `ConnectionWrapper` class can write into the log when most of its methods are called. To enable more methods, just set the list of logging methods to use by calling `ConnectionWrapper::setLogMethods()`, as follows:

```php
<?php
$con = Propel::getWriteConnection(MyObjTableMap::DATABASE_NAME);
$con->setLogMethods(array(
  'exec',
  'query',
  'execute', // these first three are the default
  'beginTransaction',
  'commit',
  'rollBack',
  'bindValue'
));
```

Note that this list takes into account the methods from both `ConnectionWrapper` and `StatementWrapper`.

### Adding Profiler Information ###

In addition to the executed queries, you can ask Propel to log the execution time for each query, the memory consumption, and more. To enable profiling, change the connection class name to `Propel\Runtime\Connection\ProfilerConnectionWrapper` in the configuration file, as follows:

<div class="conftabs">
<ul>
<li><a href="#tabyaml-3">propel.yaml</a></li>
<li><a href="#tabphp-3">propel.php</a></li>
<li><a href="#tabjson-3">propel.json</a></li>
<li><a href="#tabini-3">propel.ini</a></li>
<li><a href="#tabxml-3">propel.xml</a></li>
</ul>
<div id="tabyaml-3">
{% highlight yaml %}
propel:
  database:
      connections:
          bookstore:
              adapter: mysql
              classname: Propel\Runtime\Connection\ProfilerConnectionWrapper
{% endhighlight %}
</div>
<div id="tabphp-3">
{% highlight php %}
<?php

return [
    'propel' => [
        'database' => [
            'connections' => [
                'bookstore' => [
                    'adapter'    => 'mysql',
                    'classname'  => 'Propel\Runtime\Connection\ProfilerConnectionWrapper'
                ]
            ]
        ]
    ]
];
{% endhighlight %}
</div>
<div id="tabjson-3">
{% highlight json %}
{
    "propel": {
        "database": {
            "connections": {
                "bookstore": {
                    "adapter": "mysql",
                    "classname": "Propel\Runtime\Connection\ProfilerConnectionWrapper",
                }
            }
        }
    }
}
{% endhighlight %}
</div>
<div id="tabini-3">
{% highlight ini %}
[propel]
;
; Database section
;
database.connections.bookstore.adapter    = mysql
database.connections.bookstore.classname  = Propel\Runtime\Connection\ProfilerConnectionWrapper
{% endhighlight %}
</div>
<div id="tabxml-3">
{% highlight xml %}
<?xml version="1.0" encoding="ISO-8859-1" standalone="no"?>
<config>
    <propel>
        <database>
            <connections>
                <connection id="bookstore">
                    <adapter>mysql</adapter>
                    <classname>Propel\Runtime\Connection\ProfilerConnectionWrapper</classname>
                </connection>
            </connections>
        </database>
    </propel>
</config>
{% endhighlight %}
</div>
</div>

The logged queries now contain profiling information for each query:

```
Feb 23 16:41:04 Propel [debug] time: 0.000 sec | mem: 1.4 MB | SET NAMES 'utf8'
Feb 23 16:41:04 Propel [debug] time: 0.002 sec | mem: 1.6 MB | SELECT COUNT(tags.NAME) FROM tags WHERE tags.IMAGEID = 12
Feb 23 16:41:04 Propel [debug] time: 0.012 sec | mem: 2.4 MB | SELECT tags.NAME, image.FILENAME FROM tags LEFT JOIN image ON tags.IMAGEID = image.ID WHERE image.ID = 12
```

### Tweaking the Profiling Information Using Configuration ###

You can tweak the type and formatting of the profiler information prefix using the `profiler` section in the configuration file:

<div class="conftabs">
<ul>
<li><a href="#tabyaml-4">propel.yaml</a></li>
<li><a href="#tabphp-4">propel.php</a></li>
<li><a href="#tabjson-4">propel.json</a></li>
<li><a href="#tabini-4">propel.ini</a></li>
<li><a href="#tabxml-4">propel.xml</a></li>
</ul>
<div id="tabyaml-4">
{% highlight yaml %}
propel:
  runtime:
    profiler:
      classname: \Propel\Runtime\Util\Profiler
      slowTreshold: 0.1
      details:
        time:
          precision: 3
          pad: 8
        mem:
          precision: 3
          pad: 8
{% endhighlight %}
</div>
<div id="tabphp-4">
{% highlight php %}
<?php

return [
    'propel' => [
        'runtime' => [
            'profiler' => [
                'classname' => '\Propel\Runtime\Util\Profiler',
                'slowTreshold' => 0.1,
                'details' => [
                    'time'    => [
                        'precision' => 3,
                        'pad' => 8
                    ],
                    'mem'    => [
                        'precision' => 3,
                        'pad' => 8
                    ]
                ]
            ]
        ]
    ]
];
{% endhighlight %}
</div>
<div id="tabjson-4">
{% highlight json %}
{
    "propel": {
        "runtime": {
            "profiler": {
                "classname": "\Propel\Runtime\Util\Profiler",
                "slowTreshold": 0.1,
                "details": {
                    "time": {
                        "precision": 3,
                        "pad": 8
                    },
                    "mem": {
                        "precision": 3,
                        "pad": 8
                    }
                }
            }
        }
    }
}
{% endhighlight %}
</div>
<div id="tabini-4">
{% highlight ini %}
[propel]
runtime.profiler.classname = "\Propel\Runtime\Util\Profiler"
runtime.profiler.slowTreshold = 0.1
runtime.profiler.details.time.precision = 3
runtime.profiler.details.time.pad = 8
runtime.profiler.details.mem.precision = 3
runtime.profiler.details.mem.pad = 8
{% endhighlight %}
</div>
<div id="tabxml-4">
{% highlight xml %}
<?xml version="1.0" encoding="ISO-8859-1" standalone="no"?>
<config>
    <propel>
        <runtime>
            <profiler>
                <classname>"\Propel\Runtime\Util\Profiler"</classname>
                <slowTreshold>0.1</slowTreshold>
                <details>
                    <time precision="3" pad="8" />
                    <mem precision="3" pad="8" />
                </details>
            </profiler>
        </runtime>
    </propel>
</config>
{% endhighlight %}
</div>
</div>

The `slowTreshold` parameter specifies when the profiler considers a query slow. By default, its value is of 0.1s, or 100ms.

>**Tip**<br/>You can choose to only log slow queries when using the `ProfilerConnectionWrapper` connection class. Just add a `isSlowOnly` attribute to the connection in `propel.database` section of your configuration file, as follows:

<div class="conftabs">
<ul>
<li><a href="#tabyaml-5">propel.yaml</a></li>
<li><a href="#tabphp-5">propel.php</a></li>
<li><a href="#tabjson-5">propel.json</a></li>
<li><a href="#tabini-5">propel.ini</a></li>
<li><a href="#tabxml-5">propel.xml</a></li>
</ul>
<div id="tabyaml-5">
{% highlight yaml %}
propel:
  database:
      connections:
          bookstore:
              adapter: mysql
              classname: Propel\Runtime\Connection\ProfilerConnectionWrapper
              attributes:
                isSlowOnly: true
{% endhighlight %}
</div>
<div id="tabphp-5">
{% highlight php %}
<?php

return [
    'propel' => [
        'database' => [
            'connections' => [
                'bookstore' => [
                    'adapter'    => 'mysql',
                    'classname'  => 'Propel\Runtime\Connection\ProfilerConnectionWrapper',
                    'attributes' => [
                        'isSlowOnly' => true
                    ]
                ]
            ]
        ]
    ]
];
{% endhighlight %}
</div>
<div id="tabjson-5">
{% highlight json %}
{
    "propel": {
        "database": {
            "connections": {
                "bookstore": {
                    "adapter": "mysql",
                    "classname": "Propel\Runtime\Connection\ProfilerConnectionWrapper",
                    "attributes": {
                        "isSlowOnly": true
                    }
                }
            }
        }
    }
}
{% endhighlight %}
</div>
<div id="tabini-5">
{% highlight ini %}
[propel]
;
; Database section
;
database.connections.bookstore.adapter    = mysql
database.connections.bookstore.classname  = Propel\Runtime\Connection\ProfilerConnectionWrapper
database.connection.bookstore.attributes.isSlowOnly = true
{% endhighlight %}
</div>
<div id="tabxml-5">
{% highlight xml %}
<?xml version="1.0" encoding="ISO-8859-1" standalone="no"?>
<config>
    <propel>
        <database>
            <connections>
                <connection id="bookstore">
                    <adapter>mysql</adapter>
                    <classname>Propel\Runtime\Connection\ProfilerConnectionWrapper</classname>
                    <attributes>
                        <option id="isSlowOnly">true</option>
                    </attributes>
                </connection>
            </connections>
        </database>
    </propel>
</config>
{% endhighlight %}
</div>
</div>

The supported `details` include `<time>` (the time used by the RDBMS to execute the SQL request), `<mem>` (the memory used so far by the PHP script), `<memDelta>` (the memory used specifically for this query), and `<memPeak>` (the peak memory used by the PHP script). For each detail, you can modify the formatting by setting any of the `name`, `precision`, and `pad` attributes.

Each detail is separated from its name using the `innerGlue` string, and the details are separated from each other using the `outerGlue` string. Modify the corresponding attributes at will.

### Tweaking the Profiling Information At Runtime ###

All the settings described above act on a `Propel\Runtime\Util\Profiler` instance. Instead of using configuration, you can modify the profiler service settings at runtime using the `StandardServiceContainer` instance:

```php
<?php
$serviceContainer = Propel::getServiceContainer();
$serviceContainer->setProfilerClass('\Runtime\Runtime\Util\Profiler');
$serviceContainer->setProfilerConfiguration(array(
   'slowTreshold' => 0.1,
   'details' => array(
       'time' => array(
           'name' => 'Time',
           'precision' => '3',
           'pad' => '8',
        ),
        'mem' => array(
            'name' => 'Memory',
            'precision' => '3',
            'pad' => '8',
        )
   ),
   'outerGlue' => ': ',
   'innerGlue' => ' | '
));
```

---
<span class="next">[Next: Inheritance &rarr;](08-inheritance.html)</span>
