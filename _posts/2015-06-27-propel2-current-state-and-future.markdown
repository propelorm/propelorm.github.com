---
layout: post
title: Propel2 current state and its future
published: true
---

We released Propel2-alpha4 over a half a year ago and with it the last blog post.
As this is a long time for an open-source project so much used, I'm going now to let you know what its current state is
and what I planned for the future.

<!-- more -->

## 2.0.0-alpha5

Since the last official release we already have [many fixes and features committed](https://github.com/propelorm/Propel2/compare/2.0.0-alpha4...master)
which is now under the `2.0.0-alpha5` tag available. With this tag we fixed and added:

* Added `--fake` option for the commands `migration:up`, `migration:down` and `migration:migrate`. This is particular useful if you update
to Propel2 and need to flag all migrations in the database as "executed". Since propel1 stored only the last executed migration in the database,
this is necessary to not execute migration files twice. Also if you executed a migration per hand and want to let this propel know, you can now do
that using the `--fake` option.
 
* new `column` attribute `typeHint` and `foreign-key` attribute `interface`. The given value (`array` or a PHP class/interface) is used as
[PHP typeHint](http://php.net/manual/en/language.oop5.typehinting.php) for the generated method's argument.
A good use case is for example `<column name="dummy_object" type="object" typeHint="Vendor\MyDummyInterface"/>`. 

* polymorphic relations are now supported. Long wished, now supported. You can use polymorphic relations to place a relation
to another entity [with additional conditions](https://github.com/propelorm/Propel2/blob/f20146a696ba5f9684fbc27d642e431286415b15/tests/Fixtures/bookstore/schema.xml#L392-L395).

* exclude tables using new configuration entry `exclude_tables` under root node. Thanks goes out to [Alexander Zhuravlev](https://github.com/SCIF).

This is only a limited set of fixes and features. Please see [the change log](https://github.com/propelorm/Propel2/compare/2.0.0-alpha4...2.0.0-alpha5) to see all changes.

### BC breaks

* Migrations hard fail now per default. In previous versions Propel just continued with the migration if one migration file failed.
This is now not longer the case. If you want to continue even after a failure use the new `--force` option. `--force` ignores all occurring errors.

### Download

You can download this release as usual via composer. Please give it a try and [report any bugs](https://github.com/propelorm/Propel2/issues/new)
you spot:

```json
{
  "require": {
    "propel/propel": "2.0.0-alpha5"
  }
}
```

All changes/commits: [github.com/propelorm/Propel2/compare/2.0.0-alpha4...2.0.0-alpha5](https://github.com/propelorm/Propel2/compare/2.0.0-alpha4...2.0.0-alpha5)

All releases: [github.com/propelorm/Propel2/releases](https://github.com/propelorm/Propel2/releases)


## Current state

I got over the past months many emails and messages about questions like "Is Propel2 stable?", "Do you advice using v2?" or "Can we use Propel2 already?".
Please apologies if I haven't found the time to answer all your emails. The short answer here is: it depends. The long is a bit more complicated.

Propel version 2 is currently in a state where its much more stable than every version before (including v1). Thanks to way more unit tests, covering now additional
SQLite and PostgreSQL in our test suite, using PSR 1/2/3/4, having great and better Symfony2 support, and with a lot of committed fixes this is without any doubt
the best Propel we ever had. I'm using it in a lot of projects (also very big projects having millions of page impressions, a lot of migrations and release tagging ongoing)
and it runs since months in every of my projects very stable. 

As awesome as this sounds, the bad news is: Do not use it if you haven't the time to fight with BC breaks. Of course, you can already use it, but then you must know
that future bug-fixes may also introduce compatibility breaks, which may or may not lead you to some extra work.

You probably ask now, why am I not going to just release v2 as beta/stable and freeze its API? Well, because the code base of Propel 2 is pretty old and we have some showstopper, which
I definitely don't want to support in the future. One of those showstopper is 

* Enforced active-record and with it the tightly coupling of the database stuff and your domain model
* A lot of static calls/globals
* Very hard to maintain classes like [ObjectBuilder 6k LOC](https://github.com/propelorm/Propel2/blob/master/src/Propel/Generator/Builder/Om/ObjectBuilder.php)
and [QueryBuilder 2k LOC](https://github.com/propelorm/Propel2/blob/master/src/Propel/Generator/Builder/Om/QueryBuilder.php)

All of the points above are fixed with the overhaul of Propel in [PR #795](https://github.com/propelorm/Propel2/pull/795).
As you can imagine this brings a lot of compatibility breaks with it, and since I don't want to release a version which is immediately after the release outdated and not maintained anymore,
I made the decision not to release anything beyond alpha before PR #795 is not finished. This is a tough decision as its postponed Propel v2 release even more backward, but I'm sure
its worth the time.

## Future

The current roadmap is visible in our Github wiki: [github.com/propelorm/Propel2/wiki](https://github.com/propelorm/Propel2/wik).

You can see there two big points still opened, concentrated in one big pull-request: [PR #795](https://github.com/propelorm/Propel2/pull/795).

This is actual the big change Propel2 is waiting for. It dissolves the suffer of our maintain pain with the big classes,
introduces data-mapper/unit of work (which makes active-record optional), and removes the need of having global/static data,
and added more cool new features.

The result is that Propel can cover now a broader range of use cases and it can be better integrated in unit tests of modern applications.

What brings this PR now in detail?

* Separation of persisting logic and domain models. Using the data-mapper patter and an unit-of-work give us the ability to have really
 light models without base class anymore. They are pure POPOs (plain old php objects) without the need of having any magic.
* Repositories. Active getters like relation getter (`$author->getBooks()`), behavior logic and your own business logic is no longer in the entity, but in the repository or
your own service using the built-in event-dispatcher.
* Active-Record optional. `<entity name="Author" activeRecord="true"` activates the good old (good for prototyping) active-record using a generated PHP trait which is used in your model
and proxies active-record methods directly to the Session or Repository. Methods are `save`, `delete`, `isNew`, `isDeleted` and all active relation getters/counters method.
* Opens the ability to support noSQL. Since we split the persisting logic and SQL related stuff into separated classes its now easier to support noSQL databases.
* [OOP PHP code generator](https://github.com/gossi/php-code-generator) and "Code Components" allowed us to split huge ObjectBuilder, QueryBuilder, EntityMapBuilder
et cetera into little very handy code classes. This was much work as we split those huge classes into over 100 components, but it was worth the work since its maintenance
is now very easy.

### How does Propel2 data-mapper work?

Well, if you activate active-record you shouldn't notice many changes, since static creation of query classes or models behave the same using - as before - globals.

```php
<?php
include 'path/to/propel-conf.php';

$car = CarQuery::create()->findByPk(23);

$brand = new Brand(‘Ford’);
$car->setBrand($brand);
$car->save();
```

This is the current way of using Propel till alpha5.

The new data-mapper allows you to make the model lighter: 

```php
<?php
$configuration = new Propel\Runtime\Configuration('path/to/propel.yml');
$session = $configuration->getSession();

$carRepository = $configuration->getRepository(‘\Car’); //provided usually per services/DI.   

$car = $carRepository->find(23);
$brand = new Brand(‘Ford’);
$car->setBrand($brand);

$session->persist($brand);
$session->persist($car);
$session->commit();
```

Since `persist` does not result in a database operation we can save now all changes in one batch (`commit()`). Our unit-of-work is
optimized to execute inserts in bulk inserts and updates in a more optimized way, which results basically in a
faster Propel for bigger change-sets.

### Current state of this Pull-Request?

Since I need to completely rewrite Propel's object/query/tableMap builder, service container, persisting logic, all behaviors and
need to touch almost every test of our 3288 tests (btw, almost more than 700 more tests than Propel v1) it took a while
to implement the rough basics. What has been done:

* Almost 10% of test suite is green (which means its compatible to [old BookstoreTest](https://github.com/marcj/Propel2/blob/data-mapper/tests/Propel/Tests/BookstoreTest.php))
* Rewrote [big builder](https://github.com/marcj/Propel2/blob/data-mapper/src/Propel/Generator/Builder/Om/ObjectBuilder.php) into [little peaces](https://github.com/marcj/Propel2/tree/data-mapper/src/Propel/Generator/Builder/Om/Component)
* MySQL/SQLite: Inserts/Updates/Deletes
* Behaviors rewritten: AggregateField (aka AggregateColumn), Archivable, AutoAddPk, ConcreteInheritance, Delegate, Timestampable, Sluggable

What needs to be done:

* Rewrite behavior: NestedSet, l18n, Sortable, QueryCache, Validate, Versionable
* Make more tests green. A lot of tests are based on `Map\*TableMap::getTableMap()` which is not available anymore. `$configuration->getEntityMap(FooEntity::class)` is now the new way.
* Make PostgreSQL support working

You can check-out my [current branch](https://github.com/marcj/Propel2) if you want to play around with it, but take care: Its far from being perfect, its slower than old versions and generated code look may look ugly.

I'm planing now to write every ~two weeks a blog post about what has been done in that data-mapper development, so you're up to date.
Please don't judge badly about the slow development,
if it is not as fast as you want to see - the contribution to this new approach is rather low since it is very complicated to build a big
library like this and costs much time. But I really believe we all are more than happy when we have finally an alternative, full featured data-mapper ORM to doctrine, which
is on top faster, easier to use and uses less magic.

Next plan is to implement the l18n and nested_set behavior, which both a pretty big. Hope to provide you more updates soon. 
