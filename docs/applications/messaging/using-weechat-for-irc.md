---
author:
    name: Linode Community
    email: docs@linode.com
description: 'Use WeeChat and GNU Screen to create and maintain connections to IRC networks'
keywords: 'weechat,irc,oftc,real time,chat'
license: '[CC BY-ND 3.0](http://creativecommons.org/licenses/by-nd/3.0/us/)'
contributor:
  name: Samuel Damashek
modified: Wednesday, August 27, 2014
modified_by: 
    name: 'Alex Fornuto'
published: 'Wednesday, August 27, 2014'
title: 'Using WeeChat for Internet Relay Chat'
external_resources:
 - '[WeeChat Home Page](http://www.weechat.org/)'
 - '[GNU Screen](http://www.gnu.org/software/screen/)'
 - '[Screen for Persistent Terminal Sessions](/docs/networking/ssh/using-gnu-screen-to-manage-persistent-terminal-sessions)'
---

*This is a Linode Community guide. [Write for us](/docs/contribute) and earn $250 per published guide.*

<hr>

**WeeChat** is a terminal-based Internet Relay Chat (IRC) client. WeeChat is written in C, and is intended to be very flexible and extensible. WeeChat has all sorts of plugins written in different languages including Python, Perl, and Ruby.  

Because WeeChat is written in C, it runs on many different platforms including Linux, Unix, BSD, Mac OS X and Windows (in Cygwin). Many users prefer WeeChat over other graphical and terminal-based clients because of its many features and its customizability. One advantage of terminal-based clients over graphical IRC clients is the ability to detach from your WeeChat instance and come back later, locally or remotely, using a terminal multiplexer such as screen or tmux.

WeeChat is usually run in a Linux terminal. It may be ran either on your computer, a Linode instance, or any computer running a supported platform. If you run WeeChat on your Linode, you can access WeeChat at any time from any system simply by connecting via SSH and attaching to your screen or tmux instance. This guide assumes you have read [Using The Terminal](/docs/networking/ssh/using-the-terminal) and [Linux System Administration Basics](/docs/tools-reference/linux-system-administration-basics), along with the [Getting Started Guide](/docs/getting-started/). 


## What is IRC?

