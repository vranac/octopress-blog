---
author: vranac
comments: true
date: 2012-02-28 22:04:56+00:00
layout: post
slug: getting-mamp-to-play-nice-with-vhosts
title: Getting MAMP to play nice with vhosts
wordpress_id: 440
categories:
- Apache
- osx
tags:
- apache
- mac
- mamp
- osx
- vhost
published: true
---

Front end developers in the team we are a part of are using MAMP to serve their websites localy.
Aside from that they had virtually no experience setting up vhosts on it.
So I thought I should put this little tutorial to help them out a bit.
<!--more-->

First things first is to get and install MAMP, you can get it from [http://www.mamp.info/en/index.html](http://www.mamp.info/en/index.html)

The free version will suffice, download it, unpack it, run the pkg instaler.
Once the process is finsihed, open the MAMP dir in applications in your finder.
Go into the conf/apache directory, you should see couple of files, but one called httpd.conf is the one that interests us.

Open the httpd.conf with TextEdit (or your favourite editor), and scroll to the end of the file.
Near the end you should find a section that looks like following:

```apache
# Virtual hosts
Include /Applications/MAMP/conf/apache/extra/httpd-vhosts.conf
```

What this line says is that the vhosts definitions are going to be in one file called httpd-vhosts.conf and that one is located deep into the Applications dir.


I prefer to keep my vhosts files in my home directory, inside of directory imaginatively called Sites.
At this point I will assume that you would like something like that as well.
So, lets tell MAMP to look for our vhosts files in the Sites directory.
We do that by adding

```apache
Include /Users/user/Sites/*.conf
```

under the current include line.

It should look something like
```apache
# Virtual hosts
Include /Applications/MAMP/conf/apache/extra/httpd-vhosts.conf
Include /Users/user/Sites/*.conf
```
**Do remember to change the user in the path to your username.**

Now create the Sites directory in your home directory and (re)start MAMP and you are done.
For seting up the vhosts file check out my [post](http://blog.code4hire.com/2011/03/setting-up-virtual-hosts-for-apache-on-ubuntu-for-local-development/) on the subject.

So from now on, if you want to enable virtual host site, you need to make a sitename.conf file and restart MAMP.
And if you want to disable some site, just add .disabled (or some other extension) to the end of file name and restart MAMP.
