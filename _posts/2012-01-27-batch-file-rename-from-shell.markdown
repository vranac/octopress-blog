---
comments: true
date: 2012-01-27 09:13:19+00:00
layout: post
slug: batch-file-rename-from-shell
title: Batch file rename from shell
wordpress_id: 430
categories:
- Linux
- osx
tags:
- awk
- bash
- linux
- osx
- sed
- sh
- shell
- terminal
published: false
comments: false
---

Recently I needed to reuse some files by renaming them or just parts of the name that have a certain word in it.
I started doing it manually, but as you can imagine it is a tedious, lengthy and error prone process.
There had to be a better way to do it.

As I have learned in recent months, there usually is and is usually connected with shell/terminal :)
<!--more-->

I wanted this to be a two part process where I could first see the proposed changes, and then execute them if needed
```bash
find . -name '*Foo*' | awk '{print("mv "$1" "$1)}' | sed 's/Foo/Bar/2'
```

What this little nugget does is search in the directory for any directory and/or file that has *foo* in it (that is the -name '*foo* part), then uses awk to construct the mv commands, and then uses sed to replace Foo with Bar but only for the second match.
The output would look something like this:

```bash
mv x/Foo x/Bar
mv x/Foo.php x/Bar.php
mv x/y/Foo x/y/Bar
```

If you are happy with the result, you can then execute it by piping it to /bin/sh (or any other shell variant)
```bash
find . -name '*Foo*' | awk '{print("mv "$1" "$1)}' | sed 's/Foo/Bar/2' | /bin/sh
```

And there you have it, files and directories are renamed.

