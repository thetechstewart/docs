---
author:
  name: James Stewart
  email: jstewart@linode.com
description: 'Installing McMyAdmin for Minecraft on Debian 7.'
keywords: 'minecraft,mcmyadmin,debian'
license: '[CC BY-ND 3.0](http://creativecommons.org/licenses/by-nd/3.0/us/)'
modified: Thusday, February 5th, 2015
modified_by:
  name: James Stewart
published: 'Thursday, February 5th, 2015'
title: Installing McMyAdmin for Minecraft on Debian 7
external_resources:
 - '[McMyAdmin Home Page](http://mcmyadmin.com/#/home)'
---

McMyAdmin is a leading control panel for Minecraft servers. It boasts compatibility with third party mods and a sleek web interface for managing your server. This guide covers the installation and configuration of a new McMyAdmin server on an up-to-date, Debian 7 Linode. If you have not followed our [Getting Started](/docs/getting-started/) guide, it's recommended that you do so prior to following this tutorial.

{: .note }
> To play on a Minecraft server you must also have a version of the game client from [Minecraft.net](https://minecraft.net/).

##Prerequisite software

1.  Update the operating system:

		sudo apt-get update && sudo apt-get upgrade

2.  Install the Java Runtime Environment:

		sudo apt-get install openjdk-7-jre

3.  If you have configured the firewall according to our [Securing Your Server](/docs/security/securing-your-server#creating-a-firewall) guide, ensure that a port is open for accessing the McMyAdmin web interface.
		
		 sudo iptables -A INPUT -p tcp --dport 8080 -j ACCEPT

    {: .note}
    > To ensure this rule is persistent through server reboots, be sure to modify your `/etc/iptables.firewall.rules` file.

##Install Mono

1.  Switch to the installation directory:

		cd /usr/local

2.  Download and extract the extra required files from McMyAdmin's website:

		sudo wget http://mcmyadmin.com/Downloads/etc.zip
		sudo unzip etc.zip; sudo rm etc.zip

##Install McMyAdmin

1.  Change to the McMyAdmin installation directory:

		mkdir ~/mcmyadmin
		cd ~/mcmyadmin

2.  Download McMyAdmin. Be sure to check the [Download](https://www.mcmyadmin.com/#/download) page for the latest version:

		wget http://mcmyadmin.com/Downloads/MCMA2_glibc26_2.zip

3.  Extract the downloaded zip file.

		unzip MCMA2_glibc26_2.zip

4.  Start the initial configuration of McMyAdmin.  Replace `PASSWORD` with the admin password for your McMyAdmin web interface:

		./MCMA2_Linux_x86_64 -setpass PASSWORD -configonly

##Running McMyAdmin

1.  Install screen, if it is not already installed:

		sudo apt-get install screen

2.  Start a screen session for the McMyAdmin client:

		screen -S mcma

3.  Check the current working directory, then execute the McMyAdmin server:

		cd ~/McMyAdmin; ./MCMA2_Linux_x86_64

##Managing your Minecraft Server

1.  Browse to the McMyAdmin web interface by visiting `http://YourLinodeIP:8080`.

	[![Login Page](/docs/assets/mcma-login-resize.png)](/docs/assets/mcma-login.png)

2.  Log in with the username `admin` and the password that you provided in the installation step.
	
	[![Configuration Page](/docs/assets/mcma-config-resize.png)](/docs/assets/mcma-config.png)

3.  Once the initial configuration steps are completed,  select your settings, then switch to the status page.
	
	[![Status Page](/docs/assets/mcma-status-resize.png)](/docs/assets/mcma-status.png)

4.  Select "Start Server"

	[![Server Started](/docs/assets/mcma-running-resize.png)](/docs/assets/mcma-running.png)


Congratulations, you have now set up a new Minecraft server using McMyAdmin.