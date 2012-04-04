---
layout: documentation
title: The Symfony2 Security Component And Propel
---

# The Symfony2 Security Component And Propel #

If you've started to play with the awesome Symfony2 Security Component, you'll know that you can configure a **provider**
to retrieve your users. Symfony2 has two providers: `in_memory` and `entity`. Unfortunately, no other providers exist.


## The ModelUserProvider ##

Symfony2 lets you to create a custom provider by using a service.
So you can write your own custom provider and the `PropelBundle` provides a dedicated class to ease that: `ModelUserProvider`.

The `ModelUserProvider` takes three arguments:

* A _class name_ which is the Propel class that owns the logic of your users;
* A _proxy class name_ which is required to manage objects that implement the `UserInterface` interface;
* A _property name_ which is the property to retrieve your users (default is: `username`).

Basically, you'll have to declare a service:

{% highlight xml %}
<!-- src/Acme/SecuredBundle/Resources/config/services.xml -->
<?xml version="1.0" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">

    <services>
        <service id="acme.secured.security.provider" class="Acme\SecuredBundle\Security\User\CustomUserProvider" />
    </services>

</container>
{% endhighlight %}

And to create the corresponding class:

{% highlight php %}
<?php
// src/Acme/SecuredBundle/Security/User/CustomUserProvider.php

namespace Acme\SecuredBundle\Security\User;

use Propel\PropelBundle\Security\User\ModelUserProvider;

class CustomUserProvider extends ModelUserProvider
{
    public function __construct()
    {
        parent::__construct('Acme\SecuredBundle\Model\User', 'Acme\SecuredBundle\Proxy\User', 'username');
    }
}
{% endhighlight %}

The _proxy class_ is designed as following:

{% highlight php %}
<?php
// src/Acme/SecuredBundle/Proxy/User.php

namespace Acme\SecuredBundle\Proxy;

use Symfony\Component\Security\Core\User\UserInterface;

use Acme\SecuredBundle\Model\User as ModelUser;

class User implements UserInterface
{
    /**
     * The model user
     *
     * @var \Acme\SecuredBundle\Model\User
     */
    private $user;

    public function __construct(ModelUser $user)
    {
        $this->user = $user;
    }

    /**
     * {@inheritDoc}
     */
    public function getRoles()
    {
        return $this->getUser()->getRoles();
    }

    /**
     * {@inheritDoc}
     */
    public function getPassword()
    {
        return $this->getUser()->getPassword();
    }

    /**
     * {@inheritDoc}
     */
    public function getSalt()
    {
        return $this->getUser()->getSalt();
    }

    /**
     * {@inheritDoc}
     */
    public function getUsername()
    {
        return $this->getUser()->getUsername();
    }

    /**
     * {@inheritDoc}
     */
    public function eraseCredentials()
    {
    }

    /**
     * {@inheritDoc}
     */
    public function equals(UserInterface $user)                                                                                                          {
        return $this->getUser()->equals($user);
    }

    /**
     * @return \Acme\SecuredBundle\Model\User
     */
    protected function getUser()
    {
        return $this->user;
    }
}
{% endhighlight %}

Once done, you'll have to register your new custom provider in the `security.yml` file:

{% highlight yaml %}
# src/Acme/SecuredBundle/Resources/config/security.yml
security:
    # ...
    providers:
        custom_provider:
            id: acme.secured.security.provider
{% endhighlight %}

You now have a working security provider with Propel.

## ACL implementation ##

The `PropelBundle` provides a model-based implementation of the Security components' interfaces.
To make us of this `AuditableAclProvider` you only need to change your security configuration.

{% highlight yaml %}
security:
    acl:
        provider: propel.security.acl.provider
{% endhighlight %}

This will switch the provider to be the `AuditableAclProvider` of the `PropelBundle`.

The auditing of this provider is set to a sensible default. It will audit all ACL failures but no success by default.
If you also want to audit successful authorizations, you need to update the auditing of the given ACL accordingly.

After adding the provider, you only need to run the `propel:init-acl` command in order to get the model generated.
If you already got an ACL database, the schema of the `PropelBundle` is compatible with the default schema of Symfony2.

### Separate database connection for ACL ###

In case you want to use a different database for your ACL than your business model, you only need to configure this service.

{% highlight yaml %}
services:
    propel.security.acl.connection:
        class: PropelPDO
        factory_class: Propel
        factory_method: getConnection
        arguments:
            - "acl"
{% endhighlight %}

The `PropelBundle` looks for this service, and if given uses the provided connection for all ACL related operations.
The given argument (`acl` in the example) is the name of the connection to use, as defined in your runtime configuration.
