---
layout: post
title: Propel2 Beta 2 Release
published: true
---

With this release, we continue version cleanups aiming API stabilization.

Many thanks to our contributors!

## 2.0.0-beta2

Here are the changes:

- PHP 8.1 and 8.2 compatibility
- Added strict types to generated code
- Added Symfony 6 support
- Dropped Symfony 3 support. The minimum required Symfony version is 4.4.0 now.
- Added Monolog 2 support
- Added a type safe access to the `StandardServiceContainer` via `Propel::getStandardServiceContainer()` method
- Added support for the `DATETIME` column type
- Moved templates into own root level
- Overall code quality improvements
- Fixed `DatabaseComparator` in order to skip migration creation if table has `skipSql` flag
- Fixed an issue with many-to-many mapping
- Fixed usage of deprecated date format
- Fixed column's time format issue
- Fixed issue with identifier quoting being ignored
- Fixed a debug mode behavior in order to use new connection with it


### BC breaking impact
Please note that all methods have param and return types being added now where they were feasible and ensure better code quality.
Make sure any extensions are updated here regarding their method signature.
TIMESTAMP column type in schema files for the MySql databases now generates column with actual TIMESTAMP type instead of DATETIME as it was previously. Propel diff considers it as a table structure change and generates migration.
As another side effect timestamps are only valid until 2037 (32bit). Make sure to adjust any databuilders or fixtures accordingly.


### Download

You can download this release as usual via Composer. 
Please give it a try and [report any bugs](https://github.com/propelorm/Propel2/issues/new)
you spot:

```json
{
  "require": {
    "propel/propel": "2.0.0-beta2"
  }
}
```

All changes/commits: [github.com/propelorm/Propel2/compare/2.0.0-beta1...2.0.0-beta2](https://github.com/propelorm/Propel2/compare/2.0.0-beta1...2.0.0-beta2)

All releases: [github.com/propelorm/Propel2/releases](https://github.com/propelorm/Propel2/releases)
