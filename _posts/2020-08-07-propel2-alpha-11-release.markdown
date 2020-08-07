---
layout: post
title: Propel2 Alpha 11 Release
published: true
---

With this release, we continue Alpha-version cleanups aiming API stabilization.
Many thanks to our contributors, who made possible this release to come that fast. 👍

See you soon!


<!-- more -->

## 2.0.0-alpha11

Here are the changes:

* Fixed return value for "no migration needed" case in MigrationMigrateCommand
* Always create unique indices by constraint for Postgres (@daniel-rose)
* Do not try to fetch related objects of a new object (@gharlan)
* Map JSON type to native Postgres type (@tienbuide)
* Fixed nullable docblock for mutator methods (@dereuromark)
* PHP 7.2+ cleanups (class visibility modifiers, native types etc)
* Dropped EOL Symfony 2, Postgres 9.4 from test matrix
* Fixed docblocks and typehinting
* PHPStan level 5 static analyzing
* Yoda notation cleanup


### BC breaks

* We don't expect any. Please create a new issue on guthub if you are facing any problems.

### Download

You can download this release as usual via Composer. Please give it a try and [report any bugs](https://github.com/propelorm/Propel2/issues/new)
you spot:

```json
{
  "require": {
    "propel/propel": "2.0.0-alpha11"
  }
}
```

All changes/commits: [github.com/propelorm/Propel2/compare/2.0.0-alpha10...2.0.0-alpha11](https://github.com/propelorm/Propel2/compare/2.0.0-alpha10...2.0.0-alpha11)

All releases: [github.com/propelorm/Propel2/releases](https://github.com/propelorm/Propel2/releases)