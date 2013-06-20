---
author: vranac
comments: true
date: 2012-01-27 08:47:39+00:00
layout: post
slug: search-and-replace-text-in-files-from-shell
title: Search and replace text in files from shell
wordpress_id: 411
categories:
- Linux
- osx
tags:
- bash
- linux
- osx
- sed
- shell
- terminal
published: true
---

Recently I needed to reuse some files by replacing certain words in them.
I started doing it manually, but as you can imagine it is a tedious, lengthy and error prone process.
There had to be a better way to do it.

As I have learned in recent months, there usually is and is usually connected with shell/terminal :)
<!--more-->

```bash
find . -type f -print0 | xargs -0 -n 1 sed -i -e 's/Foo/Bar/g'
```

What this little nugget does is search the directory you are in (the . after the find, you can specify one or more directories to search in if you need to), for files (the -type f part), and send their path/names to the std output which will be received by sed to edit files in place searching for Foo and replacing it with Bar.


This process will leave backup files that will have an '-e' appended after the extension, while nice thing to have, for my needs and purposes it is junk, so it is better to remove them

```bash
find . -type f -name '*-e' -exec rm -rf {} \;
```

This command will search for all the files that have -e at the end, and will execute rm -rf for them.


And there you have it, contents of the files searched and replaced and junk removed.
