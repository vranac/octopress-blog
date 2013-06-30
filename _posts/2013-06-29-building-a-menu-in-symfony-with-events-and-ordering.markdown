---
layout: post
title: "Building a menu in Symfony with events and ordering"
date: 2013-06-30 17:40
comments: true
categories:
    - symfony
    - KnpMenu
    - events
    - php
tags:
    - symfony
    - knpmenu
    - events
published: true
---
Menus are simple things, you have a root element, it has children, their children have children, and so on.
All nice, clean and simple and utterly dull.

Add Knp Menu and bundle to the composer file, run update, go through the [docs][1] and set it up, and be on your way.

But what happens if you need to construct a menu from different bundles and you have no idea how many items there are and their order.
<!--more-->

That is when things start to be just a bit tricky.

Lets assume there are three bundles, Alpha, Users and Scouts, Alpha bundle will be the one that generates the menu, and that following is the menu tree we need to end up with

```
root
    dashboard               <- comes from the alpha bundle
    pages
        alpha pages         <- comes from alpha bundle
        users pages         <- comes from the users bundle
    users
        admins
            list            <- comes from the users bundle
            add new         <- comes from the users bundle
        users
            list            <- comes from the users bundle
            add new         <- comes from the users bundle
            scout users     <- comes from scout bundle
            search scouts   <- comes from scout bundle
            scout jobs      <- comes from scout bundle
    logout
```

Yes, you could just go in and add all the values into your Builder file and be done with it
``` php

$menu = $factory->createItem('root');

// Pages
$pages = $factory->createItem('Pages', array());

$alphaPages = $factory->createItem('Alpha Pages', array('route' => 'list_alpha_pages'));
$pages->addChild($menu);

$usersPages = $factory->createItem('Users Pages', array('route' => 'list_users_pages'));
$pages->addChild($menu);

$menu->addChild($pages);

// Users
$usersRoot = $factory->createItem('Users', array());

$admins = $factory->createItem('Admins', array());
// ... add more children
$usersRoot->addChild($admins);

$users = $factory->createItem('Users', array());
// ... add more children
$usersRoot->addChild($users);

$menu->addChild($usersRoot);

// ... add more children
```

This particular approach would get out of hand pretty fast, if you have lots of menu items, from different parts of the system,
and especially if their appearance depends on other factors (if the bundle is used at all for example).
Soon you would be injecting all kinds of services and doing all kinds of checks and end up with a [Megamoth with Jenga issues].
Yes you could split the code into smaller methods and compose it all, but still it is bad.

Better thing do to is to extent the menu with events, the how to is explained pretty good in the [docs][2], but in my use case it had a shortcoming with ordering, to be perfectly clear, ***this is not a shortcoming of the bundle/lib itself, execution of all the registered event listeners can be guaranteed, but NOT the order of their execution***.

My approach differs slightly from the one described in the [docs][2].
I opted to have a separate menu event type classes per bundle, so they could be as small(or as big) as possible. The event object class on the other hand is same and is defined once (in Alpha bundle) and used everywhere.

First Thing is to create a menu event types class for Alpha bundle, as mentioned previously the Alpha bundle will have the menu builder, so it will need the **CONFIGURE_ROOT** event type defined in it, as well as **CONFIGURE_PAGES** event type (it was developed first, and only pages were inside of it, things changed...)

``` php src/AlphaBundle/Event/MenuEvents.php

namespace AlphaBundle\Event;

final class MenuEvents
{
    /**
     * @var string
     */
    const CONFIGURE_ROOT = 'alpha.menuConfigureRoot';

    /**
     * @var string
     */
    const CONFIGURE_PAGES = 'alpha.menuConfigurePages';
}
```

For Users bundle it would look like

``` php src/UsersBundle/Event/MenuEvents.php

namespace UsersBundle\Event;

final class MenuEvents
{
    /**
     * @var string
     */
    const CONFIGURE_USERS = 'users.menuConfigureUsers';

    /**
     * @var string
     */
    const CONFIGURE_ADMINS = 'users.menuConfigureAdmins';
}
```

The Scouts bundle does not need its menu event type class, because the menu items from it will not have any children items.
If that changes in future it would be easy to add the class and set things up.

Next up is the Event object class, aptly named ConfigureMenuEvent

