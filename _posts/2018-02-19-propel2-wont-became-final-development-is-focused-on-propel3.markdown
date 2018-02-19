---
layout: post
title: Propel2 won't became final, the development is focused on Propel3
published: true
---

As the [README.md](https://github.com/propelorm/Propel2/blob/master/README.md) and all the [version tags](https://github.com/propelorm/Propel2/releases) are saying it, Propel2 will never be finished.
Since [Propel2](https://github.com/propelorm/Propel2) is already in wide use and to make a clean cut, [Propel3](https://github.com/propelorm/Propel3) is written from scratch with a dedicated repository.

<!-- more -->
The code base as well as the chosen design patterns of the Propel2 project is simple outdated.
To not break the existing Propel2 installations, we have created a [dedicated Propel3](https://github.com/propelorm/Propel3) repository.
Propel3 has replaced the [active record](https://en.wikipedia.org/wiki/Active_record_pattern) pattern with the [data mapper](https://en.wikipedia.org/wiki/Data_mapper_pattern) pattern.
It is using [PHP 7.1](https://secure.php.net/releases/7_1_0.php) as base and as an optional active record pattern support via traits.
Since Propel3 is in a pre-alpha-/not-ready-to-use phase. There is [issue list](https://github.com/propelorm/Propel3/issues) available. Feel free to contribute (via code or via comment) to this.
