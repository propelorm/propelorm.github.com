---
layout: documentation
title: Configuration Properties Reference
---

# Configuration Properties Reference #

Here is a list of properties that can be set to affect Propel behavior: how it builds your model, how it performs migrations or runs.

## Where and how to Specify Properties ##

You can specify configuration properties in a file, in the main directory of your project, named `propel.ext`, where `.ext` means: one of the supported extensions. See [Configuration chapter](/documentation/10-configuration.html) for further information.

First, some conventions:

* Text surrounded by a  `/` is text that you would provide and is not defined in the language. (i.e. a table name is a good example of this.)
* Items where you have an alternative choice have a `|` character between them (i.e. true|false)
* Alternative choices may be delimited by `{` and `}` to indicate that this is the default option, if not overridden elsewhere.


## Property List ##

The following configuration properties are written in yaml but you can use your favorite format among the supported ones.

A valid configuration file must start with a `propel` key, which is the parent of all other properties. If you omit this, Propel can't process and validate the configuration tree.

```yaml
propel:
```

### General Settings ###

```yaml
  general:
      # The name of your project.
      project: /Your-Project-Name/
      version: /propel version/
```

### Exclude tables ###

```yaml
  exclude_tables:
      # you can use wildcard as you wish
      - /name of table to ignore/|string
      - /name of other table to ignore/|string
```

There are two Propel actions which reverse the database structure: diff and
reverse schema creation. For better interoperability Propel provides a feature
to ignore tables by whole name or pattern. For example, if you have Wordpress
installed in your database and you don't want to maintain or work with
its tables — just add `wp_*`. You can skip some tables by pattern and use
wildcard at any position, for example: `module_*_cache*` is also valid.

### Directories and Filenames ###

```yaml
  ### Directories and Filenames ###
  paths:
      # Directory where the project files (`schema.xml`, etc.) are located.
      # Default value is current path #
      projectDir:  {current_path}|string

      # The directory where Propel expects to find your `schema.xml` file.
      schemaDir: {current_path}|string

      # The directory where Propel should output classes, sql, config, etc.
      # Default value is current path #
      outputDir: {current_path}|string

      # The directory where Propel should output generated object model classes.
      phpDir: {current-path/generated-classes}|string

      # The directory where Propel should output the compiled runtime configuration.
      phpConfDir: {current-path/generated-conf}|string

      # The directory where Propel should output the generated migrations.
      migrationDir: {current-path/generated-migrations}|string

      # The directory where Propel should output the generated DDL (or data insert statements, etc.)
      sqlDir: {current-path/generated-sql}|string

      # Directory in which your composer.json resides
      composerDir: {empty}|string
```

### Database settings ###

In this section you can define all configured connections and other databases properties. Propel expects you to define one connection at least.

>**Tip**<br /> If you come from Propel 1.x: this section replaces the old `runtime-conf.xml` file. This section is also used by `config:convert` command, to write down the bootstrap Propel file.

```yaml
  ## All Database settings ##
  #
  database:
      # All database sources
      connections:
          /the name of the connection/:
```
#### Adapter ####

The adapter to use for Propel. Currently supported adapters: sqlite, pgsql, mysql, oracle, mssql. Note that it is possible that your adapter could be different from your connection driver (e.g. if using ODBC to connect to MSSQL database, you would use an ODBC PDO driver, but MSSQL Propel adapter).

```yaml
              adapter: {empty}|string
```

#### Classname ####

A `ConnectionInterface` class (ConnectionWrapper or DebugPDO etc.) that you would like to use for the connection.

This can be used to specify the alternative **DebugPDO** class bundled with Propel, or your own subclass. Your class must inherit from ConnectionWrapper, because Propel requires the ability to nest transactions (without having exceptions being thrown by PDO).

```yaml
              # Connection class. One of the Propel\Runtime\Connection classes
              classname: {Propel\Runtime\Connection\ConnectionWrapper}|string
```

#### dsn ####