``` php src/AlphaBundle/Event/ConfigureMenuEvent.php

namespace AlphaBundle\Event;

use Knp\Menu\FactoryInterface;
use Knp\Menu\ItemInterface;
use Symfony\Component\EventDispatcher\Event;

class ConfigureMenuEvent extends Event
{
    private $factory;
    private $menu;

    /**
     * @param \Knp\Menu\FactoryInterface $factory
     * @param \Knp\Menu\ItemInterface $menu
     */
    public function __construct(FactoryInterface $factory, ItemInterface $menu)
    {
        $this->factory = $factory;
        $this->menu = $menu;
    }

    /**
     * @return \Knp\Menu\FactoryInterface
     */
    public function getFactory()
    {
        return $this->factory;
    }

    /**
     * @return \Knp\Menu\ItemInterface
     */
    public function getMenu()
    {
        return $this->menu;
    }
}
```

As you can see pretty identical to the way docs describe it, sans the menu event type constant.

To start the ball rolling and build the menu, you would need to trigger the **CONFIGURE_ROOT** event from your menu builder class.

``` php src/AlphaBundle/Menu/AdminMenuBuilder.php
    private function buildMenu()
    {
        $menu = $this->factory->createItem('root');

        $this->eventDispatcher->dispatch(MenuEvents::CONFIGURE_ROOT, new ConfigureMenuEvent($this->factory, $menu));

        $this->reorderMenuItems($menu);

        return $menu;
    }
```

With this you now have a menu builder emitting events that will need some listeners to pick up.
As described in the docs it would look something like this for the Alpha bundle

``` yml src/AlphaBundle/Resources/config/services.yml
services:
    alpha.MenuEventsListener:
        class: %alpha.MenuEventsListener.Class%
        arguments: [@event_dispatcher]
        tags:
            - { name: kernel.event_listener, event: alpha.menuConfigureRoot, method: onMenuConfigureRoot }
            - { name: kernel.event_listener, event: alpha.menuConfigurePages, method: onMenuConfigurePages }
```

for the Users bundle

``` yml src/UsersBundle/Resources/config/services.yml
services:
    users.MenuEventsListener:
        class: %users.MenuEventsListener.Class%
        arguments: [@event_dispatcher]
        tags:
            - { name: kernel.event_listener, event: alpha.menuConfigureRoot, method: onMenuConfigureRoot }
            - { name: kernel.event_listener, event: users.menuConfigureUsers', method: onMenuConfigureUsers }
            - { name: kernel.event_listener, event: users.menuConfigureAdmins, method: onMenuConfigureAdmins }
            - { name: kernel.event_listener, event: alpha.menuConfigurePages, method: onMenuConfigurePages }
```

and finally for the Scouts bundle

``` yml src/ScoutsBundle/Resources/config/services.yml
services:
    scouts.MenuEventsListener:
        class: %scouts.MenuEventsListener.Class%
        arguments: [@event_dispatcher]
        tags:
            - { name: kernel.event_listener, event: users.menuConfigureUsers', method: onMenuConfigureUsers }
```

The listeners are now registered in the services, the listener class for the Alpha bundle is pretty much what you would expect

``` php src/AlphaBundle/EventsListener/MenuEventsListener.php

namespace AlphaBundle\EventsListener;

use AlphaBundle\Event\MenuEvents;
use AlphaBundle\Event\ConfigureMenuEvent;

class MenuEventsListener
{
    protected $eventDispatcher;

    /**
     * @param FactoryInterface @factory
     */
    public function __construct($eventDispatcher)
    {
        $this->eventDispatcher = $eventDispatcher;
    }

    /**
     * @param AlphaBundle\Events\ConfigureMenuEvent $event
     */
    public function onMenuConfigureRoot($event)
    {
        $parentMenu = $event->getMenu();
        $factory = $event->getFactory();

        $menu = $factory->createItem('Dashboard', array('route' => 'admin'));

        $menu->setExtra('orderNumber', 1);
        $parentMenu->addChild($menu);

        $menu = $factory->createItem('Pages', array('route' => 'list_pages'));
        $menu->setAttribute('id', 'menu_admin_pages');
        $menu->setAttribute('class', 'submenu');

        $this->eventDispatcher->dispatch(MenuEvents::CONFIGURE_PAGES, new ConfigureMenuEvent($factory, $menu));

        $menu->setExtra('orderNumber', 10);
        $parentMenu->addChild($menu);

        $menu = $factory->createItem('Logout', array('route' => '_account_logout'));

        $menu->setExtra('orderNumber', 100);
        $parentMenu->addChild($menu);
    }

    /**
     * @param AlphaBundle\Events\ConfigureMenuEvent $event
     */
    public function onMenuConfigurePages($event)
    {
        $parentMenu = $event->getMenu();
        $factory = $event->getFactory();
        $menuType = $event->getMenuType();

        $menu = $factory->createItem('Alpha Pages', array('route' => 'list_pages'));

        $menu->setExtra('orderNumber', 10);
        $parentMenu->addChild($menu);
    }
}
```

