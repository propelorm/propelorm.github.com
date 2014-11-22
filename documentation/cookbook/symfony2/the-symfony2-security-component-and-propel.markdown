---
layout: documentation
title: The Symfony2 Security Component And Propel
---

# The Symfony2 Security Component And Propel #

If you've started to play with the awesome Symfony2 Security Component, you'll
know that you can configure a *provider* to retrieve your users. Symfony2, and the
PropelBundle provide a propel provider, so you just need to add the following
configuration to your `security.yml` file:

``` yaml
# app/config/security.yml
providers:
    main:
        propel:
            class:    My\AwesomeBundle\Model\User
            property: username
```

Your `User` class have to implement the [UserInterface](https://github.com/symfony/symfony/blob/master/src/Symfony/Component/Security/Core/User/UserInterface.php):

``` php
<?php
// src/My/AwesomeBundle/Model/User.php

use Symfony\Component\Security\Core\User\UserInterface;

class User extends BaseUser implements UserInterface
{
}
```

That's all!

## ACL implementation ##

The *PropelBundle* provides a model-based implementation of the Security
components' interfaces. To make use of this *AuditableAclProvider* you only need
to change your security configuration.

``` yaml
security:
    acl:
        provider: propel.security.acl.provider
```

This will switch the provider to be the *AuditableAclProvider* of the
*PropelBundle*.

The auditing of this provider is set to a sensible default. It will audit all
ACL failures but no success by default. If you also want to audit successful
authorizations, you need to update the auditing of the given ACL accordingly.

After adding the provider, you only need to run the `propel:acl:init` command in
order to get the model generated. If you already got an ACL database, the schema
of the *PropelBundle* is compatible with the default schema of Symfony2.

### Separate database connection for ACL ###

In case you want to use a different database for your ACL than your business
model, you only need to configure this service.

``` yaml
services:
    propel.security.acl.connection:
        class:          Propel\Runtime\Connection\DebugPDO
        factory_class:  Propel\Runtime\Propel
        factory_method: getConnection
        arguments:      [ "acl" ]
```

The *PropelBundle* looks for this service, and if given uses the provided
connection for all ACL related operations. The given argument (`acl` in the
example) is the name of the connection to use, as defined in your runtime
configuration.
