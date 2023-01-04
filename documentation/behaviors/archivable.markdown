---
layout: documentation
title: Archivable Behavior
---

# Archivable Behavior #

The `archivable` behavior gives model objects the ability to be copied to an archive table. By default, the behavior archives objects on deletion, which makes it the Propel implementation of the "soft delete" pattern.
## List of Parameters ##

Adjust the behavior by adding parameters:
```xml
<behavior name="archivable">
  <parameter name="archive_table" value="special_book_archive" />
  <parameter name="archive_phpname" value="MySpecialBookArchive" />
  <parameter name="log_archived_at" value="true" />
  <parameter name="archived_at_column" value="archival_date" />
  <parameter name="inherit_foreign_key_relations" value="true" />
  <parameter name="inherit_foreign_key_constraints" value="true" />
</behavior>
```

These parameters are available:

| Parameter                      | Value           | Description | Default     |
|--------------------------------|-----------------|-------------|-------------|
|archive_table                   | literal         | Name of the table storing archive data. Will be created if it does not exist. Cannot be combined with the `archive_class` parameter. |  Original table name with suffix `_archive`| |
| archive_phpname                | literal         | Sets the name of the generated PHP model and query classes. | Pascal-case version of `archive_table` |
| sync                           | `true`/`false`  | If true, changes to the source table will also be applied to the `archive_table`. | `false` |
| archive_class                  | literal         | Name of an existing Model class. The underlying table will be used to store the archive. Cannot be combined with the `archive_class` parameter. | none |
| log_archived_at                | `true`/`false`  | Enables or disables additional archival timestamp column. | `true` |
| archived_at_column             | literal         | Sets the name of the column storing the archival datetime. | `archived_at` |
| archive_on_insert              | `true`/`false`  | Archive row data on insert. | `false` |
| archive_on_update              | `true`/`false`  | Archive row data on update. | `false` |
| archive_on_delete              | `true`/`false`  | Archive row data on delete. | `true`  |
| inherit_foreign_key_relations  | `true`/`false`  | Inherit foreign key relations from the source table. Will add getters/setters on model and join methods on query class of archive table | `false` |
| inherit_foreign_key_constraints| `true`/`false`  | Same as `inherit_foreign_key_relations`, but also creates foreign key constraints in the database to the referenced columns of the source table. | `false ` |
| foreign_keys                   | list (see blow) | Manually set foreign keys on archive table | none |

## Basic Usage ##

In the `schema.xml`, use the `<behavior>` tag to add the `archivable` behavior to a table:

```xml
<table name="book">
  <column name="id" required="true" primaryKey="true" autoIncrement="true" type="integer" />
  <column name="title" type="varchar" required="true" primaryString="true" />
  <behavior name="archivable" />
</table>
```

Rebuild your model, insert the table creation sql again, and you're ready to go. The model now has one new table, `book_archive`, with the same columns as the original `book` table. This table stores the archived `books` together with their archive date. To archive an object, call the `archive()` method:

```php
<?php
$book = new Book();
$book->setTitle('War And Peace');
$book->save();
// copy the current Book to a BookArchive object and save it
$archivedBook = $book->archive();
```

The archive table contains only the freshest copy of each archived objects. Archiving an object twice doesn't create a new record in the archive table, but updates the existing archive.

The `book_archive` table has generated ActiveRecord and ActiveQuery classes, so you can browse the archive at will. The archived objects have the same primary key as the original objects. In addition, they contain an `ArchivedAt` property storing the date where the object was archived.

```php
<?php
// find the archived book
$archivedBook = BookArchiveQuery::create()->findPk($book->getId());
echo $archivedBook->getTitle(); // 'War And Peace'
echo $archivedBook->getArchivedAt(); // 2011-08-23 18:14:23
```

The ActiveRecord class of an `archivable` model has more methods to deal with the archive:

```php
<?php
// restore an object to the state it had when last archived
$book->restoreFromArchive();
// find the archived version of an existing book
$archivedBook = $book->getArchive();
// populate a book based on an archive
$book = new book();
$book->populateFromArchive($archivedBook);
```