OK, so we got this far, events are being fired, listened to, and handled, this looks pretty much plain vanilla setup as in the [docs][1], so why all this scribbling the dribble then, you might ask, rightly so. Well, note the lines 28, 35 and 37.

While looking around for a possible solution for the ordering of elements, I came across useful functions like [moveToPosition($position)], as useful as it is, I could not use it because I would have to execute it only after all the elements of the menu were assembled. Digging around some more I got to the [reorderChildren($order)] which would take the array of the item names and reorder the menu item in relation to the order of the names in the array. This was promising, but still the problem of passing around the order number remained.

I discovered you can pass extra info with each menu item in Knp Menu. That, there, and then was my A-HA moment. I could use this extra info to pass around the order number I need for each menu item relative to its parent, and when it is all said and done then call the [reorderChildren($order)] which would sort things out properly.

The implementation is on lines 28 and 37. That extra info will be used later to sort the menu items.

Line 35 triggers an event that would collect all the child items for the pages menu item, you can see the setup of those items for this bundle on line 49 in method ***onMenuConfigurePages***.

The listener class for the Users bundle would look like this

``` php src/UsersBundle/EventsListener/MenuEventsListener.php

namespace UsersBundle\EventsListener;

use UsersBundle\Event\MenuEvents;
use AlphaBundle\Event\ConfigureMenuEvent;

class MenuEventsListener
{
    protected $eventDispatcher;

    /**
     * @param FactoryInterface @factory
     */
    public function __construct($eventDispatcher)
    {
        $this->eventDispatcher = $eventDispatcher;
    }

    /**
     * @param AlphaBundle\Events\ConfigureMenuEvent $event
     */
    public function onMenuConfigureRoot($event)
    {
        $parentMenu = $event->getMenu();
        $factory = $event->getFactory();

        $menu = $factory->createItem('Users', array('route' => 'list_users'));
        $menu->setAttribute('id', 'menu_admin_users');
        $menu->setAttribute('class', 'submenu');

        $submenuAdmins = $factory->createItem('Admins', array('route' => 'list_admins'));
        $submenuAdmins->setAttribute('id', 'menu_admin_users_admins');
        $submenuAdmins->setAttribute('class', 'submenu');

        $this->eventDispatcher->dispatch(MenuEvents::CONFIGURE_ADMINS, new ConfigureMenuEvent($factory, $submenuAdmins));

        $submenuAdmins->setExtra('orderNumber', 10);
        $menu->addChild($submenuAdmins);


        $submenuUsers = $factory->createItem('Users', array('route' => 'list_users'));
        $submenuUsers->setAttribute('id', 'menu_admin_users_users');
        $submenuUsers->setAttribute('class', 'submenu');

        $this->eventDispatcher->dispatch(MenuEvents::CONFIGURE_USERS, new ConfigureMenuEvent($factory, $submenuUsers));

        $submenuUsers->setExtra('orderNumber', 20);
        $menu->addChild($submenuUsers);


        $menu->setExtra('orderNumber', 20);
        $parentMenu->addChild($menu);
    }

    /**
     * @param AlphaBundle\Events\ConfigureMenuEvent $event
     */
    public function onMenuConfigureUsers($event)
    {
        $parentMenu = $event->getMenu();
        $factory = $event->getFactory();
        $menuType = $event->getMenuType();

        $menu = $factory->createItem('List Users', array('route' => 'list_users'));

        $menu->setExtra('orderNumber', 10);
        $parentMenu->addChild($menu);

        $menu = $factory->createItem('Add New User', array('route' => 'add_user'));

        $menu->setExtra('orderNumber', 20);
        $parentMenu->addChild($menu);
    }

    /**
     * @param AlphaBundle\Events\ConfigureMenuEvent $event
     */
    public function onMenuConfigureAdmins($event)
    {
        $parentMenu = $event->getMenu();
        $factory = $event->getFactory();
        $menuType = $event->getMenuType();

        $menu = $factory->createItem('List Admins', array('route' => 'list_admins'));

        $menu->setExtra('orderNumber', 10);
        $parentMenu->addChild($menu);

        $menu = $factory->createItem('Add New Admin', array('route' => 'add_admin'));

        $menu->setExtra('orderNumber', 20);
        $parentMenu->addChild($menu);
    }

    /**
     * @param AlphaBundle\Events\ConfigureMenuEvent $event
     */
    public function onMenuConfigurePages($event)
    {
        $parentMenu = $event->getMenu();
        $factory = $event->getFactory();
        $menuType = $event->getMenuType();

        $menu = $factory->createItem('User Pages', array('route' => 'list_pages_user'));

        $menu->setExtra('orderNumber', 20);
        $parentMenu->addChild($menu);
    }

}
```

