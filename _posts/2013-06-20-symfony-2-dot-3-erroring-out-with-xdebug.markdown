---
layout: post
title: "Symfony 2.3 erroring out with Xdebug"
date: 2013-06-20 18:08
comments: true
categories:
    - symfony
    - xdebug
    - php
pubished: true
---
When working with Symfony 2.3 there is a chance that xdebug will start erroring out with "Maximum function nesting level of '100' reached, aborting!" pretty quick.
<!--more-->
To solve it, add
```
xdebug.max_nesting_level = 1000
```

into your xdebug.ini file.

It will make your life a whole lot easier...