By default, an `archivable` model is archived just before deletion:

```php
<?php
$book = new Book();
$book->setTitle('Sense and Sensibility');
$book->save();
// delete and archive the book
$book->delete();
echo BookQuery::create()->count(); // 0
// find the archived book
$archivedBook = BookArchiveQuery::create()
  ->findOneByTitle('Sense and Sensibility');
```

>**Tip**The behavior does not take care of archiving the related objects. This may be surprising on deletions if the deleted object has 'ON DELETE CASCADE' foreign keys. If you want to archive relations, override the generated `archive()` method in the ActiveRecord class with your custom logic.

To recover deleted objects, use `populateFromArchive()` on a new object and save it:

```php
<?php
// create a new object based on the archive
$book = new Book();
$book->populateFromArchive($archivedBook);
$book->save();
echo $book->getTitle(); // 'Sense and Sensibility'
```

If you want to delete an `archivable` object without archiving it, use the `deleteWithoutArchive()` method generated by the behavior:

```php
<?php
// delete the book but don't archive it
$book->deleteWithoutArchive();
```

## Archiving A Set Of Objects ##

The `archivable` behavior also generates an `archive()` method on the generated ActiveQuery class. That means you can easily archive a set of objects, in the same way you archive a single object:

```php
<?php
// archive all books having a title starting with "war"
$nbArchivedObjects = BookQuery::create()
  ->filterByTitle('War%')
  ->archive();
```

`archive()` returns the number of archived objects, and not the current ActiveQuery object, so it's a termination method.

>**Tip**Since the `archive()` method doesn't duplicate archived objects, it must iterate over the results of the query to check whether each object has already been archived. In practice, `archive()` issues 2n+1 database queries, where `n` is the number of results of the query as returned by a `count()`.

As explained earlier, an `archivable` model is archived just before deletion by default. This is also true when using the `delete()` and `deleteAll()` methods of the ActiveQuery class:

```php
<?php
// delete and archive all books having a title starting with "war"
$nbDeletedObjects = BookQuery::create()
  ->filterByTitle('War%')
  ->delete();

// use deleteWithoutArchive() if you just want to delete
$nbDeletedObjects = BookQuery::create()
  ->filterByTitle('War%')
  ->deleteWithoutArchive();

// you can also turn off the query alteration on the current query
// by calling setArchiveOnDelete(false) before deleting
$nbDeletedObjects = BookQuery::create()
  ->filterByTitle('War%')
  ->setArchiveOnDelete(false)
  ->delete();
```

## Archiving on Insert, Update, or Delete ##

As explained earlier, the `archivable` behavior archives objects on deletion by default, but insertions and updates don't trigger the `archive()` method. You can disable the auto archiving on deletion, as well as enable it for insertion and update, in the behavior `<parameter>` tags. Here is the default configuration:

```xml
<table name="book">
  <column name="id" required="true" primaryKey="true" autoIncrement="true" type="integer" />
  <column name="title" type="varchar" required="true" primaryString="true" />
  <behavior name="archivable">
    <parameter name="archive_on_insert" value="false" />
    <parameter name="archive_on_update" value="false" />
    <parameter name="archive_on_delete" value="true" />
  </behavior>
</table>
```

If you turn on `archive_on_insert`, a call to `save()` on a new ActiveRecord object archives it - unless you call `saveWithoutArchive()`.

If you turn on `archive_on_update`, a call to `save()` on an existing ActiveRecord object archives it, and a call to `update()` on an ActiveQuery object archives the results as well. You can still use `saveWithoutArchive()` on the ActiveRecord class and `updateWithoutArchive()` on the ActiveQuery class to skip archiving on updates.

Of course, even if `archive_on_insert` or any of the similar parameters isn't turned on, you can always archive manually an object after persisting it by simply calling `archive()`:

```php
<?php
// create a new object, save it, and archive it
$book = new Book();
$book->save();
$book->archive();
```

