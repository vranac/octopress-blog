---
author: vranac
comments: true
date: 2010-05-30 18:30:26+00:00
layout: post
slug: ruby-gem-tool-and-windows-7
title: Ruby Gem tool and Windows 7
wordpress_id: 212
categories:
- Rails
- Ruby
- Windows
tags:
- command prompt
- Gem
- Rails
- ROR
- Ruby
- windows 7
published: true
---

While trying to setup the RoR with XAMPP on my windows 7 machine I encoundered an error

{% codeblock lang:bash %}

ERROR:  While executing gem ... (Errno::EEXIST)

{% endcodeblock %}

while trying to run

{% codeblock lang:bash %}

gem install rails --include-dependencies

{% endcodeblock %}


Turns out none of the other gem commands worked.

The solution is simple, you need to run the command prompt with elevated privileges (run as admin) to get it to work.

