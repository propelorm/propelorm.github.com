---
layout: post
title: Propel2 Alpha 12 Release
published: true
---

With this release, we continue Alpha-version cleanups aiming API stabilization.

Many thanks to our contributors!

<!-- more -->

## 2.0.0-alpha12

Here are the changes:

* PHP 8 compatibility
* Widening the range of Symfony v4 to 4.0+ (instead of 4.3+)
* Fixed transaction handling when \Throwable is thrown
* Fixed identifierQuoting for Versionable behavior
* Fixed invalid hydration when using mergeWith of criteria with "with" models
* Adds the ability for locking reads, either shared or exclusive
* Updated TableMap generator to add column name map for normalization and performance speedup
* Use temporal formatter in the toArray() generator, fixes the issue of entities wrongly being marked as dirty due to differences in the datetime formatting

### BC breaks

Please note that due to PHP7 + PHP8 versions both able to be supported with this library, the PDO access had to be refactored in a not fully BC way. Instead of directly extending the PHP core classes, we now depend on interface contracts.

If your software has directly extended those in the past, please make sure to adjust your extensions accordingly.

PDOStatement => Propel\Runtime\Connection\StatementInterface
PdoConnection extends PDO implements ConnectionInterface => only implements the latter and proxies to PDO instead.


### Download

You can download this release as usual via Composer. Please give it a try and [report any bugs](https://github.com/propelorm/Propel2/issues/new)
you spot:

```json
{
  "require": {
    "propel/propel": "2.0.0-alpha12"
  }
}
```

All changes/commits: [github.com/propelorm/Propel2/compare/2.0.0-alpha11...2.0.0-alpha12](https://github.com/propelorm/Propel2/compare/2.0.0-alpha11...2.0.0-alpha12)

All releases: [github.com/propelorm/Propel2/releases](https://github.com/propelorm/Propel2/releases)