## Archiving To Another Database ##

The behavior can use another database connection for the archive table, to make it safer. To allow cross-database archives, you must declare the archive schema manually in another XML schema, and reference the archive class in the behavior parameter:

```xml
<database name="main">
  <table name="book">
    <column name="id" required="true" primaryKey="true" autoIncrement="true" type="integer" />
    <column name="title" type="varchar" required="true" primaryString="true" />
    <behavior name="archivable">
      <parameter name="archive_class" value="MyBookArchive" />
    </behavior>
  </table>
</database>
<database name="backup">
  <table name="my_book_archive" phpName="MyBookArchive">
    <column name="id" required="true" primaryKey="true" type="integer" />
    <column name="title" type="varchar" required="true" primaryString="true" />
    <column name="archived_at" type="timestamp" />
  </table>
</database>
```

The archive table must have the same columns as the archivable table, but without autoIncrements, and without foreign keys.

With this setup, the behavior uses `MyBookArchive` and `MyBookArchiveQuery` for all operations on archives, and therefore uses the `backup` connection.

## Foreign Keys ##

There are four ways to handle foreign keys on the archive table:

1. Inherit all relations from the source table, but without database constraints.  
2. Same as above, but also add database constraints.
3. Declare foreign keys on the behavior.
4. Manually create the archive table with foreign keys.

### Inheriting Relations But Not Foreign Key Constraints

Use this option if foreign key references might break after archiving. This often occurs when data in the referenced table is archived at a different time, so that referenced data might be in the original table or in the archive. In this case, it is not possible to enforce a foreign key constraint on database level. However, Propel can still create logical relations between model and query classes of the source and archive table, which makes it easy to work with the tables:

```php
// getters/setters on the archive model
$author = $archivedBook->getAuthor();
$archivedBook->setAuthor($myAuthor);

// getters/setters on the source model
$archivedBook = $author->getArchivedBooks();
$author->addArchivedBook($archivedBook);

// join methods on the archive query object
ArchivedBookQuery::create()->joinWithAuthor();

// join methods on the source query object
Author::create()->useArchivedBookQuery();
```

To inherit relations from the source table, set the `inherit_foreign_key_relations` parameter:

```xml
<behavior name="archivable">
  <parameter name="inherit_foreign_key_relations" value="true" />
</behavior>
```

### Inheriting Relations And Foreign Key Constraints

If foreign key references of archived data point to the same table as from the source table, it is best to ensure referential integrity through a database constraint. A typical example is when the the archived table contains links to lookup tables.

To create foreign key constraints in the database, use the `inherit_foreign_key_constraints` parameter:

```xml
<behavior name="archivable">
  <parameter name="inherit_foreign_key_constraints" value="true" />
</behavior>
```

### Declaring Foreign Keys On The Behavior

For more complex scenarios, it is possible to declare foreign keys on the behavior. This allows to add relations that do not apply to the source table and gives more control over which relations and constraints should be created. A typical example is an archive table that references data in another archived table.
Relations defined through this parameter override inherited relations with the same name, so it can be used to fix up individual references that break during archival.

With the `foreign_keys` parameter, a list of foreign key definitions is passed to the behavior. The parameters are the same as in regular `<foreign-key>` declarations, except that source and target column are also declared by the parameter (`localColumn` and `foreignColumn` respectively), instead of a `<reference>` tag. That means complex foreign keys cannot be declared on behaviors.

Using the `skipSql` parameter will create relations without creating database foreign key constraints.  


