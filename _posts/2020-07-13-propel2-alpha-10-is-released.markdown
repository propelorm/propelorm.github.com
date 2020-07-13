---
layout: post
title: Propel2 Alpha 10 is released
published: true
---

Less than a month passed after the latest Alpha release, and we already happy to introduce a
new Alpha 10, which includes a full support of Symfony 5 components,
a first wave of PhpStan code quality fixes and a bunch of bugfixes.

You can consider this release as a good/safe option for dependent project migrations, as it contains the
longest dependency list, including EOL components (which will be reduced in the next releases).

See you soon!


<!-- more -->

## 2.0.0-alpha10

Here are the changes:

* Full support with Symfony components 2.7+, 3.3+, 4.0+, 5.0+
* Adds support for MYSQL_ATTR_SSL_VERIFY_SERVER_CERT configuration option
* Fixed an issue where propel reverse breaks with system-versioned tables
* Applies PHPStan Level 1 Fixes, which corrects many warnings, type checks, missing types, and incorrect annotations.

### BC breaks

* We don't expect any. Please create a new issue on guthub if you are facing any problems.

### Download

You can download this release as usual via Composer. Please give it a try and [report any bugs](https://github.com/propelorm/Propel2/issues/new)
you spot:

```json
{
  "require": {
    "propel/propel": "2.0.0-alpha10"
  }
}
```

All changes/commits: [github.com/propelorm/Propel2/compare/2.0.0-alpha9...2.0.0-alpha10](https://github.com/propelorm/Propel2/compare/2.0.0-alpha9...2.0.0-alpha10)

All releases: [github.com/propelorm/Propel2/releases](https://github.com/propelorm/Propel2/releases)