On lines 34 and 44 events are triggered to collect the Admin and User sub menu items, and those are added in the ***onMenuConfigureAdmins*** and ***onMenuConfigureUsers*** methods respectively.

On line 49 in method ***onMenuConfigurePages*** page menu item children are set.

And finally the listener for the Scouts bundle would look like this

``` php src/ScoutsBundle/EventsListener/MenuEventsListener.php

namespace ScoutsBundle\EventsListener;

use UsersBundle\Event\MenuEvents;
use AlphaBundle\Event\ConfigureMenuEvent;

class MenuEventsListener
{
    protected $eventDispatcher;

    /**
     * @param FactoryInterface @factory
     */
    public function __construct($eventDispatcher)
    {
        $this->eventDispatcher = $eventDispatcher;
    }

    /**
     * @param AlphaBundle\Events\ConfigureMenuEvent $event
     */
    public function onMenuConfigureUsers($event)
    {
        $parentMenu = $event->getMenu();
        $factory = $event->getFactory();
        $menuType = $event->getMenuType();

        $menu = $factory->createItem('Scout Users', array('route' => 'scouts_list_users'));

        $menu->setExtra('orderNumber', 30);
        $parentMenu->addChild($menu);

        $menu = $factory->createItem('Search Scouts', array('route' => 'scouts_search_users'));

        $menu->setExtra('orderNumber', 40);
        $parentMenu->addChild($menu);

        $menu = $factory->createItem('Scout Jobs', array('route' => 'scouts_jobs'));

        $menu->setExtra('orderNumber', 50);
        $parentMenu->addChild($menu);
    }

}
```

In method ***onMenuConfigureUsers*** the children of the users menu item are added, and their order number is set.

You might have noted that I am using increments of 10 for the order number, reasoning is simple, if you need to add a new item in between the existing ones you should have room to, with least surprises in ordering.

With all this in place your menu would be constructed, the items you wanted would be present but the ordering would be questionable for now, because, as previously stated, the only thing you can be sure of is that the events will finish before line 7 in the snippet below, but you still have no idea on their order of execution.


``` php src/AlphaBundle/Menu/AdminMenuBuilder.php
    private function buildMenu()
    {
        $menu = $this->factory->createItem('root');

        $this->eventDispatcher->dispatch(MenuEvents::CONFIGURE_ROOT, new ConfigureMenuEvent($this->factory, $menu));

        $this->reorderMenuItems($menu);

        return $menu;
    }
```

And with that we have reached the climax of this writeup, the reorderMenuItems function executed on line 7, the code of the method would look like this


``` php src/AlphaBundle/Menu/AdminMenuBuilder.php
    public function reorderMenuItems($menu)
    {

        $menuOrderArray = array();

        $addLast = array();

        $alreadyTaken = array();

        foreach ($menu->getChildren() as $key => $menuItem) {

            if ($menuItem->hasChildren()) {
                $this->reorderMenuItems($menuItem);
            }

            $orderNumber = $menuItem->getExtra('orderNumber');

            if ($orderNumber != null) {
                if (!isset($menuOrderArray[$orderNumber])) {
                    $menuOrderArray[$orderNumber] = $menuItem->getName();
                } else {
                    $alreadyTaken[$orderNumber] = $menuItem->getName();
                    // $alreadyTaken[] = array('orderNumber' => $orderNumber, 'name' => $menuItem->getName());
                }
            } else {
                $addLast[] = $menuItem->getName();
            }
        }

        // sort them after first pass
        ksort($menuOrderArray);

        // handle position duplicates
        if (count($alreadyTaken)) {
            foreach ($alreadyTaken as $key => $value) {
                // the ever shifting target
                $keysArray = array_keys($menuOrderArray);

                $position = array_search($key, $keysArray);

                if ($position === false) {
                    continue;
                }

                $menuOrderArray = array_merge(array_slice($menuOrderArray, 0, $position), array($value), array_slice($menuOrderArray, $position));
            }
        }

        // sort them after second pass
        ksort($menuOrderArray);

        // add items without ordernumber to the end
        if (count($addLast)) {
            foreach ($addLast as $key => $value) {
                $menuOrderArray[] = $value;
            }
        }

        if (count($menuOrderArray)) {
            $menu->reorderChildren($menuOrderArray);
        }
    }
```

