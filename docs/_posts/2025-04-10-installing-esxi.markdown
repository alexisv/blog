---
layout: post
title:  "Installing ESXi"
date:   2025-04-10 17:53:00 -0400
categories: deephackmode.io update
---
I have downloaded the ESXi installer from the Broadcom download portal.  I chose to download the custom ISO file for Dell.  The file name was:

```
VMware-VMvisor-Installer-8.0.0.update03-24414501.x86_64-Dell_Customized-A03.iso
```

The installation was straightforward.  It didn't take a lot of time too.

The following is the series of steps and the screens that I captured as the installation progressed to completion.  I used iDRAC's Virtual Console to see the main console of the server and to mount the ISO file as a virtual DVD.

While the server was powered off, I mounted the ISO file as a virtual DVD.

Click Virtual Media from the top row:
![System powered off](/assets/images/2025-04-10-installing-esxi/image-01.png "System powered off"){: width="1024" }{: .popup-img }

Click "Choose File" under "Map CD/DVD":
![Mapping the ISO file](/assets/images/2025-04-10-installing-esxi/image-02.png "Mapping the ISO file"){: width="1024" }{: .popup-img }

Find and open the installer ISO file:
![Opening the ISO file](/assets/images/2025-04-10-installing-esxi/image-03.png "Opening the ISO file"){: width="1024" }{: .popup-img }

Click "Map Device":
![Mapping the Virtual Media](/assets/images/2025-04-10-installing-esxi/image-04.png "Mapping the Virtual Media"){: width="1024" }{: .popup-img }

Click "Boot" from the top row, and then click "Virtual CD/DVD/ISO":
![Mapping the Virtual Media](/assets/images/2025-04-10-installing-esxi/image-05.png "Mapping the Virtual Media"){: width="1024" }{: .popup-img }

Click "Power" from the top row, and then click "Power On System".  The server will boot up and will switch screens a few times as such:
![Dell starting up](/assets/images/2025-04-10-installing-esxi/image-06.png "Dell starting up"){: width="1024" }{: .popup-img }

![Loading ESXi installer 1](/assets/images/2025-04-10-installing-esxi/image-07.png "Loading ESXi installer 1"){: width="1024" }{: .popup-img }

![Loading ESXi installer 2](/assets/images/2025-04-10-installing-esxi/image-08.png "Loading ESXi installer 2"){: width="1024" }{: .popup-img }

Welcome to the VMware ESXi 8.0.3 Installation!  Just press Enter at this screen.
![Welcome to the installation](/assets/images/2025-04-10-installing-esxi/image-09.png "Welcome to the installation"){: width="1024" }{: .popup-img }

Press F11 to accept the EULA and continue.
![Accept EULA](/assets/images/2025-04-10-installing-esxi/image-10.png "Accept EULA"){: width="1024" }{: .popup-img }

Select a disk to install ESXi.
![Select a disk](/assets/images/2025-04-10-installing-esxi/image-11.png "Select a disk"){: width="1024" }{: .popup-img }

It will scan the disk.  Just be patient and wait for it.
![Scan the disk](/assets/images/2025-04-10-installing-esxi/image-12.png "Scan the disk"){: width="1024" }{: .popup-img }

Choose how you want to install ESXi.
![Choose installation](/assets/images/2025-04-10-installing-esxi/image-13.png "Choose installation"){: width="1024" }{: .popup-img }

Choose a keyboard layout.
![Choose keyboard layout](/assets/images/2025-04-10-installing-esxi/image-14.png "Choose keyboard layout"){: width="1024" }{: .popup-img }

Enter a root password of your choice.
![Enter root password](/assets/images/2025-04-10-installing-esxi/image-15.png "Enter root password"){: width="1024" }{: .popup-img }

Press Enter to continue.  Nothing much I can do about this warning.
![Acknowledge warning](/assets/images/2025-04-10-installing-esxi/image-16.png "Acknowledge warning"){: width="1024" }{: .popup-img }

Press F11 to continue installation, and then "watch paint dry".  
![Continue installation](/assets/images/2025-04-10-installing-esxi/image-17.png "Continue installation"){: width="1024" }{: .popup-img }

Actually, it won't take long.  Installation complete!
![Installation complete](/assets/images/2025-04-10-installing-esxi/image-19.png "Installation complete"){: width="1024" }{: .popup-img }

Unmap the Virtual Media device and disconnect the Virtual Media.
![Disconnect DVD](/assets/images/2025-04-10-installing-esxi/image-21.png "Disconnect DVD"){: width="1024" }{: .popup-img }

Press Enter to reboot.
![Reboot](/assets/images/2025-04-10-installing-esxi/image-22.png "Reboot"){: width="1024" }{: .popup-img }

This is what the console looks like when the boot up has completed.
![ESXi ready](/assets/images/2025-04-10-installing-esxi/image-25.png "ESXi ready"){: width="1024" }{: .popup-img }

To manage the host, log on to the ESXi Host Client using your browser.
![ESXi client ready](/assets/images/2025-04-10-installing-esxi/image-26.png "ESXi client ready"){: width="1024" }{: .popup-img }

Log on with the "root" username and its password.  You'll be greeted by this screen.
![ESXi client ready](/assets/images/2025-04-10-installing-esxi/image-27.png "ESXi client ready"){: width="1024" }{: .popup-img }

Switch to the console to customize the Management Network settings.  Press F2, and log in with the root account and password.
![Customize System](/assets/images/2025-04-10-installing-esxi/esxi-customize-log-in.png "Customize System"){: width="1024" }{: .popup-img }

Select "Configure Management Network".
![Configure Management Network](/assets/images/2025-04-10-installing-esxi/esxi-system-customization.png "Configure Management Network"){: width="1024" }{: .popup-img }

Set a static IP and gateway.
![IPV4 Configuration](/assets/images/2025-04-10-installing-esxi/esxi-network-configuration.png "IPV4 Configuration"){: width="1024" }{: .popup-img }

Configure DNS.
![DNS Configuration](/assets/images/2025-04-10-installing-esxi/esxi-dns-configuration.png "DNS Configuration"){: width="1024" }{: .popup-img }

Configure DNS Suffixes.
![DNS Suffixes](/assets/images/2025-04-10-installing-esxi/esxi-dns-suffixes.png "DNS Suffixes"){: width="1024" }{: .popup-img }

Save your changes.
![Confirm settings](/assets/images/2025-04-10-installing-esxi/esxi-confirm-management-network-settings.png "Confirm settings"){: width="1024" }{: .popup-img }

Test settings.
![Test settings](/assets/images/2025-04-10-installing-esxi/esxi-test-management-network.png "Test settings"){: width="1024" }{: .popup-img }

Make sure the test results are all OK!
![Test results](/assets/images/2025-04-10-installing-esxi/esxi-test-network-results.png "Test results"){: width="1024" }{: .popup-img }

Switch to the ESXi Web UI, and configure Time settings.  Click "Host->Manage", and then click "System->Time & date".
![Time settings](/assets/images/2025-04-10-installing-esxi/esxi-time-settings.png "Time settings"){: width="1024" }{: .popup-img }

Assign a valid license for the ESXi product.
![x](/assets/images/2025-04-10-installing-esxi/esxi-assign-license.png "y"){: width="1024" }{: .popup-img }

**Congratulations!  Your ESXi host is now up and running!**
