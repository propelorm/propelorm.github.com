---
layout: documentation
title: UUID Binary Columns
---

# Store UUIDs as binary data #

The `UUID_BINARY` column type is a special type to store UUID data in binary columns. It can be used on all database systems, but it is particularly useful when no native UUID column type is available (MySQL, SQLite).

To create a `UUID_BINARY` column, set the type in schema.xml:

```xml
<table name="my_table">
  <column name="uuid_bin" type="UUID_BINARY" defaultExpr="uuid_to_bin(UUID(), 1)"/>
</table>
```

The column can be used like a text-based column:
```php
$myTableObject->setUuidBin('8ddb2ec4-f996-4777-b4f4-d59399530734');
$myTableObject->getUuidBin(); // will return '8ddb2ec4-f996-4777-b4f4-d59399530734'
MyTableQuery::create()->filterByUuidBin(['8ddb2ec4-f996-4777-b4f4-d59399530734', 'f7195b5f-4544-4854-ac3e-99d6718e32c7']);
```

Propel will automatically convert the UUIDs between string and binary representation when interacting with the database. Data is stored in a native binary format:

| DBMS | Type |
| ---  | ---  |
| MySQL | BINARY(16) |
| MS SQL | BINARY(16) |
| Oracle | RAW(16) |
| PostgreSQL | BYTEA |
| SQLite | BLOB |

## Manual conversion ##

In statements where UUIDs cannot be converted automatically, Propel offers methods for manual conversion:
```php
UuidConverter::uuidToBin(string $uuid, bool $swapFlag = true): string
UuidConverter::binToUuid(string $bin, bool $swapFlag = true): string
```
These work similar to MySQL's [UUID_TO_BIN()](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_uuid-to-bin) and [BIN_TO_UUID()](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_bin-to-uuid) functions.

Example:
```php
$arrayData = MyTableQuery::create()->setFormatter(SimpleArrayFormatter::class)->find();
foreach($arrayData as $rows){
    $rows['uuid_bin'] = UuidConverter::binToUuid($row['uuid_bin'];
}
```

## The swap flag ##

The second parameter of the conversion functions turns swapping of UUIDs on or off. If activated, the order of the first eight bytes will be changed to allow more efficient indexing and storing (a good explanation can be found [here](https://lefred.be/content/mysql-uuids/)).
This only applies to v1/v2 UUIDs, which is the recommended type for databases. Native generator function like `UUID()` in MySQL produce these kinds of UUIDs. If you use random UUIDs (v4), swapping has no advantage.

**Propel swaps UUID data per default**. But the behavior can be changed in schema.xml by setting vendor information:

```xml
<database>
  <vendor type="mysql">
    <parameter name="UuidSwapFlag" value="false"/>
  </vendor>
  ...
```

Note that changing swapping behavior is not detected by Propel at the moment - any necessary migrations have to be done manually.

## Migrating between `CHAR` and `UUID_BINARY` columns on MySQL ###

For MySQL, Propel generates migrations to change column type between `CHAR`-types and `UUID_BINARY`, where data from the existing column is converted into a new column, which then replaces the old one.

To migrate from `UUID_BINARY` to a `CHAR` type, the column content needs to be declared as uuid in schema.xml, so that propel knows that the found `BINARY` column contains UUID data:
```xml
<table name="my_table">
  <column name="uuid_bin" type="CHAR" size="36" content="UUID"/>
</table>
```

> **!** The migrations will not work on MySQL below version 8 or MariaDB, since the used builtin functions are not available there **!**

## Enabling native `UUID` columns on MariaDB

MariaDB uses the same configuration as MySQL, which cannot use the native `UUID` column type in MariaDB. Declaring a column type as `UUID` will generate `UUID_BINARY` columns.

However, it is possible to override this behavior in the [propel configuration file](/documentation/reference/configuration-file.html):

```yaml
propel:
    adapters:
      mysql:
        uuidColumnType: native # <------ here
```

> **!** If you use multiple databases, note that this setting affects all MySQL systems in you setup. At the moment  it is not possible to set the behavior on a database level **!**

## Issues and bugs ##

Please report issues and bugs to the [Propel2 issues tracker on GitHub](https://github.com/propelorm/Propel2/issues)