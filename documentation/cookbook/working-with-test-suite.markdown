---
layout: documentation
title: Working with Propel's Test Suite
---

# Working with Propel's Test Suite #

Propel uses [PHPUnit](http://phpunit.de/) to test the build and runtime frameworks.

You can find the unit test classes and support files in the `tests/` directory.

## Setup requirements ##

### Setup the environment ###

You need to install the following PHP modules:

* php7.*-mysql
* php7.*-sqlite
* php7.*-pgsql
* php7.*-iconv

Increase the memory_limit in your php.ini to something very high:

```
memory_limit = 512M
```

Set a default timezone in your `php.ini`:

```
date.timezone = Europe/Berlin
```

>**Info**All following commands need to be executed in the propel root directory.

### Install Dev Dependencies ###

In order to run the tests, you must install the development dependencies. Propel
uses [Composer](https://getcomposer.org/) to manage its dependencies. To install
it, you are done with:

    $ wget http://getcomposer.org/composer.phar

Or if you don't have `wget` on your machine:

    $ curl -s http://getcomposer.org/installer | php

Then run Composer to install the necessary stuff:

    $ php composer.phar install


### Setup Database ###

We provide three scripts that setup your database.

Those scripts drop basically all required databases/schemas (`test`,
`second_hand_books`, `contest`, `bookstore_schemas`, `migration`) and re-create it,
so the test suite has an empty database.

```
tests/bin
├── setup.mysql.sh
├── setup.pgsql.sh
└── setup.sqlite.sh
```

Call one of those scripts in your console:

```bash
./tests/bin/setup.mysql.sh
```

If your database is configured using a password or a different user than the
default, you can configure it via environment variables.

Example:

```bash
DB_HOSTNAME=192.168.0.3 DB_USER=nonroot DB_PW=topsecret ./tests/bin/setup.mysql.sh
DB_HOSTNAME=192.168.0.3 DB_USER=postgres DB_PW=secret ./tests/bin/setup.pgsql.sh
./tests/bin/setup.sqlite.sh #does not have any configuration ability
```

>**Info**./tests/bin/setup.sqlite.sh just removes the file at `tests/test.sq3` to trigger a re-creating by our test suite.

## Writing Tests ##

### How the Tests Work ###

Every method in the test classes that begins with 'test' is run as a test case by
PHPUnit. All tests are run in isolation; the `setUp()` method is called at the
beginning of ''each'' test and the `tearDown()` method is called at the end.

The `BookstoreTestBase` class specifies `setUp()` and `tearDown()` methods which
populate and depopulate, respectively, the database. This means that every unit
test is run with a cleanly populated database. To see the sample data that is
populated, take a look at the `BookstoreDataPopulator` class. You can also add
data to this class, if needed by your tests; however, proceed cautiously when
changing existing data in there as there may be unit tests that depend on it.
More typically, you can simply create the data you need from within your test
method. It will be deleted by the `tearDown()` method, so no need to clean up
after yourself.


### Write first test ###

If you've made a change to a template or to Propel behavior, the right thing to do is write a unit test that ensures that it works properly -- and continues to work in the future.

Writing a unit test often means adding a method to one of the existing test classes. For example, let's test a feature in the Propel templates that supports saving of objects when only default values have been specified. Just add a `testSaveWithDefaultValues()` method to the `GeneratedObjectTest` class, as follows:

```php
<?php
# tests/Propel/Tests/MyFirstTest.php

namespace Propel\Tests;

use Propel\Tests\TestCaseFixturesDatabase;

/**
 * @group database
 */
class MyFirstTest extends TestCaseFixturesDatabase
{
    /**
     * Test saving object when only default values are set.
     */
    public function testSaveWithDefaultValues() {

      // Relies on a default value of 'Penguin' specified in schema
      // for publisher.name col.

      $pub = new Publisher();
      $pub->setName('Penguin');
        // in the past this wouldn't have marked object as modified
        // since 'Penguin' is the value that's already set for that attribute
      $pub->save();

      // if getId() returns the new ID, then we know save() worked.
      $this->assertNotNull($pub->getId(), "Expect Publisher->save() to work with only default values.");
    }
}
```

Run the test again using the command line to check that it passes:

    $ ./vendor/bin/phpunit GeneratedObjectTest


You can also write additional unit test classes to any of the directories in `test/testsuite/` (or add new directories if needed). The `./vendor/bin/phpunit` command will find these files automatically and run them.

## Using the correct TestCase class ##

Propel provides several test cases you can extend to have some additional functionalities.


`Propel\Tests\TestCase`

This class gives your some basic methods to see against which database the current suite is running at.

### Automatically Build Fixtures ###

`Propel\Tests\TestCaseFixtures`

This test case class makes sure all fixtures in `tests/Fixtures/` have been build and are up to date.
You have to use this class if you need built model loaded connection configurations.

We place in `tests/Fixtures/fixtures_built` the current used settings/credentials used in the last running test suite.

### Automatically Build Fixtures and Update Database ###

`Propel\Tests\TestCaseFixturesDatabase`

This test case class makes, in addition to the one above, sure that the database
schema is up to date as well. You have to use this if you want to have, in addition
to the one above, a communication with the database through your models.

## Using the correct test group ##

In our continuous integration setup we have four different builds running. One of those builds does execute tests
that do not communicate with the database. So we need at each test class a `@group database` if that test does only work with
configured database.

![Builds](/images/documentation/cookbook-test-suite-builds.png)

The following scripts are used to start phpunit in our build server.

```
tests/bin/
├── phpunit.agnostic.sh
├── phpunit.mysql.sh
├── phpunit.pgsql.sh
└── phpunit.sqlite.sh
```

You can see there that we filter by those groups:

```bash
$ cat tests/bin/phpunit.agnostic.sh
#!/bin/sh
./vendor/bin/phpunit -c tests/agnostic.phpunit.xml
```

```bash
$ cat tests/bin/phpunit.pgsql.sh
#!/bin/sh
./vendor/bin/phpunit -c tests/pgsql.phpunit.xml
```

and so on.

The actual Travis build server is configured like this:

```yaml
script:
    - ./vendor/bin/phpunit -v -c tests/$DB.phpunit.xml;
```

To group our tests we have basically only two `@group` annotation values that
PHPUnit is going to filter for us.

Example:

```php
<?php
# tests/Propel/Tests/Generator/Reverse/MysqlSchemaParserTest.php
/**
 * Tests for Mysql database schema parser.
 *
 * @author William Durand
 *
 * @group database
 */
class MysqlSchemaParserTest extends TestCaseFixturesDatabase
{
```

`@group database` means this test requires a configured database.

`@group mysql` means this test is mysql only.


## Running Unit Tests ##

You start the whole test suite with default configuration (mysql, root,
non-password) by just calling `./vendor/bin/phpunit` in the Propel directory.

### Start only one test case ###

You can pass a file path to `phpunit` to execute only all tests within this file.

```bash
./vendor/bin/phpunit tests/Propel/Tests/Runtime/ActiveQuery/CriteriaTest.php
```

### Use custom configuration/credentials ###

You can use again environment variables to tell propel's test suite which database credentials it should use
 to build the fixtures and thus all connections.

Examples:

```bash
DB=pgsql DB_HOSTNAME=192.168.0.2 DB_USER=postgres DB_PW=topsecret ./vendor/bin/phpunit
DB=sqlite ./vendor/bin/phpunit
DB=mysql DB_HOSTNAME=192.168.0.2 DB_USER=nonroot DB_PW=topsecret ./vendor/bin/phpunit
```

The best way to execute the test suite is to use our phpunit scripts:

```bash
./tests/bin/phpunit.agnostic.sh # non database tests only
DB_USER=postgres DB_PW=secret ./tests/bin/phpunit.pgsql.sh
./tests/bin/phpunit.mysql.sh
./tests/bin/phpunit.sqlite.sh
```

These scripts already filter by the required database.

### Rebuild fixtures ###

Since we're creating or updating fixtures based on your environment configuration you don't have to call something
 special anymore to build those fixtures by hand. But if you change some schemas you still need a way to tell
 the test suite it has to re-build the whole thing. To do so just remove the file `tests/Fixtures/fixtures_built`
 where we store our current used test suite configuration.

```bash
rm tests/Fixtures/fixtures_built
```

If you change your credentials or adapter it will be automatically detected and triggers a rebuild as well.

### Specific tests only ###

Since we're using test grouping we have some tests that work only with a configured database and tests
that work only with mysql.

To filter by those groups you can pass `--group` to the phpunit command.

Only Non-MySQL database specific tests:

```bash
./vendor/bin/phpunit --group database --exclude-group mysql;
```

Only database specific tests. Won't work for SQLite and PostgreSQL since phpunit includes here mysql tests as well,
which will naturally fail on a non-mysql database:

```bash
./vendor/bin/phpunit --group database;
```

Only non-database specific tests:

```bash
./vendor/bin/phpunit --exclude-group database;
```

See [Using the correct test group](#using-the-correct-testcase-class) at this page
to get more information about this.

### More Tips ###

```bash
phpunit --stop-on-failure
```

Stops on first failure so you don't have to wait until the whole test suite is done.

```bash
phpunit --group test
```

Execute only a subset of test methods that have `@group test` annotation. Example:

```php
/**
 * This tests tests the whole world.
 *
 * @group test
*/
public function testWholeWorld(){}
```

This is very handy if you're trying to fix or write one particular set of test methods and need to execute it
again and again.


## Fix Coding Style ##

You can fix your coding style __before creating your commit__ using [PHP_CodeSniffer](https://github.com/squizlabs/PHP_CodeSniffer)
and the provided project configuration.

    $ ./vendor/bin/phpcs -p -s --standard=config/phpcs.xml $(git ls-files -m)
    
CodeSniffer will try to fix errors when calling 

    $ ./vendor/bin/phpcbf -p -s --standard=config/phpcs.xml $(git ls-files -m)