Line 6 addLast array will hold all the menu items that have no order number, for my needs those would be added to the end of the menu tree, as usual YMMV.

Line 8 alreadyTaken array will hold the items that need to be added, but their spot is already taken. The idea is that in first pass all the menu items without position collision would be setup, and in second pass the position collisions would be resolved.

Line 12 if the menu item has children do a recursion and sort out the child items first before proceeding with ordering.

Lines 18-27 either add to addLast array, alreadyTaken array or set the order position of the element.

Line 37 when the execution gets to this point, you should have the menu items with their order number setup, and possibly have elements in the addLast and alreadyTaken arrays to be processed

Lines 34-47 process the already taken and insert them at their appropriate place in the array by slicing and merging.

Lines 53-57 will process the menu elements that have no order numbers and add them to the end of the tree.

And finally on line 60 the new order array is passed to the Knp Menu to reorder each the items.

This kind of events based menu will allow for greater flexibility as your application grows, and easier menu ordering if/when needed.

For example if the Scout bundle got its own variant of pages, and they needed to appear under the pages menu item, the changes in service.yml would be the following


``` yml src/ScoutsBundle/Resources/config/services.yml
services:
    scouts.MenuEventsListener:
        class: %scouts.MenuEventsListener.Class%
        arguments: [@event_dispatcher]
        tags:
            - { name: kernel.event_listener, event: users.menuConfigureUsers', method: onMenuConfigureUsers }
            - { name: kernel.event_listener, event: alpha.menuConfigurePages, method: onMenuConfigurePages }
```

And the changes for listener are listener for the Scouts bundle would be

``` php src/ScoutsBundle/EventsListener/MenuEventsListener.php

namespace ScoutsBundle\EventsListener;

use UsersBundle\Event\MenuEvents;
use AlphaBundle\Event\ConfigureMenuEvent;

class MenuEventsListener
{
    protected $eventDispatcher;

    /**
     * @param FactoryInterface @factory
     */
    public function __construct($eventDispatcher)
    {
        $this->eventDispatcher = $eventDispatcher;
    }

    /**
     * @param AlphaBundle\Events\ConfigureMenuEvent $event
     */
    public function onMenuConfigureUsers($event)
    {
        $parentMenu = $event->getMenu();
        $factory = $event->getFactory();
        $menuType = $event->getMenuType();

        $menu = $factory->createItem('Scout Users', array('route' => 'scouts_list_users'));

        $menu->setExtra('orderNumber', 30);
        $parentMenu->addChild($menu);

        $menu = $factory->createItem('Search Scouts', array('route' => 'scouts_search_users'));

        $menu->setExtra('orderNumber', 40);
        $parentMenu->addChild($menu);

        $menu = $factory->createItem('Scout Jobs', array('route' => 'scouts_jobs'));

        $menu->setExtra('orderNumber', 50);
        $parentMenu->addChild($menu);
    }

    /**
     * @param AlphaBundle\Events\ConfigureMenuEvent $event
     */
    public function onMenuConfigurePages($event)
    {
        $parentMenu = $event->getMenu();
        $factory = $event->getFactory();
        $menuType = $event->getMenuType();

        $menu = $factory->createItem('Scout Pages', array('route' => 'list_pages_scout'));

        $menu->setExtra('orderNumber', 30);
        $parentMenu->addChild($menu);
    }
}
```
Boom you are done.


[1]:https://github.com/KnpLabs/KnpMenuBundle/blob/master/Resources/doc/index.md
[2]:https://github.com/KnpLabs/KnpMenuBundle/blob/master/Resources/doc/events.md
[Megamoth with Jenga issues]:http://www.codinghorror.com/blog/2012/07/new-programming-jargon.html
[moveToPosition($position)]: https://github.com/KnpLabs/KnpMenu/blob/master/src/Knp/Menu/Util/MenuManipulator.php#L15
[reorderChildren($order)]: https://github.com/KnpLabs/KnpMenu/blob/master/src/Knp/Menu/MenuItem.php#L428
