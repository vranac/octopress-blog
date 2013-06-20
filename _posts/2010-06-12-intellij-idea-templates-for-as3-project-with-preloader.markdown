---
author: vranac
comments: true
date: 2010-06-12 15:15:34+00:00
layout: post
slug: intellij-idea-templates-for-as3-project-with-preloader
title: IntelliJ Idea templates for AS3 project with preloader
wordpress_id: 224
categories:
- Actionscript
- AS3
- IDE
- IntelliJ Idea
tags:
- Actionscript
- AS3
- Flash
- IDE
- IntelliJ Idea
published: true
---


I've been playing with [IntelliJ Idea Ultimate edition](http://www.jetbrains.com/idea/features/flex_ide.html) for the last couple of days to do as3 development in it.
I really like what I see so far, but to make my life easier, I created two template files, that should give me setup similar to Flash Develop AS3 project with preloader.
<!--more-->
The result is in [GitHub repository](http://github.com/vranac/IntelliJ-Idea-Templates-AS3), and they do require a bit of manual setup

To use the templates download/clone them and copy them to your ** .IntelliJIdea90\config\fileTemplates\ ** on windows it's in ** \users\USERNAME\.IntelliJIdea90\config\fileTemplates\ **
After that when you start Idea in as3 project you will have 2 new templates to create as in the image



[![IntelliJ Idea create new item from template menu](/images/2010-06-12-intellij-idea-templates-for-as3-project-with-preloader/idea-create-new-227x300.png)](/images/2010-06-12-intellij-idea-templates-for-as3-project-with-preloader/idea-create-new.png)



To complete the setup you need to go to Project Structure -> Modules -> Flex Compiler Settings, and set your preloader class as Main Class, and set the
{% codeblock lang:bash %}
-frame=start,Main
{% endcodeblock %}
in "Additional compiler options" (in my case Main is the ActionScript for Preloader generated Class)

**Do not forget to check OFF the Use custom compiler configuration file**




[![IntelliJ Idea preloader setup](/images/2010-06-12-intellij-idea-templates-for-as3-project-with-preloader/idea-preloader-setup-300x166.png)](/images/2010-06-12-intellij-idea-templates-for-as3-project-with-preloader/idea-preloader-setup.png)



