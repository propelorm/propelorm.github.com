---
layout: post
title: ! 'New In Propel 1.5: Concrete Table Inheritance And A New Behavior. Wait,
  That''s The Same Thing.'
published: true
---

Inheritance is very common in the Object-Oriented world, much less in the database world. Yet, being able to extend a model allows for clean code organization and more complex logic. Some RDBMS, like PostgreSQL, offer <a href="http://www.postgresql.org/docs/8.1/static/ddl-inherit.html">native inheritance</a>. Starting with Propel 1.5, Propel gives the same ability to every compatible storage (including MySQL, Oracle, SQLite, and MSSQL) through the new <span style="font-family: courier new,monospace;">concrete_inheritance</span> behavior.

## Understanding Inheritance Design Patterns

Propel has offered <a href="http://propel.phpdb.org/trac/wiki/Users/Documentation/1.5/Inheritance#SingleTableInheritance">Single Table Inheritance</a> for a long time. This feature allows several model classes to extend one another, but all the records are persisted in a single table. For complex inheritance patterns, this leads to tables with a lot of columns - and a lot of NULL values in the records.

Another common strategy for implementing inheritance in an ORM is called <a href="http://www.martinfowler.com/eaaCatalog/concreteTableInheritance.html">Concrete Table Inheritance</a>. In this case, each child class has its own table for storage, and the inheritance adds the columns of the parent class to each of the child classes. It's this implementation that Propel 1.5 introduces - with a twist.

<strong>Tip</strong>: If you want to know more about table inheritance and other design patterns, I highly recommend the reading of "<a href="http://martinfowler.com/books.html#eaa">Patterns Of Enterprise Application Architecture</a>", by Martin Fowler.

## The Concrete Table Inheritance Behavior

In the following example, the `article` and `video` tables use this behavior to inherit the columns and foreign keys of their parent table, `content`:

~~~xml
<table name="content">
  <column name="id" type="INTEGER" primaryKey="true" autoIncrement="true">
  <column name="title" type="VARCHAR" size="100">
  <column name="category_id" required="false" type="INTEGER">
  <foreign-key foreignTable="category" onDelete="cascade">
    <reference local="category_id" foreign="id" />
  </foreign-key>
</table>

<table name="category">
  <column name="id" required="true" primaryKey="true" autoIncrement="true" type="INTEGER" />
  <column name="name" type="VARCHAR" size="100" primaryString="true" />
</table>

<table name="article">
  <behavior name="concrete_inheritance">
    <parameter name="extends" value="content" />
  </behavior>

  <column name="body" type="VARCHAR" size="100"/>
</table>

<table name="video">
  <behavior name="concrete_inheritance">
    <parameter name="extends" value="content" />
  </behavior>
  <column name="resource_link" type="VARCHAR" size="100"/>
</table>
~~~

The behavior copies the columns of the parent table to the child tables. That means that the generated `Article` and `Video` models have a `Title` property and a `Category` relationship:

~~~php
<?php
// create a new Category
$cat = new Category();
$cat->setName('Movie');
$cat->save();
// create a new Article
$art = new Article();
$art->setTitle('Avatar Makes Best Opening Weekend in the History');
$art->setCategory($cat);
$art->setContent('With $232.2 million worldwide total, Avatar had one of the best-opening weekends in the history of cinema.');
$art->save();
// create a new Video
$vid = new Video();
 $vid->setTitle('Avatar Trailer');
$vid->setCategory($cat);
$vid->setResourceLink('http://www.avatarmovie.com/index.html')
$vid->save();
~~~

## Model Inheritance

If Propel stopped there, the `concrete_inheritance` behavior would only provide a shorcut to avoid repeating tags in the schema. But the behaviors uses PHP's inheritance system to make the models of the child tables extends the model of the parent table: the `Article` and `Video` classes actually extend the `Content` class:

~~~php
<?php
class Content extends BaseContent
{
  public function getCategoryName()
  {
    return $this-&gt;getCategory()-&gt;getName();
  }
}
echo $art->getCategoryName(); // 'Movie'
echo $vid->getCategoryName(); // 'Movie'
~~~

Imagine how convenient this class extension is to avoid repeating code across similar classes. And contrary to the Single Table Inheritance pattern, the Concrete Table Inheritance scales without problems to dozens of tables with very different columns.

The fact that the behavior system, introduced in Propel 1.4, provides the best implementation for the Concrete Table Inheritance, shows how powerful behaviors can be. Propel keeps getting new features without adding too much complexity to the core.

## One More Thing

Usually, ORMs stop the Concrete Table Inheritance implementation there. But not Propel. The `concrete_inheritance` behavior does not only copy the <em>table structure</em>, it also copies <em>data</em>.<p /> Every time you save an `Article` or a `Video` object, Propel saves a copy of the `title` and `category_id` columns in a `Content` object. Consequently, retrieving objects regardless of their child type becomes very easy:

~~~php
<?php
$conts = ContentQuery::create()->find();
foreach ($conts as $content) {
  echo $content->getTitle() . "(". $content->getCategoryName() ")/n";
}
// Avatar Makes Best Opening Weekend in the History (Movie)
// Avatar Trailer (Movie)
~~~

The resulting relational model is denormalized - in other terms, data is copied across tables - but the behavior takes care of everything for you. That allows for very effective read queries on complex inheritance structures.

Check out the brand new <a href="http://propel.phpdb.org/trac/wiki/Users/Documentation/1.5/Inheritance#ConcreteTableInheritance">Inheritance Documentation</a> for more details on using and customizing this behavior.
