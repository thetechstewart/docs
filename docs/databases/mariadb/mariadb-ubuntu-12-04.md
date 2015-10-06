---
author:
    name: Linode Community
    email: docs@linode.com
description: 'Install MariaDb v10.0 in Ubuntu 12.04'
keywords: 'install mariadb, mariadb, Ubuntu mariadb'
license: '[CC BY-ND 3.0](http://creativecommons.org/licenses/by-nd/3.0/us/)'
published: 'Tuesday, October 6th, 2015'
modified_by:
    name: Linode
title: 'Install MariaDb v10.0 in Ubuntu 12.04'
contributor:
    name: Manikandan
    link: https://twitter.com/mani_90
external_resources:
 - '[For More About MariaDB](https://mariadb.com/kb/en)'
---

MariaDB is a database server that drop-in replacement functionality for MySQL. MariaDB is built my some of original authors of MySQL, with assistance from the broader community of Free and Open source software developers. In addition to core functionality of MySQL, MariaDB offers a rich set of feature enhancement including alternate storage engine, server optimization and patches. This guide will help beginners get started with MariaDB v10.0 on a Ubuntu 12.04 using `apt-get` package manager.
{: .note}
>
>The steps required in this guide require root privileges. Be sure to run the steps below as **root** or with the `sudo` prefix. For more information on privileges see our [Users and Groups](/docs/tools-reference/linux-users-and-groups) guide.

## Step 1: Add MariaDB apt-get Repositories

First you need to add MariaDB repositories in your system. Before adding repository make sure you have required package install and also add apt-get key for MariaDB.

Here are the commands to run to add MariaDB repository to your system.
	apt-get install python-software-properties
	apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xcbcb082a1bb943db
	add-apt-repository 'deb http://kartolo.sby.datautama.net.id/mariadb/repo/10.0/ubuntu precise main'

For other Ubuntu version [Click here](https://downloads.mariadb.org/mariadb/repositories/#mirror=kaist&distro=Ubuntu)

## Step 2: Install MariaDB in Ubuntu

After adding MariaDB apt-get repository in your system, use following command to install MariaDB.
	apt-get update
	apt-get install mariadb-server

[![install command for mariadb](/docs/assets/ install_mariadb_small.jpg)]( (/docs/assets/ install_mariadb.jpg)

while installing MariaDB installer will prompt for MariaDB root account password.  Please refer the below screenshot.

[![root password request for mariadb](/docs/assets/ password_prompt_full_small.jpg)]( (/docs/assetspassword_prompt_full.jpg)


## Step 3: Login to MariaDB

After successful installation you can connect MariaDB using following command.
	mysql –uroot –p
# mysql -u root -p 

Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 42
Server version: 10.0.15-MariaDB-1~precise-log mariadb.org binary distribution

Copyright (c) 2000, 2014, Oracle, SkySQL Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]

You can `Start` or `Stop` or `Restart` MariaDB service using following commands
	/etc/init.d/mysql start | stop | restart