Internet Relay Chat (IRC) is a protocol which is used to create IRC "networks", sets of IRC servers which can be connected to using IRC clients. Networks are usually independent. Inside a network, there are many channels which can be joined by users. Usually anybody can create a channel. Channels are usually prefixed with hash signs (**#**), and sometimes contain multiple hash signs to represent different types of channels. Individual users can also chat with each other privately using private messages. Many Linode customers use IRC to get technical help and exchange knowledge.

The official Linode channel is **#linode** on the OFTC network (**irc.oftc.net**). 

On IRC, users are classified by four things:

  * Nickname, a unique user-chosen string which is shown as their handle.
  * Username, a separate string from the nickname which is provided by the user. Does not have to be unique.
  * Host, the IP or hostname from which a user is connecting.
  * Real Name, an optional argument containing your name (spaces are allowed)

A user is often represented as ``nickname!username@host``.

## WeeChat Prerequisites

Before installing WeeChat, we suggest:

  * Completing the instructions in the [Getting Started](/docs/getting-started/) guide.
  * Completing the **Adding a New User** section in the [Securing Your Server](/docs/security/securing-your-server#adding-a-new-user) guide.
  * This guide is written for a non-root user. Commands that require elevated privileges are prefixed with ``sudo``. If you're not familiar with the ``sudo`` command, you can check our [Users and Groups](/docs/tools-reference/linux-users-and-groups) guide.

## Using GNU Screen

GNU Screen allows you to start WeeChat and leave it running, even if you disconnect from your Linode. We recommend running WeeChat in Screen, so our instructions include Screen-specific commands. For more information, see [Using GNU Screen to Manage Persistent Terminal Sessions](/docs/networking/ssh/using-gnu-screen-to-manage-persistent-terminal-sessions)

## Installing WeeChat

Below are instructions for installing WeeChat and Screen on different Operating Systems. Find your Operating System and follow the instructions there.

### Debian 7

1. Add the repository maintained by WeeChat decelopers to ensure the most up-to-date version of WeeChat:

        sudo touch /etc/apt/sources.list.d/weechat.list
        echo "deb http://debian.weechat.org wheezy main" | sudo tee  /etc/apt/sources.list.d/weechat.list


2. Update `apt` and install Weechat:
 
        apt-get update
        pt-get install screen weechat

### Ubuntu:

    apt-get install screen weechat

### Fedora/CentOS: :

    yum install screen weechat

### Arch Linux: :

    pacman -S screen weechat

### Mac OS X (HomeBrew):

    brew update
    brew install screen
    brew install weechat

### Mac OS X (MacPorts):

    port install screen
    port install weechat

### Windows (Cygwin)

1. Install Cygwin.  Ensure that subversion and wget are marked to be included during the installation process

2. Install apt-cyg with the following commands

        svn --force export http://apt-cyg.googlecode.com/svn/trunk/ /bin/
        chmod +x /bin/apt-cyg

3. Install WeeChat using the apt-cyg package

        apt-cyg install weechat



## Running WeeChat

To start WeeChat in a screen on most systems (including Debian 7), run:

    screen weechat-curses

You should now see the WeeChat chat window. If you don't, try running ``screen weechat`` instead of ``screen weechat-curses``.

When you first launch WeeChat, it automatically creates a configuration file in ``~/.weechat``.

## Using WeeChat


### Adding and Connecting to a Server 

To add a server (in this case the OFTC network), you will use the ``/server`` command.

    /server add oftc irc.oftc.net/6697 -ssl -autoconnect

This adds a server named "oftc" with hostname "irc.oftc.net" connecting on port 6697. WeeChat will connect using SSL and will automatically connect when you start WeeChat. Once you define the server, you can run:

    /connect oftc

This will tell WeeChat to connect to the server you just set up.

To disconnect, run:

    /disconnect oftc


### Joining and Parting Channels


To join a channel, run:

    /join channel

For example, ``/join #linode``.

Make sure to run join/part commands in the proper server window. You can use ``Alt+x`` to switch server windows.

To part a channel, run:

    /part channel

For example, ``/part #linode``.

### Switching Channels/Buffers

If you have mouse support enabled and also have installed buffers.pl (see the WeeChat Commands section below), then you can simply click on buffers you have joined then type messages in the bottom bar. ``Return`` will submit your message.

Otherwise, you can use ``/buffer x`` to switch between buffers by number or name. ``/buffer 1`` will switch to buffer 1, while ``/buffer #linode`` will switch to the #linode buffer.
You can also press ``Alt-x`` (``Esc-x`` on a Mac), where x is 1-9, to switch to that buffer number. ``Alt 4`` (``Esc-4`` on a Mac) will switch to buffer 4.

<!--- My friend tells me that you have to use Esc instead of Alt. For example, Option-4 would give him ¢.
Pressing ``Ctrl-N`` will switch to the next buffer, and ``Ctrl-P`` will switch to the previous buffer. You can also use ``/buffer +1`` to go to the next buffer, or ``/buffer -1`` to go to the previous buffer. 
-->

### Sending Private Messages

To send a private message to a nickname, run:

    /msg nick message

For example, to message someone with the nickname `friend` "Have you heard about Linode?", run:

    /msg friend Have you heard about Linode?

You can also open a buffer for a nickname with ``/query nickname``. This will create a new buffer which you can send and receive messages in to and from a user. For example, ``/query friend`` will open a conversation with "friend".

### Changing your Nickname

To change your nickname after you have connected, run:

    /nick newnickname

Note that this will only work if the new nickname is not already in use.

### Quitting WeeChat

To quit WeeChat completely, run:

    /quit

## Configuring WeeChat

You usually will not have to directly edit any WeeChat configuration files. Most configuration is done through WeeChat commands.

### Installing Plugins

WeeChat has a plugins system which allows you to install different modifications to WeeChat for different use cases and user preference. In WeeChat versions 0.3.9 and above, a script management system is included. ``/script`` will open up a list of available and installed scripts. From there, you can follow the instructions to install scripts interactively, or install a script using ``/script install <script name>``.


### WeeChat Commands

All WeeChat commands begin with a **/**. Every channel in WeeChat is a *buffer*. Servers are also buffers. By default, WeeChat does not include a list of buffers, but you may install a plugin which does. The buffers.pl plugin is recommended and displays a list of buffers on the left of the screen. This allows you to see what channels and servers you are in without having to remember special commands. It also shows you buffer numbers if you want to use ``/buffer <x>`` to switch to a buffer.

    /script install buffers.pl

``/mouse enable`` will enable mouse support, which allows you to scroll as well as click buffers to change channels and servers.

A list of basic commands is below.

  {: .table .table-striped }
  | Command    | Description                                                                     |
  | --------   | ------------------------------------------------------------------------------- |
  | `/help`    | Lists commands, if a command is given then shows command usage and description  |
  | `/join`    | Joins a channel                                                                 |
  | `/close`   | Closes a buffer, parting the channel if you are in it                           |
  | `/quit`    | Quit WeeChat                                                                    |
  | `/msg`     | Send a message to a nick (or channel)                                           |
  | `/query`   | Opens a private buffer with a nick                                              |
  | `/ban`     | Ban a user from a channel                                                       |
  | `/unban`   | Unban a user from a channel                                                     |
  | `/kick`    | Kick a user from a channel                                                      |
  | `/kickban` | Kick and ban a user from a channel                                              |
  | `/part`    | Parts a channel but does not close the buffers                                  |
  | `/topic`   | Sets channel topic                                                              |
  | `/whois`   | Shows information about a user                                                  |

### Setting Default Channels

WeeChat uses the ``/set`` command to manipulate WeeChat settings. It allows you to change many different attributes about WeeChat, including appearance and functionality.

You can tell WeeChat to automatically connect to some channels when it connects to a server using the ``irc.server.<server>.autojoin`` setting. This setting should be a comma separated list of channels to join. For example, if I want to join #linode when I connect to the oftc network, I would run:

    /set irc.server.oftc.autojoin "#linode"

Then, whenever I connect to the oftc server, I will automatically join #linode.

### Setting Default Nickname, Username, and Real Name

Setting the default nickname, username, and real name is just as simple. To set your default nickname, run:

    /set irc.server_default.nicks "nickname"

You can also specify backup nicknames in case the one you want is taken when you connect.

    /set irc.server_default.nicks "nickname,othernickname"

Setting the default username:
  
    /set irc.server_default.username "username"

Setting the default real name:

    /set irc.server_default.realname "realname"


## Accessing your WeeChat instance

If you ran WeeChat in a screen as specified above, you have the ability to detach from your WeeChat instance and to reattach later. To detach from the screen, press ``Ctrl-A D``. To reattach to your screen, run ``screen -x``. You can reattach to your screen even if you have logged out from your Linode instance and connected later.