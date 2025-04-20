---
layout: post
title:  "Rebranding OEMR to Dell"
date:   2025-04-09 17:48:00 -0400
categories: deephackmode.io update
---
So, remember that I mentioned that there was no hint of "Dell" or "Poweredge" during the boot up at all.

The boot up screen only showed this:
![Dell booting up](/assets/images/2025-04-09-rebranding-oemr-to-dell/Dell-booting.png "Dell booting up"){: width="1024" }{: .popup-img }

The iDRAC screen showed this boring gray page:
![iDRAC screen](/assets/images/2025-04-09-rebranding-oemr-to-dell/idrac1.png "iDRAC screen"){: width="1024" }{: .popup-img }

Dell is the "D" in iDRAC, and the literal word "Dell" was removed from the title of the page.

Well, I found a way to rebrand it back to Dell.  Check out the following references:

[Re/De-Branding Dell OEMR 14G Servers](https://www.reddit.com/r/homelab/comments/x7nws8/redebranding_dell_oemr_14g_servers/){:target="_blank"} 

[Debrand a Dell EMC IDPA DP4400 to a PowerEdge R740xd](https://blog.tkrn.io/debrand-a-dell-emc-idpa-dp4400-to-a-poweredge-r740xd/){:target="_blank"}

[DELL PowerEdge OEMR XL Dell Branded ID Module with Windows Embedded OS Support](https://www.dell.com/support/home/en-sr/drivers/driversdetails?driverid=4xhvw&oscode=ubs22&productcode=oth-r740-xl){:target="_blank"}

Downloaded this to a windows machine:
![Dell download](/assets/images/2025-04-09-rebranding-oemr-to-dell/Dell-download.png "Dell download"){: width="1024" }{: .popup-img }

Followed these instructions:
![Dell-installation-instructions](/assets/images/2025-04-09-rebranding-oemr-to-dell/Dell-installation-instructions.png "Dell-installation-instructions"){: width="1024" }{: .popup-img }

I didn’t get a screenshot but saw something like this after around step 11 above (during server booting up).  Image copied from the [tkrn’s blog](https://blog.tkrn.io/debrand-a-dell-emc-idpa-dp4400-to-a-poweredge-r740xd/){:target="_blank"}.

![rebranding-capture](/assets/images/2025-04-09-rebranding-oemr-to-dell/rebranding-capture.png "rebranding-capture"){: width="1024" }{: .popup-img }

Afterwards, it is branded as Dell PowerEdge R740!  Yay! 

Boot up screen now shows Dell.

![Dell-booting-new](/assets/images/2025-04-09-rebranding-oemr-to-dell/Dell-booting-new.png "Dell-booting-new"){: width="1024" }{: .popup-img }

IDRAC screen is now blue, instead of the boring gray!  "Dell" is now seen in the page.

![iDRAC rebranded](/assets/images/2025-04-09-rebranding-oemr-to-dell/idrac-new.png "iDRAC rebranded"){: width="1024" }{: .popup-img }





