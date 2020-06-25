---
layout: post
title: Propel2 Alpha 9 is released
published: true
---

We just released Propel2-alpha9 as a part of stabilization process aiming a stable release.
There is still interest in AR based ORM with rapid development features and therefore demand
on Propel2 development.

The strategy we consider sustainable is evolutionary process rather than revolutionary.
Therefore avoiding heavy BC-breaks, while still evolving and cleaning up internals with
better architecture and patterns. This release will be followed with a few more alpha
releases where we need to get rid of outdated, EOD dependencies and enhanced compatibility.

<!-- more -->

## 2.0.0-alpha9

Here are the changes:

* Added compatibility PHP 7.4

* Added support of PSQL expressions [CURRENT_TIMESTAMP, LOCALTIMESTAMP]

* Allowed Symfony 5 dependency

### BC breaks

* Removed support of PHP5, which is EOL of 1.2019.

### Download

You can download this release as usual via Composer. Please give it a try and [report any bugs](https://github.com/propelorm/Propel2/issues/new)
you spot:

```json
{
  "require": {
    "propel/propel": "2.0.0-alpha9"
  }
}
```

All changes/commits: [github.com/propelorm/Propel2/compare/2.0.0-alpha8...2.0.0-alpha9](https://github.com/propelorm/Propel2/compare/2.0.0-alpha8...2.0.0-alpha9)

All releases: [github.com/propelorm/Propel2/releases](https://github.com/propelorm/Propel2/releases)