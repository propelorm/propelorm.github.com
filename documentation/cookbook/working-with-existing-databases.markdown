---
layout: documentation
title: Working With Existing Databases
---

# Working With Existing Databases #

The following topics are targeted for developers who already have a working database solution in place, but would like to use Propel to work with the data. Propel provides command-line utilities to help migrate database structures into Propel's abstract schema format, enabling model generation and cross-database support.

## Working with Database Structures

Propel uses an abstract XML schema file to represent databases (see the [XML Schema Format](/documentation/reference/schema.html)). Propel builds SQL for your target database based on this schema — and can also reverse-engineer the schema file based on existing database metadata.

### Creating an XML Schema from a DB Structure

Propel provides the `database:reverse` command to reverse-engineer your database into a `schema.xml` file. This is useful when integrating with a legacy database or starting a project based on an existing schema.

To generate the schema:

1. If your database includes tables you don’t want to include (e.g. third-party ones), [exclude them](/documentation/reference/configuration-file.html#exclude-tables) in the configuration using `exclude_tables`.

2. Create a working directory for your project:

    ```bash
    mkdir legacyapp && cd legacyapp
    ```

3. Run the `database:reverse` command with your DSN or connection name:

    ```bash
    propel database:reverse "mysql:host=localhost;dbname=db;user=root;password=pwd"
    ```

    The argument is a [PDO DSN](/documentation/reference/configuration-file.html#dsn), quoted to preserve special characters.

4. Review the console output for any errors or warnings, then inspect the generated `schema.xml` file.

5. The schema file will be located in the `generated-reversed-database/` directory by default. You can now proceed with model generation:

    ```bash
    propel model:build
    ```

#### Additional Options

You can customize the behavior using the following options:

| Option             | Description                                                                                 | Default                        |
|--------------------|---------------------------------------------------------------------------------------------|--------------------------------|
| `--output-dir`     | Where to write the generated `schema.xml`                                                  | `generated-reversed-database` |
| `--database-name`  | Name to use in the schema XML `<database name="...">`                                      | `default` or connection name   |
| `--schema-name`    | Logical schema name used inside the generated file (helpful for multi-schema setups)       | `schema`                       |
| `--namespace`      | PHP namespace to use for generated model classes (optional)                                | —                              |

Example using options:

```bash
propel database:reverse "mysql:host=localhost;dbname=db;user=root;password=pwd" \
  --output-dir=app/schema \
  --database-name=legacy \
  --schema-name=core \
  --namespace="Legacy\Models"
```

#### Limitations

`database:reverse` does not reverse-engineer views, materialized views, or enum types in PostgreSQL. These structures are not currently supported by Propel as first-class schema objects. If you want to use a view within Propel, you can manually define it as a `<table>` with `skipSql="true"` to generate read-only model/query classes.


Note: database:reverse replaces the old reverse command in Propel 2.x. The alias reverse is still available but may be deprecated in future versions.

>**Tip**The reverse engineering classes may not be able to provide the same level of detail for all databases. In particular, metadata information for SQLite is often very basic since SQLite is a typeless database.

### Migrating Structure to a New RDBMS ###

Because Propel has both the ability to create XML schema files based on existing database structures and to create RDBMS-specific DDL SQL from the XML schema file, you can use Propel to convert one database into another.

To do this you would simply:

1. Follow the steps above to create the `schema.xml` file from existing db.

2. Then you would change the target database type and specify connection URL for new
   database in the project's configuration file (for further information, see the
   [configuration](/documentation/10-configuration.html) documentation)

3. And then run the `sql:build` task to generate the new DDL:

    ```bash
    $ propel sql:build
    ```

4. And (optionally) the `sql:insert` task to create the new database:

    ```bash
    $ propel sql:insert
    ```
