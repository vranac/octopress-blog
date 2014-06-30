---
layout: post
title: "Sublime Text psql build system"
date: 2014-04-11 18:30
comments: true
categories:
    - Sublime Text
    - PostgreSQL
    - automation
keywords: Sublime Text, ST3, postgresql, psql, automation
published: true
---

Today I got to work with [Amazon Redshift][aws-redshift]. First thing I did was go through their docs on how to connect to the server.

I tried [Workbench/J][workbench-j] and it works, it is ugly as hell, and requires Java (not a big problem, but I'd avoid it if I can).
Then I went to the psql and tried it, but damn, that syntax is long before I got to the good part.
<!--more-->
For those that haven't encountered it, it goes along the lines of
{% codeblock lang:bash %}
psql -h your-server-name -U your-username -d your-database-name -p we-could-have-a-port-as-well -c "select count(*) from your_table;"
{% endcodeblock %}

Not too bad, but a whole lot longer than I would have liked, and then there is the case of editing the sql and repeating the process, and before execution it would prompt me for the password. Not a prospect I was looking forward to.

Yes, I know that there is a [.pgpass][pgpass] file, but I do not want my project specific u/p to be in my home directory

Being a ~~lazy bastard and~~ [Sublime Text][sublime] user, and knowing that it has a nifty thing called build systems, I wondered if I could get psql to play with Sublime.

And surprise, surprise, it actually can.

{% codeblock lang:json %}
{
    "env": { "PGPASSWORD":"password" },
    "cmd": ["path-to-psql", "-h", "server-url", "-U", "user", "-d", "db-name", "-p", "5439", "-f", "$file", "-o", "results.txt"]
}
{% endcodeblock %}

As you remember (hopefully you did not just skim through), I did not want to use the [.pgpass][pgpass] file, so my first order of business was to somehow set the password for the connection.

You actually can set it by using the [Environment Variables][libpq-envars] for PostgreSQL, the variable of interest is *PGPASSWORD*, with it you can preset the password.
Digging through the [Build Systems Reference][build-docs] I learned that you can actually set the environment vars before calling command.

With that set, my next task was to figure out how to use the contents of the file for the execution, that would be the psql -f option, again by checking the [Build Systems Reference][build-docs] I learned about the wonderful [build system variable named $file][build-docs-vars] that sends the full path to the current file.

**What that means in practice is that you need to have the file saved before you can execute build on it.**

OK, so now we got the input sorted out, what to do with the output, psql has an -o option that allows the results of the query to be sent to the file specified.
I chose the oh so imaginative name of results.txt. That means that the results.txt will be created in the current working directory.

So with all of this put together you end up with [psql-build.sublime-build][repo] file.

OK, now what?

Well, the idea is that you take this build system, put it in your project directory (possibly give it some meaningful name), edit it for your credentials, and then...

Create a symlink to your Sublime Text build system location.

Depending on your OS, and Sublime Text version that would be for either in the **Packages** or **Packages/User directory** in your Sublime Text path (I prefer the Packages/User), for OSX, the path would be ~/Library/Application Support/Sublime Text 3/Packages/User

Once you have done that, open a new file in Sublime Text, save it, select the build system from Tools -> Build System, and on mac press cmd+b (or select Tools -> Build), if you setup everything properly, the results.txt should be created, and you can open it to see results.

[{% img http://blog.code4hire.com/images/2014-04-11-sublime-text-psql-build-system/Sublime-psql-build-system.png 'Build System in Action' %}](http://blog.code4hire.com/images/2014-04-11-sublime-text-psql-build-system/Sublime-psql-build-system.png)

Link to the [Github Repo][repo]

[aws-redshift]:http://aws.amazon.com/redshift/
[workbench-j]:http://www.sql-workbench.net/
[pgpass]:http://www.postgresql.org/docs/9.2/static/libpq-pgpass.html
[sublime]:http://www.sublimetext.com/
[libpq-envars]:http://www.postgresql.org/docs/9.2/static/libpq-envars.html
[build-docs]:http://sublime-text-unofficial-documentation.readthedocs.org/en/latest/reference/build_systems.html
[build-docs-vars]:http://sublime-text-unofficial-documentation.readthedocs.org/en/latest/reference/build_systems.html#build-system-variables
[repo]:https://github.com/vranac/Sublime-psql-build-system