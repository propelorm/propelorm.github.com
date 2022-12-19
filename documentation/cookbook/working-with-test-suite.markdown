---
layout: documentation
title: Working with Propel's Test Suite
---

# Working with Propel's Test Suite #

Propel uses [PHPUnit](http://phpunit.de/) to test the build and runtime frameworks.

You can find the unit test classes and support files in the `tests/` directory.

Tests that need a database run on at least one of MySQL, Postgres or SQLite.

## Setup requirements ##

To run tests, the required PHP libraries need to be installed. Also, most tests will need access to at least one database system (out of MySQL, Postgres or SQLite) through PHP.

### Setup PHP ###

You need to install the following PHP extensions:

* php-mysql
* php-sqlite3
* php-pgsql

Running tests needs more memory than regular PHP scripts. If cli runs out of memory, increase `memory_limit` in your php.ini:

```
memory_limit = 512M
```

Set a default timezone in your php.ini:

```
date.timezone = Europe/Berlin
```

>**Info**All following commands need to be executed in the propel root directory.

### Install Dependencies ###

[Install composer](https://getcomposer.org/download/) and download dependencies with


    $ composer install


### Setup the Database ###

To setup the database, call the script matching your DBMS in `./tests/bin/`:
```
tests/bin
├── setup.mysql.sh
├── setup.pgsql.sh
└── setup.sqlite.sh
```
Call them like any shell script:
```bash
./tests/bin/setup.mysql.sh
```

They will drop and recreate the required databases/schemas (`test`, `second_hand_books`, `contest`, `bookstore_schemas`, `migration`) and build Propel connection and model classes.



For MySQL and Postgres, setup parameters are provided through environment variables:

| Variable    | Description   | Default Value MySQL | Default Value Postgres |
|-------------|---------------|---------------------|------------------------|
| DB_HOSTNAME | Database host | 127.0.0.1           | 127.0.0.1              |
| DB_PORT     | Database port | default port (3306) | default port (5432)    |
| DB_USER     | Username      | root                | postgres               |
| DB_PW       | Password      | none                | none                   |
| DB_NAME     | Database name | test                | postgres               |

In MySQL `DB_NAME` only changes the name of the main database, the other databases (`second_hand_books`, `contest`, `bookstore_schemas`, `migration`) will always be created with default names.

>**INFO** Username and password are used to both create the database and schemas and run the tests. Make sure the user can create and drop databases and schemas.


Example:

```bash
DB_HOSTNAME=192.168.0.3 DB_USER=nonroot DB_PW=topsecret ./tests/bin/setup.mysql.sh
DB_HOSTNAME=192.168.0.3 DB_USER=postgres DB_PW=secret ./tests/bin/setup.pgsql.sh
./tests/bin/setup.sqlite.sh #does not have any configuration ability
```

The SQLite test database is located at `./tests/test.sq3`, running the setup script will delete that file.

### Rebuilt Test Fixtures ###

If you need to rebuild the test fixtures created during setup, delete the file `./tests/Fixtures/fixtures_built` and the classes will be created again next time a test is started.

## Running Tests ##

Run the test suits through the corresponding composer scripts: 

```bash
composer test:agnostic
composer test:mysql
composer test:pgsql
composer test:sqlite
```

Composer passes on all arguments after a double dash (`--`) to phpunit. This allows you to run individual files:

```bash
composer test:mysql -- --stop-on-failure tests/Propel/Tests/Runtime/ActiveQuery/CriteriaTest.php
```


### Agnostic Tests vs Database tests

Simple unit tests do not require a database. These are called agnostic tests and are run on their own. Agnostic tests are not executed when running database tests and vice versa.

Tests are considered agnostic tests unless they are annotated by the `@group database` annotation. If the test is intended for a specific database, add the `mysql` or `pgsql` group:
```php
/**
 * @group database
 * @group mysql
 */
class MyClassTest extends TestCase
{
```

If you run a test class and phpunit reports `No tests executed!`, most likely it is an agnostic test run as a database test or the other way round:
```bash
$ composer test:agnostic -- tests/Propel/Tests/Runtime/ActiveQuery/CriteriaTest.php
Tests started in temp /tmp.
PHPUnit 9.5.26 by Sebastian Bergmann and contributors.

No tests executed!
```


## Writing Tests ##

### How Tests Work ###

If you have not worked with tests before, now is the best time to look at a short introduction.

- Test class names must end with "Test" and extend a TestCase class, i.e. `MyClassTest extends TestCase`.
- Test functions must start with "test", i.e. `testMyClassBehavior()`.
- `setUp()` and `tearDown()` methods run before and after each test function respectively.
- Assertion functions test if an expected value matches the actual outcome (see [available assertions](https://phpunit.readthedocs.io/en/9.5/assertions.html)). 
- [Data providers](https://phpunit.readthedocs.io/en/9.5/writing-tests-for-phpunit.html#data-providers) allow you to run test functions with different input.

Also
- Test cases should not depend on modifications made by a previous test.
- A test function should validate exactly one condition.
- Writing test cases should follow the same guidelines as writing other code. 

### Directory structure

Test classes are located in `./tests/Propel/Tests/`. The precise location should mirror the location of the tested class, i.e. `./src/Propel/Runtime/ActiveQuery/Criteria.php` is tested in `./tests/Propel/Tests/Runtime/ActiveQuery/CriteriaTest.php`.

If you have changed a class rather than created a new one, you most likely will not have to create a new test class, but add tests to the one at the expected location.


## Using the correct TestCase class ##

Depending on the functionality needed, a test class needs to extend the correct TestCase class. These classes will setup and tear down the environment and provide additional functionality.

> **INFO** If the extended TestCase class uses the database, the test class has to be annotated with `@group database`.

Most common classes are:

| Class | needs @database | Description | Used for |
|-------|-----------------|-------------|----------|
| \Propel\Tests\TestCase | no | Most general Propel test case with easy access to driver and platform data. | Agnostic tests without model access. |
| \Propel\Tests\TestCaseFixtures | no | Guarantees that fixtures are available. | Agnostic tests which use models without reading from or writing to database. | 
| \Propel\Tests\TestCaseFixturesDatabase | yes | Same as TestCaseFixtures but also initializes database tables. | Database access outside of model classes. |
| \Propel\Tests\Helpers\Bookstore\BookstoreTestBase | yes | Initializes connection to bookstore test database. | Model classes with database access. 

>**INFO** Always extend the TestCase matching required functionality, as it will reduce execution time.

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

## Automatic code checks ##

If you plan to create a pull request, your code must pass automatic code checks. 

### Stan and Psalm ###

Run static analyzers [stan](https://phpstan.org/) and [psalm](https://psalm.dev/) through composer scripts:
```bash
composer stan
composer psalm
```

### Code style ###

You can fix code style before creating a PR using [PHP_CodeSniffer](https://github.com/squizlabs/PHP_CodeSniffer) and the provided project configuration.

```bash
composer cs-fix # automatically adjusts formatting
composer cs-check # output remaining errors
```

Add `--` to process individual files:
```bash
composer cs-fix -- src/Propel/Generator/
```

Combine with other commands for easy execution, i.e. lint all modified files with:
```bash
composer cs-fix -- $(git ls-files -m)
```

