---
author:
    name: Darryl Hon
    email: docs@linode.com
description: 'How to run a custom Linux distribution on a KVM Linode'
keywords: 'kvm,custom distro,centos,virt-manager,vm-install'
license: '[CC BY-ND 3.0](http://creativecommons.org/licenses/by-nd/3.0/us/)'
contributor:
    name: Darryl Hon
modified: Friday, April 3, 2015
modified_by:
    name: James Stewart
published: 'Saturday, July 17th, 2010'
title: 'Installing a Custom Linux Distribution on a KVM Linode in Place'
---

Linode has a great selection of system images that are ready to get you up and running fast. But sometimes you may need a more customized deployment that might include boot-time encryption, deep-customization; or maybe you just want to roll your own installation. This guide will lead you through deploying a customized CentOS in-place installation on a KVM Linode. While CentOS 6.6 is used in this example, the process outlined here can be applied to most Linux distributions.

{: .note} 
> This guide is for Linodes that have migrated to KVM. For Xen Linodes please refer to our older [Custom Linux Distros on Linode](/docs/tools-reference/custom-kernels-distros/running-a-custom-linux-distro-on-a-linode-vps) guide.

This method takes full advantage of your Linode’s bandwidth and storage speed by performing the installation in place on the Linode, rather than installing and then uploading the system image from your local computer.

{: .caution }
> This method of installation will create a partition table in your Linode disk image, making it incompatible with our [Backup Service](/docs/platform/backup-service). You will also be unable to resize the disk with our Resize tool. If you plan on utilizing Linode backups, follow [this method]() to install your custom distro.

## Before You Begin

