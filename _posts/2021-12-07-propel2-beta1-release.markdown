---
layout: post
title: Propel2 Beta 1 Release
published: true
---

With this release, we continue version cleanups aiming API stabilization.

Many thanks to our contributors!

## 2.0.0-beta1

Here are the changes:

- PHP 8.1 compatibility
- Fixes for PHP 7.4 preloading
- Fixed usage of default on-update and on-delete behavior
- Show names of uncommitted migrations
- BehaviorLocator now looks in dev packages, as well
- Aggregate multiple columns behavior & parameter list support
- Fixes around aliases and cross joins and subqueries
- Added support for `keytype` in the magic import/export methods
- PSR naming fixes for variables and methods
- Reset partial flag when populating a relation
- Added `exists` operator
- Escape quotes in behavior
- Quote primary table name if identifier quoting is enabled
- Formats insert `DATE` values as `Y-m-d` instead of `Y-m-d H:i:s.u`
- Allow default-value for concrete-inheritance to be instantiable
- Pluralize `Box` to `Boxes`
- Allow `NO ACTION` for foreign key references (in the dtd/xsd)
- Use object-equality instead of reference-equality to compare object properties
- Generates data dictionary documentation
- PHPStan related code cleanup


### BC breaking impact
Please note that methods have param and return types being added now where they were feasible and ensure better code quality.
Make sure any extensions are updated here regarding their method signature.
Some internal methods were also renamed to fit PSR coding standards.

Due to the support of PHP 7.4 preloading, an update will need the configuration to be rebuilt once by calling `config:convert`, see https://github.com/propelorm/Propel2/wiki/Exception-Target:-Loading-the-database#for-imported-configuration

### Download

You can download this release as usual via Composer. 
Please give it a try and [report any bugs](https://github.com/propelorm/Propel2/issues/new)
you spot:

```json
{
  "require": {
    "propel/propel": "2.0.0-beta1"
  }
}
```

All changes/commits: [github.com/propelorm/Propel2/compare/2.0.0-alpha12...2.0.0-beta1](https://github.com/propelorm/Propel2/compare/2.0.0-beta1...2.0.0-beta2)

All releases: [github.com/propelorm/Propel2/releases](https://github.com/propelorm/Propel2/releases)