The PDO DSN that Propel will use to connect to the database.

See the PHP documentation for specific format:

 * [MySQL DSN](http://www.php.net/manual/en/ref.pdo-mysql.connection.php)
 * [PostgreSQL DSN](http://www.php.net/manual/en/ref.pdo-pgsql.connection.php)
 * [SQLite DSN](http://www.php.net/manual/en/ref.pdo-sqlite.connection.php)
 * [Oracle DSN](http://www.php.net/manual/en/ref.pdo-oci.connection.php)
 * [MSSQL DSN](http://www.php.net/manual/en/ref.pdo-dblib.connection.php)

Note that some databases (e.g. PostgreSQL) specify username and password as part of the DSN while the others specify user and password separately.

```yaml
              # The PDO dsn
              dsn: /your dsn/
```

#### Username and Password ####

Specifies credentials for databases that specify username and password separately (e.g. MySQL, Oracle).

```yaml
              user: /Your username/
              password: /Your password/
```

#### options ####

This property contains any options which *must* be specified when the ConnectionInterface connection is created. For example, the ATTR_PERSISTENT option must be specified at the object creation time.

See the [PDO documentation](http://www.php.net/pdo) for more details.

```yaml
              # Driver options. See http://www.php.net/manual/en/pdo.construct.php
              # options must be passed to the contructor of the connection object
              options: {empty}|array
```

#### attributes ####

`attributes` are similar to `options`; the difference is that options specified in `attributes` are set after the ConnectionInterface object has been created. These are set using the [ConnectionInterface::setAttribute()](http://us.php.net/PDO-setAttribute) method.

In addition to the standard attributes that can be set on the ConnectionInterface object, there are also the following Propel-specific attributes that change the behavior of the Propel connection:

|Attribute constant                     | Valid Values (Default)    | Description |
|---------------------------------------|---------------------------|-----------------------------------------------------------------
| PropelPDO::PROPEL_ATTR_CACHE_PREPARES | true/false (false)        | Whether to have the PropelPDO connection cache the PDOStatement prepared statements.  This will improve performance if you are executing the same query multiple times by your script (within a single request / script run).

> Note that attributes in the configuration file can be specified with or without the PDO:: (or PropelPDO::) constant prefix.

```yaml
              # See http://www.php.net/manual/en/pdo.getattribute.php
              # Attributes are set via `setAttribute()` method, after the connection object is created
              attributes: {empty}|array
```

#### settings ####

Settings are Propel-specific options used to further configure the connection -- or perform other user-defined initialization tasks.

Currently supported settings are:

 * charset
 * queries

```yaml
              #Propel specific settings
              settings: {empty}|array
```

##### Charset #####

Specifies the character set to use. Currently you must specify the charset in the way that is understood by your RDBMS. Also note that not all database systems support specifying charset (e.g. SQLite must be compiled with specific charset support). Specifying this option will likely result in an exception if your database doesn't support the specified charset.

```yaml
                charset: {utf8}|string
```

#### Queries ####

Specifies any SQL statements to run when the database connection is initialized. This can be used for any environment setup or db initialization you would like to perform. These statements will be executed each time Propel get initialized (e.g. each time a PHP script is loaded).

```yaml
                #Array of queries to run when the database connection is initialized
                queries: {empty}|array
```

#### Master/Slaves ####

The `slaves` properties groups lists slave `connection` elements which provide support for configuring slave db servers -- when using Propel in a master-slave replication environment. See the [Master-Slave documentation](/documentation/cookbook/replication.html) for more information.


```yaml
              slaves:
                  - dsn: mysql:host=slave-host-1;dbname=bookstore
                  - ......
```

#### Model paths ####

`model_paths` specifies the folders where the models are located. Removing unneeded
folders can improve the performance of fixture creation. The default folders are
`src` and `vendor`.

```yaml
              model_paths:
                  - src
                  - vendor
                  - ......
```

### Specific adapter settings ###

In this section you can define some settings for specific DBMS.

```yaml
      ## Specific adapter settings
      adapters:
          ## Mysql ##
          mysql:
```

#### Default table type. ####

You can override this setting if you wish to default to another engine for all tables (for instance InnoDB, or HEAP). This setting can also be overridden on a per-table basis using the `<vendor>` element in the schema (see Schema AddingVendorInfo).

```yaml
              # Default table type
              tableType: {InnoDB}|MyIsam
```
```yaml

              # Keyword used to specify the table engine in the CREATE SQL statement.
              # Defaults to 'ENGINE', users of MYSQL < 5 should use 'TYPE' instead.
              tableEngineKeyword: {ENGINE}|TYPE

          ## Sqlite ##
          sqlite:
              foreigKey: {empty}|string
              tableAlteringWorkaround: {empty}|boolean

          ## Oracle ##
          oracle:
              autoincrementSequencePattern: ${table}_SEQ
```

#### UUID column type (MySQL/MariaDB only) ####

Allows to enable the native UUID column type available in MariaDB (see [UUID_BINARY columns](/documentation/reference/uuid-binary-columns.html)):
```yaml
              uuidColumnType: native|off
```

### Migrations settings ###

```yaml
  ## Migration settings ##
  migrations:
      # Whether to specify PHP names that are the same as the column names.
      samePhpName: {false}|true

      # Whether to add the vendor info. It does provide additional information (such as full-text indexes) which can
      # affect the generation of the DDL from the schema.
      addVendorInfo: {false}|true

      # The name of migrations table
      tableName: {propel_migration}|string

      # The name of the parser class
      # If you leave this property blank, Propel looks for an appropriate parserClass, based on platform: e.g.
      # if the platform is `MysqlPlatform` then parser is `\Propel\Generator\Reverse\MysqlSchemaParser`
      parserClass: {empty}|string
```

### Reverse engineering ###

This section configures the reverse engineering, to create an xml schema from an existing database.

```yaml
  ## Reverse settings
  reverse:
    # The connection to use to reverse the database
    connection: /One of the connections defined in database section/

    # Reverse parser class can be different from migration one
    # If you leave this property blank, Propel looks for an appropriate parser class, based on platform: i.e.
    # if the platform is `MysqlPlatform` then parser is `\Propel\Generator\Reverse\MysqlSchemaParser`
    parserClass: {empty}|string
```

### Runtime settings ###

```yaml
  ## Runtime settings ##
  runtime:
      defaultConnection: /One of the connections defined in database section/
      # Datasources as defined in database.connections
      # This section affects config:convert command
      connections:
         - default

      ## Log and loggers definitions ##
      # For `type` and `level` options see Monolog documentation https://github.com/Seldaek/monolog
      log:
        defaultLogger:
          type: {empty}|string
          path: {empty}|string
          level: {empty}|integer

      ## Profiler configuration ##
      # To enable the profiler for a connection, set the `classname` option to \Propel\Runtime\Connection\ProfilerConnectionWrapper
      # see: http://propelorm.org/documentation/07-logging.html
      profiler:
        classname: {\Propel\Runtime\Util\Profiler}|string
        slowTreshold: 0.1
        time:
          precision: {3}|integer
          pad: {8}|integer
        memory:
          precision: {3}|integer
          pad: {8}|integer
        innerGlue: {":"}|integer
        outerGlue: {"|"}|integer
```

### Generator settings ###

```yaml
  ## Generator settings ##
  generator:
      defaultConnection: /One of the connections defined in database section/
      # Datasources as defined in database.connections
      connections:
          - default

      # Add a prefix to all the table names in the database.
      # This does not affect the tables phpName.
      # This setting can be overridden on a per-database basis in the schema.
      tablePrefix: {empty}|string

      # Platform class name
      platformClass: {Propel\Generator\Platform\MysqlPlatform}|string

      # The package to use for the generated classes.
      # This affects the value of the @package phpdoc tag, and it also affects
      # the directory that the classes are placed in. By default this will be
      # the same as the project. Note that the target package (and thus the target
      # directory for generated classes) can be overridden in each `<database>` and
      # `<table>` element in the XML schema.
      targetPackage: {empty}|string

      # Whether to join schemas using the same database name into a single schema.
      # This allows splitting schemas in packages, and referencing tables in another
      # schema (but in the same database) in a foreign key. Beware that database
      # behaviors will also be joined when this parameter is set to true.
      packageObjectModel: {true}|false

      # If you use namespaces in your schemas, this setting tells Propel to use the
      # namespace attribute for the package. Consequently, the namespace attribute
      # will also stipulate the subdirectory in which model classes get generated.
      namespaceAutoPackage: {true}|false

      # If you have multiple schema files in multiple subdirectories, this setting
      # tells Propel to look recursively in `schemaDir` for XML files.
      recursive: {false}|true

      schema:
          # The schema base name
          basename: {schema}|string
          # If your XML schema specifies SQL schemas for each table, you can copy the
          # value of the `schema` attribute to other attributes.
          # To copy the schema attribute to the package attribute, set this to true
          autoPackage: {false}|true
          # To copy the schema attribute to the namespace attribute, set this to true
          autoNamespace: {false}|true
          # To use the schema attribute as a prefix to all model phpNames, set this to true
          autoPrefix: {false}|true

          # Whether to transform the XML schema using the XSL file.
          # It is disabled by default.
          # The default XSL file is located under `resources/xsl/database.xsl`
          # and you can use a custom XSL file by changing the `propel.schema.xsl.file`
          # property.
          transform: {false}|true
```

#### Date/Time settings ####

```yaml
      ## Date/Time settings ##
      dateTime:

          # Enable full use of the DateTime class.
          # Setting this to true means that getter methods for date/time/timestamp
          # columns will return a DateTime object when the default format is empty.
          useDateTimeClass: {true}|false

          # Specify a custom DateTime subclass that you wish to have Propel use
          # for temporal values.
          dateTimeClass: {DateTime}|string

          # These are the default formats that will be used when fetching values from
          # temporal columns in Propel. You can always specify these when calling the
          # methods directly, but for methods like getByName() it is nice to change
          # the defaults.
          # To have these methods return DateTime objects instead, you should set these
          # to empty values
          defaultTimeStampFormat: {Y-m-d H:i:s}
          defaultTimeFormat: { %X }|string
          defaultDateFormat: { %x }|string
```

#### Customizing generated object model ####

```yaml
      objectModel:
          # Whether to add generic getter/setter methods.
          # Generic accessors are `getByName()`, `getByPosition(), ` and `toArray()`.
          addGenericAccessors: {true}|false
          # Generic mutators are `setByName()`, `setByPosition()`, and `fromArray()`.
          addGenericMutators: {true}|false
          emulateForeignKeyConstraints: {false}|true
          addClassLevelComment: {true}|false
          defaultKeyType: {phpName}|string
          addSaveMethod: {true}|false
          namespaceMap: {Map}|string

          # Whether to add a timestamp to the phpdoc header of generated OM classes.
          # If you use a versioning system, don't set this to true, or the classes
          # will be committed too often with just a date change.
          addTimeStamp: {false}|true

          # Whether to support pre- and post- hooks on `save()` and `delete()` methods.
          # Set to false if you never use these hooks for a small speed boost.
          addHooks: {true}|false

          # Some sort of "namespacing": All Propel classes will get the Prefix
          # "My_ORM_Prefix_" just like "My_ORM_Prefix_BookTableMap".
          classPrefix: {empty}|string

          # Identifier quoting may result in undesired behavior (especially in Postgres),
          # it can be disabled in DDL by setting this property to true.
          disableIdentifierQuoting: {false}|true

          # Whether the generated `doSelectJoin*()` methods use LEFT JOIN or INNER JOIN.
          useLeftJoinsInDoJoinMethods: {true}|false

          # Pluralizer class (used to generate plural forms)
          # Use StandardEnglishPluralizer instead of DefaultEnglishPluralizer for better pluralization
          # (Handles uncountable and irregular nouns)
          pluralizerClass: \Propel\Common\Pluralizer\StandardEnglishPluralizer

          # Builder classes
          builders:
              object: \Propel\Generator\Builder\Om\ObjectBuilder
              objectstub: \Propel\Generator\Builder\Om\ExtensionObjectBuilder
              objectmultiextend: \Propel\Generator\Builder\Om\MultiExtendObjectBuilder
              tablemap: \Propel\Generator\Builder\Om\TableMapBuilder
              query: \Propel\Generator\Builder\Om\QueryBuilder
              querystub: \Propel\Generator\Builder\Om\ExtensionQueryBuilder
              queryinheritance: \Propel\Generator\Builder\Om\QueryInheritanceBuilder
              queryinheritancestub: \Propel\Generator\Builder\Om\ExtensionQueryInheritanceBuilder
              interface: \Propel\Generator\Builder\Om\InterfaceBuilder
              # SQL builders
              datasql: \Propel\Generator\Builder\Sql\PgsqlDataSQLBuilder
```

## Complete file ##

Here it is the complete file, without interruptions.

```yaml
propel:
  general:
      # The name of your project.
      project: /Your-Project-Name/
      version: /propel version/

  exclude_tables:
      # you can use wildcard as you wish
      - /name of table to ignore/|string

  ### Directories and Filenames ###
  paths:
      # Directory where the project files (`schema.xml`, etc.) are located.
      # Default value is current path #
      projectDir:  {current_path}|string

      # The directory where Propel expects to find your `schema.xml` file.
      schemaDir: {current_path}|string

      # The directory where Propel should output classes, sql, config, etc.
      # Default value is current path #
      outputDir: {current_path}|string

      # The directory where Propel should output generated object model classes.
      phpDir: {current-path/generated-classes}|string

      # The directory where Propel should output the compiled runtime configuration.
      phpConfDir: {current-path/generated-conf}|string

      # The directory where Propel should output the generated DDL (or data insert statements, etc.)
      sqlDir: {current-path/generated-sql}|string

      # Directory in which your composer.json resides
      composerDir: {empty}|string

  ## All Database settings ##
  database:
      # All database sources
      connections:
          /the name of the connection/:
              adapter: {mysql}|string
              # Connection class. One of the Propel\Runtime\Connection classes
              classname: {Propel\Runtime\Connection\ConnectionWrapper}|string
              # The PDO dsn
              dsn: /your dsn/
              user: /Your username/
              password: /Your password/
              # Driver options. See http://www.php.net/manual/en/pdo.construct.php
              # options must be passed to the contructor of the connection object
              options: {empty}|array
              # See http://www.php.net/manual/en/pdo.getattribute.php
              # Attributes are set via `setAttribute()` method, after the connection object is created
              attributes: {empty}|array
              #Propel specific settings
              settings: {empty}|array
              charset: {utf8}|string
              #Array of queries to run when the database connection is initialized
              query: {empty}|array
              slaves:
                  - dsn: mysql:host=slave-host-1;dbname=bookstore
                  - ......
              # Array of folders which include the models
              model_paths:
                  - src
                  - vendor

      ## Specific adapter settings
      adapters:
          ## Mysql ##
          mysql:
              # Default table type
              tableType: {InnoDB}|MyIsam

              # Keyword used to specify the table engine in the CREATE SQL statement.
              # Defaults to 'ENGINE', users of MYSQL < 5 should use 'TYPE' instead.
              tableEngineKeyword: {ENGINE}|TYPE

          ## Sqlite ##
          sqlite:
              foreignKey: {empty}|string
              tableAlteringWorkaround: {empty}|boolean

          ## Oracle ##
          oracle:
              autoincrementSequencePattern: ${table}_SEQ

  ## Migration settings ##
  migrations:
      # Whether to specify PHP names that are the same as the column names.
      samePhpName: {false}|true

      # Whether to add the vendor info. It does provide additional information (such as full-text indexes) which can
      # affect the generation of the DDL from the schema.
      addVendorInfo: {false}|true

      # The name of migrations table
      tableName: {propel_migration}|string

      # The name of the parser class
      # If you leave this property blank, Propel looks for an appropriate parser class, based on platform: i.e.
      # if the platform is `MysqlPlatform` then parser is `\Propel\Generator\Reverse\MysqlSchemaParser`
      parserClass: {empty}|string

  ## Reverse settings
  reverse:
    # The connection to use to reverse the database
    connection: /One of the connections defined in database section/

    # Reverse parser class can be different from migration one
    # If you leave this property blank, Propel looks for an appropriate parser class, based on platform: i.e.
    # if the platform is `MysqlPlatform` then parser is `\Propel\Generator\Reverse\MysqlSchemaParser`
    parserClass: {empty}|string

  ## Runtime settings ##
  runtime:
      defaultConnection: /One of the connections defined in database section/
      # Datasources as defined in database.connections
      # This section affects config:convert command
      connections:
         - default

      ## Log and loggers definitions ##
      # For `type` and `level` options see Monolog documentation https://github.com/Seldaek/monolog
      log:
        defaultLogger:
          type: {empty}|string
          path: {empty}|string
          level: {empty}|integer

      ## Profiler configuration ##
      # To enable the profiler for a connection, set the `classname` option to \Propel\Runtime\Connection\ProfilerConnectionWrapper
      # see: http://propelorm.org/documentation/07-logging.html
      profiler:
        classname: {\Propel\Runtime\Util\Profiler}|string
        slowTreshold: 0.1
        details:
          time:
            precision: {3}|integer
            pad: {8}|integer
          memory:
            precision: {3}|integer
            pad: {8}|integer
          memDelta:
            precision: {3}|integer
            pad: {8}|integer
          memPeak:
            precision: {3}|integer
            pad: {8}|integer
        innerGlue: {":"}|integer
        outerGlue: {"|"}|integer

  ## Generator settings ##
  generator:
      defaultConnection: /One of the connections defined in database section/
      # Datasources as defined in database.connections
      connections:
          - default

      # Add a prefix to all the table names in the database.
      # This does not affect the tables phpName.
      # This setting can be overridden on a per-database basis in the schema.
      tablePrefix: {empty}|string

      # Platform class name
      platformClass: {Propel\Generator\Platform\MysqlPlatform}|string

      # The package to use for the generated classes.
      # This affects the value of the @package phpdoc tag, and it also affects
      # the directory that the classes are placed in. By default this will be
      # the same as the project. Note that the target package (and thus the target
      # directory for generated classes) can be overridden in each `<database>` and
      # `<table>` element in the XML schema.
      targetPackage: {empty}|string

      # Whether to join schemas using the same database name into a single schema.
      # This allows splitting schemas in packages, and referencing tables in another
      # schema (but in the same database) in a foreign key. Beware that database
      # behaviors will also be joined when this parameter is set to true.
      packageObjectModel: {true}|false

      # If you use namespaces in your schemas, this setting tells Propel to use the
      # namespace attribute for the package. Consequently, the namespace attribute
      # will also stipulate the subdirectory in which model classes get generated.
      namespaceAutoPackage: {true}|false

      # If you have multiple schema files in multiple subdirectories, this setting
      # tells Propel to look recursively in `schemaDir` for XML files.
      recursive: {false}|true

      schema:
          # The schema base name
          basename: {schema}|string
          # If your XML schema specifies SQL schemas for each table, you can copy the
          # value of the `schema` attribute to other attributes.
          # To copy the schema attribute to the package attribute, set this to true
          autoPackage: {false}|true
          # To copy the schema attribute to the namespace attribute, set this to true
          autoNamespace: {false}|true
          # To use the schema attribute as a prefix to all model phpNames, set this to true
          autoPrefix: {false}|true

          # Whether to transform the XML schema using the XSL file.
          # This was used in previous Propel versions to clean up the schema, but tended
          # to hide problems in the schema. It is disabled by default since Propel 1.5.
          # The default XSL file is located under `resources/xsl/database.xsl`
          # and you can use a custom XSL file by changing the `propel.schema.xsl.file`
          # property.
          transform: {false}|true

      ## Date/Time settings ##
      dateTime:

          # Enable full use of the DateTime class.
          # Setting this to true means that getter methods for date/time/timestamp
          # columns will return a DateTime object when the default format is empty.
          useDateTimeClass: {true}|false

          # Specify a custom DateTime subclass that you wish to have Propel use
          # for temporal values.
          dateTimeClass: {DateTime}|string

          # These are the default formats that will be used when fetching values from
          # temporal columns in Propel. You can always specify these when calling the
          # methods directly, but for methods like getByName() it is nice to change
          # the defaults.
          # To have these methods return DateTime objects instead, you should set these
          # to empty values
          defaultTimeStampFormat: {Y-m-d H:i:s}
          defaultTimeFormat: { %X }|string
          defaultDateFormat: { %x }|string

      objectModel:
          # Whether to add generic getter/setter methods.
          # Generic accessors are `getByName()`, `getByPosition(), ` and `toArray()`.
          addGenericAccessors: {true}|false
          # Generic mutators are `setByName()`, `setByPosition()`, and `fromArray()`.
          addGenericMutators: {true}|false
          emulateForeignKeyConstraints: {false}|true
          addClassLevelComment: {true}|false
          defaultKeyType: {phpName}|string
          addSaveMethod: {true}|false
          namespaceMap: {Map}|string

          # Whether to add a timestamp to the phpdoc header of generated OM classes.
          # If you use a versioning system, don't set this to true, or the classes
          # will be committed too often with just a date change.
          addTimeStamp: {false}|true

          # Whether to support pre- and post- hooks on `save()` and `delete()` methods.
          # Set to false if you never use these hooks for a small speed boost.
          addHooks: {true}|false

          # Some sort of "namespacing": All Propel classes with get the Prefix
          # "My_ORM_Prefix_" just like "My_ORM_Prefix_BookTableMap".
          classPrefix: {empty}|string

          # Identifier quoting may result in undesired behavior (especially in Postgres),
          # it can be disabled in DDL by setting this property to true.
          disableIdentifierQuoting: {false}|true

          # Whether the generated `doSelectJoin*()` methods use LEFT JOIN or INNER JOIN
          # (see ticket:491 and ticket:588 to understand more about why this might be
          # important).
          useLeftJoinsInDoJoinMethods: {true}|false

          # Pluralizer class (used to generate plural forms)
          # Use StandardEnglishPluralizer instead of DefaultEnglishPluralizer for better pluralization
          # (Handles uncountable and irregular nouns)
          pluralizerClass: \Propel\Common\Pluralizer\StandardEnglishPluralizer

          # Builder classes
          builders:
              object: \Propel\Generator\Builder\Om\ObjectBuilder
              objectstub: \Propel\Generator\Builder\Om\ExtensionObjectBuilder
              objectmultiextend: \Propel\Generator\Builder\Om\MultiExtendObjectBuilder
              tablemap: \Propel\Generator\Builder\Om\TableMapBuilder
              query: \Propel\Generator\Builder\Om\QueryBuilder
              querystub: \Propel\Generator\Builder\Om\ExtensionQueryBuilder
              queryinheritance: \Propel\Generator\Builder\Om\QueryInheritanceBuilder
              queryinheritancestub: \Propel\Generator\Builder\Om\ExtensionQueryInheritanceBuilder
              interface: \Propel\Generator\Builder\Om\InterfaceBuilder
              # SQL builders
              datasql:
```
