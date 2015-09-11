---
author:
    name: Linode
    email: docs@linode.com
description: 'Install a Salt Master and Salt Minions.'
keywords: 'salt, saltstack, install, beginner, Debian 8'
license: '[CC BY-ND 3.0](http://creativecommons.org/licenses/by-nd/3.0/us/)'
modified: Thursday, July 2nd, 2015
modified_by:
    name: Linode
published: 'Thursday, July 2nd, 2015'
title: Install Salt
---

Salt is designed for server management. A single Salt Master controls many Salt Minions. The directions below will configure two Debian 8 Linodes, one as a Salt Master, and one as a Salt Minion.

##Install a Salt Master and a Salt Minion

1.  <a href="http://docs.saltstack.com/en/latest/ref/configuration/nonroot.html" target="_blank">As the root user</a> log into both Linode 1 and Linode 2, then <a href="https://www.linode.com/docs/getting-started#setting-the-hostname" target="_blank">set the hostnames</a>. Without changing the configurations in Salt, the Salt Master's ID and Salt Minions' IDs default to the hostname. 

2.  On both Linodes, create and open `/etc/apt/sources.list.d/salt.list`, then add the following lines: 
    
	{:.file }
	/etc/apt/sources.list.d/salt.list
	:  ~~~  
	   # salt
	   deb http://debian.saltstack.com/debian jessie-saltstack main
	   ~~~

3.  On both Linodes, use `wget` to add the repository key:
	
		wget -q -O- "http://debian.saltstack.com/debian-salt-team-joehealy.gpg.key" | apt-key add -

4.  On both Linodes, run the update command:

        apt-get update

###Installing and Configuring the Salt Master

1.  On Linode 1, the Salt Master, install Salt:

        apt-get install salt-master

2.  On Linode 1, open `/etc/salt/master`, uncomment the `#interface:` line, and replace `<master's IP address>` below with the public, Salt Master's IP address:

    {:.file }
    /etc/salt/master 
    :   ~~~  
        # The address of the interface to bind to:
          interface: <master's IP address>
        ~~~

        {: .caution}
    >
    > Ensure that there are two spaces in front of "interface" and a space between the colon, in `interface:`, and the IP address. YAML formatting follows two space nesting.



3.  On Linode 1, restart Salt:

        systemctl restart salt-master

###Installing and Configuring a Salt Minion

1.  On Linode 2, the Salt Minion, install Salt:

        apt-get install salt-minion
    
2.  On Linode 2, open `/etc/salt/minion`, uncomment the `#master: salt` line, and replace "salt" with the IP address of Linode 1, the Salt Master:

    {:.file }
    /etc/salt/minion 
    :   ~~~ 
        # Set the location of the salt master server. If the master server cannot be
        # resolved, then the minion will fail to start. 
          master: <master's IP address>
        ~~~

        {: .caution}
    >
    > Ensure that there are two spaces in front of "master" and a space between the colon, in `master: `, and the IP address. YAML formatting follows two space nesting.


3.  On Linode 2, restart Salt:

        systemctl restart salt-minion

##Using the Salt Master

1.  From Linode 1, list the known Salt Minions linked to the Salt Master:

        salt-key -L

3.  For security purposes, verify the Minions' IDs on both the Salt Master and the Salt Minions. The Minions' IDs are most likely the hostname from their Linode.
        
    On the Salt Master, replace `<hostname or Minion ID>` below and run:

        salt-key -f <hostname or Minion ID>

    On the Salt Minions:

        salt-call key.finger --local


2.  If the IDs have been verified, accept the listed Salt Minions.

    To accept all Minions:    

        salt-key -A

    Or accept an individual Minion. Replace `<hostname or Minion ID>` below and run:

        salt-key -a <hostname or Minion ID>

3.  Check that the accepted Minions are up:

        salt-run manage.up

4.  Ping the Minions, using '*' for all:

        salt '*' test.ping

For possible next steps, continue building a multi-server configuration setup and read more about [configuration management with Salt States](/docs/applications/salt/salt-states-apache-mysql-php-fail2ban).