1.  You will need a KVM Linode. You can follow instructions [here](/docs/platform/kvm#how-to-enable-kvm) on how to migrate an existing Linode, or set your account to create KVM Linodes.

2.  Disable the [Lassie Shutdown Watchdog](/docs/uptime/monitoring-and-maintaining-your-server#configuring-shutdown-watchdog).

## Create the System Disks
One temporary disk will be needed for the installation media, and as many disks as needed to achieve your preferred system layout. In this example we will use a single disk for the operating system.

1.  Click on the **Dashboard** sub-tab.
2.  Under the **Disks** header, Click on **Create a new Disk**.
3.  Under the **Edit Disk** header, enter **Installation Media** as the name.
4.  Select **ext4** as the type.
5.  Enter **512MB** as the size. Be sure to adjust this value as needed if you are using a larger installation ISO.
6.  Click the **Save Changes** button.

    [![Media Disk](/docs/assets/snap0004.png)](/docs/assets/snap0004.png)

7.  Repeat the steps above, creating an 8192MB ext4 disk image named **System**. If you plan on using a separate swap disk, you can create it now as well.

## Load the Installation Media

Linodes don't have a CD/DVD drive, so we'll load the distribution installer ISO to the `Installation Media` disk.

1.  Click on the **Rescue** tab.
2.  Make a note of path to disk mappings. In this guide **/dev/sda** is the **Installation Media** disk, and **/dev/sdb** is the **System** disk.
3.  Click on the **Reboot Into Rescue Mode** button.

    [![Rescue Mode](/docs/assets/snap0006.png)](/docs/assets/snap0006.png)

4.  Click on the **Remote Access** tab.
5.  Open the Linode console, accessible from the **Console Access** header, with either the **Launch Lish Ajax Console** or with **Lish via SSH**.

    [![Launch Lish](/docs/assets/snap0007.png)](/docs/assets/snap0007.png)
    
    {: .note}
    > You can learn more about Lish from our [Using the Linode Shell](https://www.linode.com/docs/networking/using-the-linode-shell-lish) guide.
    
6.  Enter the following commands into the Lish Web Console to mount the **System** disk, download the CentOS net install ISO, and copy it to the **Installation Media** disk. The final line is a shutdown command:

        mkdir /mnt/system	
        mount /dev/sdb /mnt/system
        cd /mnt/system
        wget http://less.cogeco.net/CentOS/6.6/isos/x86_64/CentOS-6.6-x86_64-netinstall.iso
        dd if=CentOS-6.6-x86_64-netinstall.iso of=/dev/sda
        shutdown –Ph now

    {: .note }
    > You can find a list of alternate mirrors [here](http://www.centos.org/download/mirrors/).

    [![Lish Commands](/docs/assets/snapresize0009.png)](/docs/assets/snap0009.png)

You have now created your installation medium and powered down your Linode. Leave the Lish console up, as we will need it again soon.

## Create the Configuration Profile
1.  Click on the **Dashboard** tab
2.  Under the **Dashboard** header, Click on **Create a new Configuration Profile**
3.  Enter a friendly name for your profile
4.  Under **Boot Settings**, Select **Direct Disk** as the Kernel
5.  Under **Block Device Assignment**:
    - Set **/dev/sda** to **System**
    - Set **/dev/sdb** to **Installation Media**
6.  Under **root / boot device** select `/dev/sdb`.
6.  Under **Filesystem/Boot Helpers**, set all options to **No**
7.  Click on the **Save Changes** button

	[![Configuration Profile](/docs/assets/snap0013_small.png)](/docs/assets/snap0013.png)

## Begin the Installation

### Over the Console

You will begin installing your custom distribution through the Lish console. Use the directional arrow keys and the tab key to select and navigate.

1.  Go back into the [Lish](/docs/networking/using-the-linode-shell-lish) console.
2.  Enter the command:

        configs
3.  Confirm the Configuration Profile created earlier is present.

    [![Boot Profiles](/docs/assets/snap0017.png)](/docs/assets/snap0017.png)

4.  Select the appropriate profile to boot:

        1. boot 2

5.  In our example the installer boot menu is not properly formatted for the serial console we're viewing it with. When prompted to continue, press the **ESC** key to enter the boot prompt:

    [![Tricky Prompt](/docs/assets/snap0019.png)](/docs/assets/snap0019.png) 

6.  Enter the following options to ensure that the install process continues in the serial console:

        linux console=ttyS0

    {: .caution}
    > 
    >If you accidentally press enter when initially prompted, the process will appear to freeze. This is because the installer defaults to console output on tty0, however the KVM console is at ttyS0. Go to the Linode Dashboard, shutdown the server, then repeat this section from the beginning.

7.  When prompted, use the arrow keys to select your preferred language.

8.  At the **Installation Method** dialog, select **URL**.

    [![NetInstall](/docs/assets/snap0022.png)](/docs/assets/snap0022.png)

9.  At the **Configure TCP/IP** dialog, select **OK** to continue.

10. At the **URL Setup** dialog, enter the web address of the mirror nearest to the server. The list of available mirrors can be viewed  [here](http://www.centos.org/download/mirrors/). For this example, with a Linode in Newark we're using the mirror `http://centos.mirror.nac.net/6.6/os/x86_64/`.

    [![NetInstall Mirror](/docs/assets/snap0025.png)](/docs/assets/snap0025.png)

    The installer then retrieves data from the mirror.

### Enabling the GUI Installer over VNC

CentOS offers the option of continuing the install process in a graphic environment over VNC. Other distributions may not have this option, and you will need to continue the installation process in the Lish console. In the case of CentOS 6.6, This option is required to avoid issues with the text installer over the serial terminal later on.

1.  At the **Would you like to use VNC** dialog, select **Start VNC**.

    [![VNC Option](/docs/assets/snap0027.png)](/docs/assets/snap0027.png)

2.  At the **VNC configuration** prompt, enter a secure but temporary password, then select **OK**.

    {: .caution}
    > Avoid using the No Password option. Bots or someone else can easily interfere with your installation instantly!

- A notice will display the **IP address** of the Linode that should be entered into your preferred VNC client.

    [![VNC Standingby](/docs/assets/custom-distro-vnc-server.png)](/docs/assets/custom-distro-vnc-server.png)

    {: .note}
    > 
    > This guide uses TightVNC as the VNC client, it can be downloaded [here.](http://www.tightvnc.com/download.php) You can use any VNC client you prefer. For OS X we recommend [RealVNC](https://www.realvnc.com/download/viewer/).

3.  Open your VNC viewer, then enter the **IP address** of your Linode **plus a colon and the number 1** into the Remote Host field.

    [![TightVNC Login](/docs/assets/snap0030.png)](/docs/assets/snap0030.png)

    {: .note}
    > 
    > The installer uses the non-default port 1.

4.  Click the **Connect** button, enter your password when prompted.

##  Continuing the Installation with the GUI
The next section of this guide goes over installation options for CentOS 6.6 from the GUI interface. If you are installing a different distribution, proceed accordingly and skip to the [next section](#finalizing-the-configuration-profile). To keep this guide simple we will proceed with the installation defaults and detail only the pertinent options.

[![VNC Install](/docs/assets/snap0032.png)](/docs/assets/snap0032.png)

1.  Proceed through installation until prompted for **Which type of installation you would like?** Select **Replace Existing Linux System(s)** then click the **Next** button.

	[![Storage Installation Type](/docs/assets/snapresize0033.png)](/docs/assets/snap0033.png)

    {: .note }
	> For those wanting to setup DMCrypt and boot-time encryption, do so now and also select Create Custom Layout. You may wish to refer to a guide specifically on full system encryption.
    
2.  Two disks will appear on left panel. The larger disk is the system disk and smaller capacity disk is the installation media created earlier. Select the correct disk, and use the **Right Arrow** button to move it to the right panel. Then click on the **Next** button.

	[![Target Device Selection](/docs/assets/snapresize0034.png)](/docs/assets/snap0034.png)



3.  Select the preferred installation type. If you're unsure, we suggest `Minimal`.

    [![Package Installation Type](/docs/assets/snapresize0037.png)](/docs/assets/snap0037.png)



4.  Click the **Next** button to begin the installation process.

    [![Install Progress](/docs/assets/snapresize0039.png)](/docs/assets/snap0039.png)

5.  Once the installation is complete, click on the **Reboot** button to end your VNC session. A system messages about the shutdown will be output on your Lish Web Console.

	[![Install Progress](/docs/assets/snapresize0040.png)](/docs/assets/snap0040.png)

6.  **Optional** Close the Lish Web Console.

## Finalizing the Configuration Profile

1.  Click on the **Dashboard** sub-tab.

2.  Next to your configuration profile, click on **Edit**.

3.  Under **Block Device Assignment** change the **root / boot device** to `/dev/sda`:

    [![Profile Update](/docs/assets/snap0047.png)](/docs/assets/snap0047.png)

4.  Click on the **Save Changes** button.

5.  Under the **Disks** header, next to the **Installation Media** disk, click **Remove**

    {: .note }
    > If you plan on installing this custom distro again on another Linode, you can use [Linode Images](/docs/platform/linode-images) to save this disk for later use before you remove it.

- Under the **Dashboard** header, below your configuration, click on the **Boot** button.
- If the Lish Web Console is still open you will see the boot process begin shortly, including a brief view of the **GNU GRUB**. (fully usable)

    [![Grub Boot](/docs/assets/snap0050.png)](/docs/assets/snap0050.png)

- If you added a boot-time password, or encrypted the system partition using DMCrypt/LUKs, you will need to use the Lish Web Console to enter your password to finish the boot process.

## Congratulations!
You’ve made it through this guide and successfully deployed a customized installation.

Now is a perfect time to begin [Securing Your Server.](https://www.linode.com/docs/security/securing-your-server)

