---
comments: true
date: 2011-08-24 07:50:40+00:00
layout: post
slug: custom-repository-with-dic-in-symfony2
title: Custom Repository with DIC in Symfony2
wordpress_id: 334
categories:
- PHP
- Symfony 2
tags:
- DI
- DIC
- IOC
- PHP
- Symfony 2
- Symfony2
published: true
comments: false
---

I am currently working on some Symfony2 bundles, I needed a custom repository to house hold my custom queries, that part is easy with sf2, and quite nicely explained in the [Manual](http://symfony.com/doc/current/book/doctrine.html#custom-repository-classes).
<!--more-->
For brevity, lets setup the CustomRepository

{% codeblock lang:php %}

namespace Code4Hire\NewBundle\Repository;

use Doctrine\ORM\EntityRepository;

class CustomRepository extends EntityRepository
{
}
{% endcodeblock %}

And there you have it, your custom repository that will have access to all the methods offered by default, and you can setup all your custom queries as well.

To use it, in your controller you would use

{% codeblock lang:php %}

$em = $this->getDoctrine()->getEntityManager();
$custom = $em->getRepository('Code4HireNewBundle:Custom')
            ->findAllOrderedByName();
{% endcodeblock %}

Which is fine and dandy, but also fugly as hell.

If only there was a way to make a simpler call...

Symfony2 has [inversion of control](http://en.wikipedia.org/wiki/Inversion_of_control), and [dependency injection](http://en.wikipedia.org/wiki/Dependency_injection), sweet, now I can wire it in and let the framework handle it.

Here is how to do it, in your Resources dir, create/edit the services.xml

{% codeblock lang:xml %}
<?xml version="1.0" ?>

<container xmlns="http://symfony.com/schema/dic/services"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">

    <services>
        <service id="code4_hire_new.entity_manager" alias="doctrine.orm.default_entity_manager" />

        <service id="code4_hire_new.repository.custom"
                factory-service="doctrine.orm.default_entity_manager"
                factory-method="getRepository"
                class="Code4Hire\NewBundle\Repository\Custom" >
            <argument>Code4Hire\NewBundle\Entity\Custom</argument>
        </service>
    </services>

</container>
{% endcodeblock %}

On line 8 the entity manager service is defined, so the DIC knows what it is, and can use it when needed.

On line 10 our custom repository is defined, and this is where the fun happen.

On line 11 we basically say that our repository will use the entity manager we defined on line 8 as factory service.

On line 12, we tell the DIC to call the getRepository method of the factory service we defined on line 11.

On line 13, we tell the DIC to use the our CustomRepository class to resolve the service.

On line 14, we tell the DIC to use the entity Code4Hire\NewBundle\Repository\Custom as a parameter for the factory method.

And finally, we need to modify our entity mapping xml, i'll shorten it for brevity.

{% codeblock lang:xml %}
...
    <entity name="Code4Hire\NewBundle\Entity\Custom" table="custom" repository-class="Code4Hire\NewBundle\Repository\Custom" >
...
{% endcodeblock %}


Note the **repository-class="Code4Hire\NewBundle\Repository\Custom"**, we are explicitly telling the doctrine to use the CustomRepository as the repository class for this entity, this will in turn allow the DIC resolve it properly and to expose all of the custom queries in the CustomRepository class, tip to [Mr. Basic](http://robertbasic.com) for pointing it out


Well, thats quite a lot of things going here, read it a few times and let it sink in, then proceed to the last part.

Time to show what the hubbub is all about.

To call our newly setup custom repository service in dic, all we have to do in our controller is:

``` php
    public function indexAction()
    {
        $customRepository = $this->get('code4_hire_new.repository.custom');

        $customs = $customRepository->callYourCustomMethod();

        return $this->render('Code4HireNewBundle:Default:index.html.twig', array(
            'customs' => $customs
        ));
    }
```

On line 3, the custom repository is called by its ID defined in the services.xml, DIC does its work and your repository is instantiated.
Then you proceed to use it as usual.

This looks much cleaner and nicer doesn't it.

