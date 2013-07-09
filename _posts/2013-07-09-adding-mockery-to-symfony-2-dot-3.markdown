---
layout: post
title: "Adding Mockery to Symfony 2.3"
date: 2013-07-09 10:27
comments: true
categories:
    - symfony
    - mockery
    - testing
    - phpunit
    - php
tags:
    - symfony
    - mockery
    - testing
    - phpunit
    - php
keywords: symfony, mockery, unit testing
published: true
---

Mockery is an essential part of my toolset. You can get more information about mockery from the [repo][mockeryrepo] itself, or from [nettuts+][nettutsurl].

My current project in Symfony framework, 2.3 LTS, and I need to use mockery to make my life a whole lot easier.
<!--more-->
The process consists of three steps, of which step 3 is least documented.

First you need to add mockery to your composer.json among other libs you will use
``` javascript
    "require": {
        ....

        "mockery/mockery": "dev-master@dev"

        ....
    },
```

Second step is to add the mockery listener to your phpunit.xml in the app dir

``` xml
    <listeners>
        <listener class="\Mockery\Adapter\Phpunit\TestListener"
            file="Mockery/Adapter/Phpunit/TestListener.php">
        </listener>
    </listeners>
```

when you run the composer install, the composer will download the mockery from the repo, and install it in the vendors dir.
The phpunit will know to use the listner, but the project itself have no idea what mockery is at this point, and where to find it.
To fix that in app/autoloader.php before 'return $loader;'

``` php

if (class_exists('PHPUnit_Runner_Version')) {
    set_include_path(get_include_path() . PATH_SEPARATOR . __DIR__.'/../vendor/mockery/mockery/library/');
    require_once('Mockery/Loader.php');
    $mockeryLoader = new \Mockery\Loader;
    $mockeryLoader->register();
}

```

And you are good to go...

[mockeryrepo]:https://github.com/padraic/mockery
[nettutsurl]:http://net.tutsplus.com/tutorials/php/mockery-a-better-way/
