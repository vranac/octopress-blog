---
author: vranac
comments: true
date: 2013-01-07 22:33:23+00:00
layout: post
slug: pear-packages-installation-under-vagrant-with-puppet
title: Pear Packages installation under Vagrant with Puppet
wordpress_id: 523
categories:
- Linux
- PHP
- Puppet
- Ubuntu
- Vagrant
tags:
- pear
- puppet
- ubuntu
- vagrant
published: true
---

I have been running Vagrant vm with Puppet for the last 4 months for my development work.
Most of the vagrant stuff is pretty straight forward, once you grasp the concepts.
This post will assume that you are familiar with Vagrant and Puppet, if you are not [this](http://docs.vagrantup.com/v1/docs/getting-started/index.html) is a great place to start.

One aspect of my setup was lacking pear packages installation was prone to errors and as a result of that failed dependencies.
More often than not I would get this fun little error message

```bash
err: /Stage[main]//Exec[pear upgrade]/returns: change from notrun to 0 failed: /usr/bin/pear upgrade returned  instead of one of [0] at /tmp/vagrant-puppet/manifests/base.pp:98
```

All other puppet tasks that rely on this one would be skipped due to failed dependencies.

The exec command looks like this:

```ruby
# upgrade pear
exec {"pear upgrade":
  command => "/usr/bin/pear upgrade",
  require => Package['php-pear'],
}
```

The problem arises from the fact that pear upgrade will return a null or '' value if there is nothing to update, and puppet will misinterpret that as an error as you can see in the error message above.

One solutions that I found was to add the possible returned values like this:

```ruby
# upgrade pear
exec {"pear upgrade":
  command => "/usr/bin/pear upgrade",
  require => Package['php-pear'],
  returns => [ 0, '', ' ']
}
```

On line 5 the returned values are defined, 0 is the default one, and I added the empty string and space just in case, that resolved the problem of the pear upgrade.

To keep things simple, pear should be allowed to auto-discover channels and dependencies.

```ruby
# set channels to auto discover
exec { "pear auto_discover" :
  command => "/usr/bin/pear config-set auto_discover 1",
  require => [Package['php-pear']]
}
```

And to wrap up the prerequisites for pear setup the pear update-channels should be executed.
It may not be necessary, but in some cases it was reported that this command would prevent the pear errors during installation of pear packages.

```ruby
exec { "pear update-channels" :
  command => "/usr/bin/pear update-channels",
  require => [Package['php-pear']]
}
```

So we come to the last piece of the puzzle, if you try to reinstall the pear package, you will get the message that the package is already installed and the task will fail.

The solution for this was a tip from [Robert Basic](http://twitter.com/robertbasic).

I need to use the **creates** parameter, that checks if the file exists, and if it does not, the install will be attempted.
So here is how I installed a good portion of tools from [The PHP Quality Assurance Toolchain ](http://phpqatools.org/)

```ruby
exec {"pear install phpunit":
  command => "/usr/bin/pear install --alldeps pear.phpunit.de/PHPUnit",
  creates => '/usr/bin/phpunit',
  require => Exec['pear update-channels']
}

# install phploc
exec {"pear install phploc":
  command => "/usr/bin/pear install --alldeps pear.phpunit.de/phploc",
  creates => '/usr/bin/phploc',
  require => Exec['pear update-channels']
}

# install phpcpd
exec {"pear install phpcpd":
  command => "/usr/bin/pear install --alldeps pear.phpunit.de/phpcpd",
  creates => '/usr/bin/phpcpd',
  require => Exec['pear update-channels']
}

# install phpdcd
exec {"pear install phpdcd":
  command => "/usr/bin/pear install --alldeps pear.phpunit.de/phpdcd-beta",
  creates => '/usr/bin/phpdcd',
  require => Exec['pear update-channels']
}

# install phpcs
exec {"pear install phpcs":
  command => "/usr/bin/pear install --alldeps PHP_CodeSniffer",
  creates => '/usr/bin/phpcs',
  require => Exec['pear update-channels']
}

# install phpdepend
exec {"pear install pdepend":
  command => "/usr/bin/pear install --alldeps pear.pdepend.org/PHP_Depend-beta",
  creates => '/usr/bin/pdepend',
  require => Exec['pear update-channels']
}

# install phpmd
exec {"pear install phpmd":
  command => "/usr/bin/pear install --alldeps pear.phpmd.org/PHP_PMD",
  creates => '/usr/bin/phpmd',
  require => Exec['pear update-channels']
}

# install PHP_CodeBrowser
exec {"pear install PHP_CodeBrowser":
  command => "/usr/bin/pear install --alldeps pear.phpqatools.org/PHP_CodeBrowser",
  creates => '/usr/bin/phpcb',
  require => Exec['pear update-channels']
}
```
