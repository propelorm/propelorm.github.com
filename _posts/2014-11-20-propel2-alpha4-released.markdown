---
layout: post
title: Propel2 alpha4 is released
published: true
---

We just released [Propel2 Alpha4](https://github.com/propelorm/Propel2/releases/tag/2.0.0-alpha4) with lot of bug fixes
and some new major features.

<!-- more -->

## What's new in this release


### Configuration System

[Cristiano Cinotti](https://github.com/cristianoc72) did a fantastic job in integrating the Symfony configuration system
into Propel. The `build.properties` file is the past, you can now define Propel's config as yml, php, ini and more.
You can find more information in the [Configuring Propel](http://propelorm.org/documentation/10-configuration.html) documentation.

### PropelBundle 

[Kévin Gomez](https://github.com/K-Phoen) did also a incredible job in updating the PropelBundle so you can use
it with Propel2. You can find more information about the integration of Propel2 in Symfony in the
[Working With Symfony2](http://propelorm.org/documentation/cookbook/symfony2/index.html) documentation.
It's available also in composer via `"propel/propel-bundle": "2.0.0-alpha4"`.

### Init Command

[Marc Scholten](https://github.com/mpscholten) gave us a brilliant [new `init` command](https://github.com/propelorm/Propel2/pull/693)
to bootstrap your Propel project now even faster.

### Bug fixes

And as always we fixed a lot of bugs with this release. Thanks to [all contributors](https://github.com/propelorm/Propel2/compare/2.0.0-alpha3...2.0.0-alpha4)!

## Roadmap

We said in our latest alpha3 release blog post that the next version will be a beta. Unfortunately, we have to correct this.
We're working hard to get things going, but we have still [many open issues](https://github.com/propelorm/Propel2/issues)
that need some investigations first before we are able to provide you a beta. In our [roadmap](https://github.com/propelorm/Propel2/wiki)
you'll find also some new big features we're planning. At the moment we can not provide you any date a beta is realistic,
although we know it's highly requested. If you find the time to investigate some issues or even provide some fixes,
then please don't hesitate - any contribution is highly appreciated by the whole propel community!


## Download

You can download this release as usual via composer. Please give it a try and [report any bugs](https://github.com/propelorm/Propel2/issues/new)
you spot:

```json
{
  "require": {
    "propel/propel": "2.0.0-alpha4"
  }
}
```

All changes/commits: [github.com/propelorm/Propel2/compare/2.0.0-alpha3...2.0.0-alpha4](https://github.com/propelorm/Propel2/compare/2.0.0-alpha3...2.0.0-alpha4)

All releases: [github.com/propelorm/Propel2/releases](https://github.com/propelorm/Propel2/releases)


Thank you guys!