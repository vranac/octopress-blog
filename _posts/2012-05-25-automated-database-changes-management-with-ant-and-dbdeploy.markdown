---
author: vranac
comments: true
date: 2012-05-25 16:20:30+00:00
layout: post
slug: automated-database-changes-management-with-ant-and-dbdeploy
title: Automated database changes management with Ant and dbdeploy
wordpress_id: 458
categories:
- Automation
- Linux
- MySql
- Ubuntu
tags:
- ant
- automation
- dbdeploy
- mysql
published: true
---

In the last couple of months we started using [Ant tool](http://ant.apache.org/) heavily to do server deployments, along with [Jenkins](http://jenkins-ci.org/) server (although not in the role it was intended, but hey, it serves the purpose).
<!--more-->

Most of the things in setup work well and it became a pretty smooth operation, but one big hurdle that remained was the db changes management.

Lack of time prevented me from exploring the options, but as luck would have it, we were attending the awesome [phpDay 2012](http://2012.phpday.it/) conference, and an exellent talk by [Jeremy Coates](https://twitter.com/#!/phpcodemonkey) titled [An introduction to Phing the PHP build system](http://joind.in/talk/view/6392) where on one of the slides two tools were mentioned, [dbdeploy](http://dbdeploy.com/) and [Liquidbase](http://www.liquibase.org/).

I did not hear about them up till then, and they were mentioned briefly as db change automation tools, and those words rang in my ears, I could not believe my luck, so few days after, I started looking into them.

What I wanted to have is a way to incrementaly update the database, or just dump all the alters in, a nice side effect would be to be able to generate one flat deploy file that could be either user for some remote deployment, or submitted to the DBA if needed.

Both tools look excellent, and they seem to do their job well, but in the end I selected the dbdeploy because it was closer to the way we structure our projects and files, and would be less intrusive to our process.

dbdeploy relies on deltas to update the db, each delta is one file that should ideally have one CREATE/ALTER/DROP command and it's undo counterpart, the files should be named in form XXX_something_or_other.sql, where XXX is numeric that identifies the order of execution of the script, with that in mind, it goes along nicely with the way we work (granted naming of directories is different, but it works, for now...).
We would have to adapt a bit to break down the initial db draft into individual alter files, and add their undos, but that is really a non-issue.

The setup consists of

  1. Ant
  2. dbdeploy, which you can get from [here](http://code.google.com/p/dbdeploy/downloads/list)
  3. MySql Connector JDBC driver, which you can get from [here](http://www.mysql.com/downloads/connector/j/), or some other JDBC driver (I am targeting MySql so YMMV)

The tasks/targets are defined in the build.xml file, more on that later, for the moment I'd like to focus on the build.properties file.
This file is not supposed to go into the VCS apart in it's build.properties.dist form (which you should rename and mod for your local setup).
This file will contain some paths and db credentials that vary from project to project and/or server to server.
This file is a standard ant properties file which consits of key/value pairs

build.properties:

```
# Property files contain key/value pairs
#key=value

# Credentials for the database migrations
db.host=localhost
db.port=3306
db.user=root
db.pass=root
db.name=dbdeploy

# Path to mysql
progs.mysql=/usr/bin/mysql

```

This is pretty straight forward stuff, but point of interest is line 12 - path to mysql, you may want to drop this, I used it to execute the flat sql file with mysql, more like a proof that it works than actual use

With that the variable properties are covered and the build.xml file can be built. My personal preference is that it goes in the VCS, but some prefer to keep only the build.xml.dist in the VCS.


build.xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project name="dbDeploy_example" basedir="." default="default">
    <description>This is an ant build.xml file for the example dbdeploy project.</description>

    <!-- Define the timestamp format for the generated files -->
    <tstamp>
        <format property="current.time" pattern="yyyy-MM-dd-HH-mm-ss"/>
    </tstamp>

    <!-- Load our configuration -->
    <property file="./build.properties"/>

    <!-- Define the sources dir -->
    <property name="src" value="."/>

    <!-- Define the path to the dbdeploy dir -->
    <property name="build.dbdeploy.dbdeploy_dir" value="${src}/../dbdeploy"/>

    <!-- Define the path to the deltas/alters dir -->
    <property name="build.dbdeploy.alters_dir" value="${src}/../db/alters"/>

    <!-- Define the path to the deploy flat file dir -->
    <property name="build.dbdeploy.deploy_dir" value="${src}/../db/deploy"/>

    <!-- Define the path to the undo flat file dir -->
    <property name="build.dbdeploy.undo_dir" value="${src}/../db/undo"/>

    <!-- Last change number to apply, useful for preventing the unchecked delta to mess things up -->
    <property name="build.dbdeploy.lastChangeToApply" value="20"/>

    <!-- Define the db driver -->
    <property name="db.driver" value="com.mysql.jdbc.Driver"/>

    <!-- Define the url to the database -->
    <property name="db.url" value="jdbc:mysql://${db.host}:${db.port}/${db.name}"/>

    <!-- Define the target DBMS -->
    <property name="db.dbms" value="mysql"/>

    <!-- Define tha path to the changelog table sql file -->
    <property name="build.dbdeploy.changelog_file" value="${build.dbdeploy.deploy_dir}/scripts/createSchemaVersionTable.mysql.sql"/>

    <!-- these two filenames will contain the generated SQL to do the deploy and roll it back-->
    <property name="build.dbdeploy.deployfile" value="deploy-${current.time}.sql"/>
    <property name="build.dbdeploy.undofile" value="undo-${current.time}.sql"/>

    <property name="use-verbose" value="false"/>

    <!-- Define the classpath for the db driver -->
    <path id="mysql.classpath">
        <fileset dir="${build.dbdeploy.dbdeploy_dir}">
            <include name="mysql*.jar"/>
        </fileset>
    </path>

    <!-- Define the classpath for the dbdeploy -->
    <path id="dbdeploy.classpath">
        <!-- include the dbdeploy-ant jar -->
        <fileset dir="${build.dbdeploy.dbdeploy_dir}">
            <include name="dbdeploy-ant-*.jar"/>
        </fileset>

        <!-- The dbdeploy task also needs the database driver jar on the classpath -->
        <path refid="mysql.classpath"/>
    </path>

    <!-- Declare the dbdeploy task -->
    <taskdef name="dbdeploy" classname="com.dbdeploy.AntTarget" classpathref="dbdeploy.classpath"/>

    <!-- Target to generate the changelog table in the database -->
    <!-- This should be run only the first time db is created (and if it is ever recreated), so ugly, but works -->
    <target name="create-changelog-table">
        <sql driver="${db.driver}" url="${db.url}"
            userid="${db.user}" password="${db.pass}" classpathref="mysql.classpath">
            <fileset file="${build.dbdeploy.changelog_file}"/>
        </sql>
    </target>

    <!-- Target to generate two scripts, one for deploy, the other for rollback to the version specified in the build properties file,
        useful when you want to submit to DBA for review -->
    <target name="dbdeploy-generate-sql">

        <!-- Generate the directories for the deploy and undo files -->
        <mkdir dir="${build.dbdeploy.deploy_dir}" />
        <mkdir dir="${build.dbdeploy.undo_dir}" />

        <!-- generate the deployment scripts -->
        <dbdeploy
               driver="${db.driver}"
               url="${db.url}"
               userid="${db.user}"
               password="${db.pass}"
               dir="${build.dbdeploy.alters_dir}"
               outputfile="${build.dbdeploy.deploy_dir}/${build.dbdeploy.deployfile}"
               undooutputfile="${build.dbdeploy.undo_dir}/${build.dbdeploy.undofile}"
               dbms="${db.dbms}"
               lastChangeToApply="${build.dbdeploy.lastChangeToApply}"
               />
    </target>

    <!-- Target to generate two scripts, one for deploy, the other for rollback, useful when you want to submit to DBA for review -->
    <target name="dbdeploy-generate-sql-all">

        <!-- Generate the directories for the deploy and undo files -->
        <mkdir dir="${build.dbdeploy.deploy_dir}" />
        <mkdir dir="${build.dbdeploy.undo_dir}" />

        <!-- generate the deployment scripts -->
        <dbdeploy
               driver="${db.driver}"
               url="${db.url}"
               userid="${db.user}"
               password="${db.pass}"
               dir="${build.dbdeploy.alters_dir}"
               outputfile="${build.dbdeploy.deploy_dir}/${build.dbdeploy.deployfile}"
               undooutputfile="${build.dbdeploy.undo_dir}/${build.dbdeploy.undofile}"
               dbms="${db.dbms}"
               />
    </target>

    <!-- Target to actually do the migration to the version specified in the build properties file -->
    <target name="dbdeploy-migrate">

        <!-- generate the deployment scripts -->
        <dbdeploy
               driver="${db.driver}"
               url="${db.url}"
               userid="${db.user}"
               password="${db.pass}"
               dir="${build.dbdeploy.alters_dir}"
               dbms="${db.dbms}"
               lastChangeToApply="${build.dbdeploy.lastChangeToApply}"
               />

    </target>


    <!-- Target to actually do the migration to the latest version -->
    <target name="dbdeploy-migrate-all">

        <!-- generate the deployment scripts -->
        <dbdeploy
               driver="${db.driver}"
               url="${db.url}"
               userid="${db.user}"
               password="${db.pass}"
               dir="${build.dbdeploy.alters_dir}"
               dbms="${db.dbms}"
               />

    </target>

    <!-- Target to import the geenrated deploy sql file into db via mysql -->
    <target name="dbdeploy-execute-sql" depends="dbdeploy-generate-sql">
        <!-- execute the SQL - Use mysql command line to avoid trouble with large files or many statements and PDO -->
        <exec
               command="${progs.mysql} -h${db.host} -u${db.user} -p${db.pass} ${db.name} &lt; ${build.dbdeploy.deployfile}"
               dir="${build.dir}"
               checkreturn="true"/>
    </target>
</project>
```

Points of interest:




  * Line 18 - the path to the dbdeploy and MySql JDBC connector jars, because this will be standardized structure it makes more sense to set it here, than in build.properties


  * Lines 20 - 27 - the paths for the deltas, deploy and undo scripts, they are defined relatively to the build.xml file or basedir definition


  * Line 30 - build.dbdeploy.lastChangeToApply key - this is basically a limiter, if you have new deltas you are working on, and do not want them in the db, you can prevent their execution by setting the number to lower than the order number they have.


  * Line 33 - the package for your db driver, to decrease the typos possible when generating a new target


  * Line 36 - the url of the database


  * Line 42 - the path to the changelog table script, it comes with dbdeploy package, and is **IMPORTANT**, dbdeploy uses it to track changes


  * Lines 45 - 46 - definiton for the deploy and undo files, I chose to use timestamp value after the name


  * Lines 51 - 55 - define classpath for the MySql Connector and the location of the jars for it


  * Lines 58 - 66 - define classpath for the dbdeploy and the location of the jars for it


  * Line 69 - declared the dbdeploy task to Ant


  * Lines 73 - 78 - This is the target that actually creates the changelog table in your db, should be used once or when recreating the db. Without this table the dbdeploy will fail


  * Lines 155 - 161 - This is more a proof of concept than it actually works kinda thing, it can be used do import the generated deploy file into the db via shell command



You might notice that the actual targets to execute are doubled, their syntax is almost the same, they differ in one important aspect.
The *-all targets will execute **ALL** the deltas present indiscriminantly.

The targets without the -all will rely on the **build.dbdeploy.lastChangeToApply** from build.properties file to execute the deltas only upto, and including the number given.

This is important to us, because this way we can make sure that only the deltas that are verified as good be executed in the db, or to shoot ourselves in the foot and lose a couple of hours trying to figure out why delta with order number x is not executed while not realising that the build.dbdeploy.lastChangeToApply value y is less than x, **SO BEWARE OF THE LIMITER**

The targets dbdeploy-generate-sql/dbdeploy-generate-sql-all will generate two files (flat sql files), one for deploy on location set by build.dbdeploy.deploy_dir, and one for undo on location set by build.dbdeploy.undo_dir. These files can either be executed on remote server or submitted to your DBA for review, or some other purpose you may need.

The targets dbdeploy-migrate/dbdeploy-migrate-all will update your db schema with the deltas that were not yet executed.

You will notice that I do not mention the undo/rollback process, from what I understand the dbdeploy currently does not support rolling back to a specific script, and it's undo will happen only if the update is broken.
Dbdeploy will assume you will fix the problem by making another delta and executing it on the db.

For our scenarios that is currently a non-issue because we test things locally and on dev server before we deploy to live. Chances of immediate rollback are small, yet still present.

The files are available on [Github](https://github.com/vranac/ant-dbdeploy)

