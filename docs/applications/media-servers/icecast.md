#What is Icecast?
Icecast is a package  that can be  used for streaming music or anything  else over the Internet.  it is easy to set up and depending on errors and testing of the pages,  will take about 2-3 hours, to get configured. Liquid soap and other automation and encoding tools that can  take advantage of ice cast will not be discussed here, just the basic set up and usage.

##What you will need
to install ice cast you will need the following

1. a linode distribution. I'm using Debian 7.6
2. a way to remotely connect to your linode. I'm using osx 10.x, and the terminal

You will also need a package manager. apt-get should be preinstalled with your linux build.

##Installing icecast.
installing ice cast is a breeze, however you might need  to update your linode  first. To do this:

1. Log in as root.
2. Enter the following 2 commands chained in to one for automation purposes.

~~~~
apt-get update && apt-get upgrade
~~~~
3.  Press enter. The process will start. follow the prompts on screen. The update and upgrade should not take anymore then about 2 minutes to complete.

Once you have this done, and your prompt reappears, enter the following   to install icecast.
~~~~
apt-get install icecast
~~~~
Press enter and follow any on screen prompts. Once ice cast is installed you will need to modify some items in the icecast xml file. The file you are looking for is found in /etc/icecast2/icecast.xml You may either do this locally by downloading the xml to your computer using ftp or scp, or you may do this in a text editor that ships with your linux distribution.

##Configuring the xml
Not all xml entries will be gone over as these are documented  on [icecast's site.](http://en.flossmanuals.net/icecast/logs-linux) Only the key ones will be gone over so that you can have a more user friendly experience.

###password
You will have to change the password  right away so your server will not get hacked. I recommend changing the pass word for at least the main stream, and the admin control panel. Be sure to make the passwords different so that you have less chance of your server getting hacked by a malicious user. The passwords you will want to change if you want maximum security  however are the following:
1. source
2. Relay.
3. Admin

###host name
If you set up your host as documented here, chances are you have a host name. Important:  **do not use this for your ice cast.** You might get failures to connect if you set up an automation package like liquid soap. In stead use the ip found in your panel. 

###logging
You will want to also log what happens in case you or one of your other broadcasters has issues streaming.  Before modifying the xml do the following.

1. From your terminal prompt while still logged in as root, 
1. cd to a separate user account in your home directory. If necessary  add a user using the adduser tool. This insures that a user can diagnose logs and that you don't need to always be logged in as root. More info can be found in the following [linode tutorial.](https://www.linode.com/docs/security/securing-your-server/)
1. type the following two lines of code hitting enter after each line 
	* touch accesslog.log  
	* touch errorlog.lo  g  
1. Modify the log directory to match your /home/username you created in step 2.


Now  in the icecast.xml file modify the following to match these paths.

	<webroot>/etc/icecast2/web</webroot>
	<adminroot>/etc/icecast2/admin</adminroot>
	<!-- <pidfile>/usr/share/icecast2/	icecast.pid</pidfile> -->

Under  the logging section modify the entries to the following paths.  

	<accesslog>access.log</accesslog>
	<errorlog>error.log</errorlog>  
	<!-- <playlistlog>playlist.log</playlistlog> -->  


The play list log is optional as seen by the commenting out of that line, however it is important to modify   the access and error logs for troubleshooting purposes.

###Final steps
Before you can call the xml good, there's one more block of code you will have to modify. After the change owner section under user put your user account you set up. For example if you created a user called IcecastIsFun then the line would be 

	<user>IcecastIsFun</user>


Also, make sure the group is set to root so that ice cast can run successfully. 

###uploading the configuration file
We have spent a lot of time, perhaps hours on this configuration file and we don't want to loose it, right?  so log in via sftp and cd to /etc/icecast2 and upload that xml you worked on. You can leave it as icecast.xml or rename the one that ships with the package to something else, leaving the xml extension and uploading the icecast.xml file you worked on.

##Starting icecast
After you have everything set up correctly you will want to start icecast.  Do the following.

1. cd to /etc/icecast2 and press enter
1. After you changed to /etc/icecast2 run the following command. the -c tells what configuration file to use if you have multiple configuration files. The -b tells icecast to run in the background and start loging to the access.log and error.log files we created earlier in the set up. Here's the command.  
	* icecast2 -c icecast.xml -b
1. Hit enter.

with a lot of luck the icecast application will start and go to the background. You can now log out as root. It is not needed anymore.

#exit
Congratulations.  You have just set up your icecast server. Of corse you need music to stream to it and maybe an automation package such as the flexible [liquidsoap](http://savonet.sourceforge.net/doc-svn/documentation.html), however that will not be covered here. 