```xml
<behavior name="archivable">
  <parameter-list name="foreign_keys">

    <!-- archived data points to other archived table -->
    <parameter-list-item>
      <parameter name="localColumn" value="sale_id" />
      <parameter name="foreignTable" value="archived_sale" />
      <parameter name="foreignColumn" value="id" />
    </parameter-list-item>

    <!-- referenced data may or may not be archived, create relation to both tables without constraints -->
    <parameter-list-item>
      <parameter name="localColumn" value="customer_id" />
      <parameter name="foreignTable" value="customer" />
      <parameter name="foreignColumn" value="id" />
      <parameter name="skipSql" value="true" />
    </parameter-list-item>
    <parameter-list-item>
      <parameter name="localColumn" value="customer_id" />
      <parameter name="foreignTable" value="archived_customer" />
      <parameter name="foreignColumn" value="id" />
      <parameter name="skipSql" value="true" />
    </parameter-list-item>

</behavior>
```

### Manually creating the archive table with foreign keys

Finally, if complex foreign keys are needed, or the statement for the `foreign_keys` parameter grows out of proportion, the archive table can be added like a regular table with regular foreign key declarations:

```xml
<table name="item">
  <behavior name="archivable"/>
  <column name="sale_id" type="integer" />
  <foreign-key foreignTable="sale">
    <reference local="sale_id" foreign="id"/>
  </foreign-key>
</table>

<table name="sale">
  <behavior name="archivable"/>
  <column name="id" type="integer" primaryKey="true"/>
</table>

<table name="item_archive">
  <column name="sale_id"/>
  <foreign-key foreignTable="sale_archive">
    <reference local="sale_id" foreign="id"/>
  </foreign-key>
</table>

<table name="sale_archive">
  <column name="id" type="integer" primaryKey="true"/>
</table>
```

Combined with the `sync` parameter, this allows precise declaration of table constraints without losing flexibility. 

## Syncing Changes From Source Table To Archive Table

When the table given in the `archive_table` parameter exists in schema.xml, all columns from the original table must be declared on it as well. This can be cumbersome, when schema changes to the source table have to be copied to the archive table. With the `sync` parameter, Propel automatically keeps the archive table up to date with the source table.

With sync enabled, changes to columns and indexes in the source table are applied to the archive table as well. Foreign key relations are also include when the `inherit_foreign_key_relations` or `inherit_foreign_key_constraints` parameter is set.

Note that sync will not override explicit declarations on the archive table, which allows to override individual elements from the source table.


```xml
<table name="my_data">
  <behavior name="archivable">
    <parameter name="archive_table" value="my_archived_data"/>
    <parameter name="sync" value="true"/>
    <parameter name="inherit_foreign_key_constraints" value="true"/>
  </behavior>
  <column name="id" type="integer" />
  <column name="my_text" type="varchar" size="32"/>
  <column name="fk1_id" type="integer" />
  <foreign-key foreignTable="fk_table1">
    <reference local="fk1_id" foreign="id"/>
  </foreign-key>
  <column name="fk2_id" type="integer" />
  <foreign-key foreignTable="fk_table2" name="second_fk">
    <reference local="fk2_id" foreign="id"/>
  </foreign-key>
  <!-- more columns, fks and indices -->
</table>

<table name="my_archived_data">
  <!-- keep old size of varchar column -->
  <column name="my_text" type="varchar" size="99"/>
  <!-- override second fk -->
  <column name="fk2_id" type="integer" />
  <foreign-key foreignTable="archived_fk_table2" name="second_fk">
    <reference local="fk2_id" foreign="id"/>
  </foreign-key>
  <!-- all other data will be copied by sync -->
</table>
```


## Migrating From `soft_delete` ##

If you use `archivable` as a replacement for the `soft_delete` behavior, here is how you should update your code:

```php
<?php
// do a soft delete
$book->delete(); // with soft_delete
$book->delete(); // with archivable

// do a hard delete
// with soft_delete
$book->forceDelete();
// with archivable
$book->deleteWithoutArchive();

// find deleted objects
// with soft_delete
$books = BookQuery::create()
  ->includeDeleted()
  ->where('Book.DeletedAt IS NOT NULL')
  ->find();
// with archivable
$bookArchives = BookArchiveQuery::create()
  ->find();

// recover a deleted object
// with soft_delete
$book->unDelete();
// with archivable
$book = new Book();
$book->populateFromArchive($bookArchive);
$book->save();
```
