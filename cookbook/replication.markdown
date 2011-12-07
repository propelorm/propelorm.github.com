---
layout: documentation
title: Replication
---

# Replication #

Propel can be used in a master-slave replication environment. These environments are set up to improve the performance of web applications by dispatching the database-load to multiple database-servers. While a single master database is responsible for all write-queries, multiple slave-databases handle the read-queries. The slaves are synchronised with the master by a fast binary log (depending on the database).

## Configuring Propel for Replication ##

* Set up a replication environment (see the Databases section below)
* Use the latest Propel-Version
* add a slaves-section to your `runtime-conf.xml` file
* verify the correct setup by checking the masters log file (should not contain "select ..." statements)

You can configure Propel to support replication by adding a `<slaves>` element with nested `<connection>` element(s) to your `runtime-conf.xml`.

The `<slaves>` section is at the same level as the master `<connection>` and contains multiple nested `<connection>` elements with the same information as the top-level (master) `<connection>`. It is recommended that they are numbered. The follwing example shows a slaves section with a several slave connections configured where "localhost" is the master and "slave-server1" and "slave-server2" are the slave-database connections.

{% highlight xml %}
<?xml version="1.0"?>
<config>
  <log>
    <ident>propel-bookstore</ident>
    <name>console</name>
    <level>7</level>
  </log>
  <propel>
    <datasources default="bookstore">
      <datasource id="bookstore">
        <adapter>sqlite</adapter>
        <connection>
          <dsn>mysql:host=localhost;dbname=bookstore</dsn>
          <user>testuser</user>
          <password>password</password>
        </connection>
        <slaves>
         <connection>
          <dsn>mysql:host=slave-server1; dbname=bookstore</dsn>
          <user>testuser</user>
          <password>password</password>
         </connection>
         <connection>
          <dsn>mysql:host=slave-server2; dbname=bookstore</dsn>
          <user>testuser</user>
          <password>password</password>
         </connection>
        </slaves>
      </datasource>
    </datasources>
  </propel>
</config>
{% endhighlight %}

## Implementation ##

The replication functionality is implemented in the Propel connection configuration and initialization code and in the generated Peer and Object classes.

### Propel::getConnection() ###

When requesting a connection from Propel (_Propel::getConnection()_), you can either specify that you want a READ connection (slave) or WRITE connection (master).  Methods that are designed to perform READ operations, like the `doSelect*()` methods of your generated Peer classes, will always request a READ connection like so:

{% highlight php %}
<?php

$con = Propel::getConnection(MyPeer::DATABASE_NAME, Propel::CONNECTION_READ);
{% endhighlight %}

Other methods that are designed to perform write operations will explicitly request a Propel::CONNECTION_WRITE connection.  The WRITE connections are also the default, however, so applications that make a call to _Propel::getConnection()_ without specifying a connection mode will always get a master connection.

If you do have configured slave connections, Propel will choose a single random slave to use per request for any connections where the mode is Propel::CONNECTION_READ.

Both READ (slave) and WRITE (master) connections are only configured on demand.  If all of your SQL statements are SELECT queries, Propel will never create a connection to the master database (unless, of course, you have configured Propel to always use the master connection -- see below).

_Important:_ if you are using Propel to execute custom SQL queries in your application (and you want to make sure that Propel respects your replication setup), you will need to explicitly get the correct connection.  For example:

{% highlight php %}
<?php

$con = Propel::getConnection(MyPeer::DATABASE_NAME, Propel::CONNECTION_READ);
$stmt = $con->query('SELECT * FROM my');
/* ... */
{% endhighlight %}

### Propel::setForceMasterConnection() ###

You can force Propel to always return a WRITE (master) connection from _Propel::getConnection()_ by calling _Propel::setForceMasterConnection(true);_.  This can be useful if you must be sure that you are getting the most up-to-date data (i.e. if there is some latency possible between master and slaves).

## Databases ##

### MySql ###

[http://dev.mysql.com/doc/refman/5.0/en/replication-howto.html](http://dev.mysql.com/doc/refman/5.0/en/replication-howto.html)

## References ##

* Henderson Carl (2006): Building Scalable Web Sites. The Flickr Way. O'Reilly. ISBN-596-10235-6.
