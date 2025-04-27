---
layout: post
title:  "Memory scale-up"
date:   2025-04-26 10:26:00 -0400
categories: deephackmode.io update
---
I’ve been planning to increase the memory for some time now, just waiting for the great deal.  

I saw a post in eBay for exactly the same model that my PowerEdge R740 came with.  Before that, I set up an alert in eBay for the specific model.  That’s how I got to know about the post.

The eBay post was titled "Samsung 32GB 2Rx4 PC4-3200AA DDR4 Server RAM Memory M393A4K40EB3-CWEGY" and the price was $39.99 each.

<!-- ![eBay post 1](/assets/images/2025-04-26-memory-scale-up/ebay-1.png "eBay post 1"){: .popup-img }{: width="256" } ![eBay post 2](/assets/images/2025-04-26-memory-scale-up/ebay-2.png "eBay post 2"){: .popup-img }{: width="256" } -->

<figure>
<div class="image-row">
<img class="popup-img" src="/assets/images/2025-04-26-memory-scale-up/ebay-1.png" alt="eBay post 1" title="eBay post 1">
<img class="popup-img" src="/assets/images/2025-04-26-memory-scale-up/ebay-2.png" alt="eBay post 2" title="eBay post 2">
</div>
<figcaption>Screenshots of the actual eBay listing by the seller</figcaption>
</figure> 

I was pretty sure that there was a typo in the part number in the title.  It has `M393A4K40EB3-CWEGY`, but the picture and description has `M393A4K40DB3-CWEGY`, which is what I have in my R740 too.  So, I think this was good to go!

I immediately ordered all of the available 8 x 32GB DIMM RAM sticks!  

And just three days later, the package has arrived.  

<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-04-26-memory-scale-up/dimms-1.jpeg" alt="Container with DIMMs" title="Container with DIMMs">
</div>
<figcaption>The DIMM sticks were neatly placed inside a plastic container.</figcaption>
</figure> 

<figure>
<div class="image-row">
<img class="popup-img" src="/assets/images/2025-04-26-memory-scale-up/dimms-2.jpeg" alt="DIMM Sticks" title="DIMM Sticks" width="256px;">
<img class="popup-img" src="/assets/images/2025-04-26-memory-scale-up/dimms-3.jpeg" alt="DIMM Stick" title="DIMM Stick" width="256px">
</div>
<figcaption>These sticks looked like new and I guess they have not been used yet.</figcaption>
</figure> 

<!-- ![DIMM sticks 1](/assets/images/2025-04-26-memory-scale-up/dimms-1.jpeg "DIMM sticks 1"){: .popup-img }{: width="512" } -->

<!-- ![DIMM sticks 2](/assets/images/2025-04-26-memory-scale-up/dimms-2.jpeg "DIMM sticks 2"){: .popup-img }{: width="256" }

![DIMM sticks 3](/assets/images/2025-04-26-memory-scale-up/dimms-3.jpeg "DIMM sticks 3"){: .popup-img }{: width="256" } 
-->

Didn’t waste any time and quickly started to install them!

I had to shut down all the TKGI foundation VM’s, NSX VM’s and vCenter VM using the steps from [the official Broadcom TKGI doc](https://techdocs.broadcom.com/us/en/vmware-tanzu/standalone-components/tanzu-kubernetes-grid-integrated-edition/1-21/tkgi/shutdown-startup.html){:target="_blank"}.  I then put the ESXi host to maintenance mode and shut it down.

<!--
This is what it looked like when it had only 4 DIMM's (4 x 32GB) installed. 
![128 GB](/assets/images/2025-04-26-memory-scale-up/128gb.jpeg "128 GB"){: .popup-img }{: width="256" }
-->

<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-04-26-memory-scale-up/128gb.jpeg" alt="128 GB" title="128 GB">
</div>
<figcaption>This is what it looked like when it had only 4 DIMM's (4 x 32GB) installed.</figcaption>
</figure> 

Read through the [Dell manual](https://www.dell.com/support/manuals/en-us/poweredge-r740/per740_ism_pub/system-memory-guidelines?guid=guid-35e102bf-db9e-4652-a16b-2b37f8fce553&lang=en-us){:target="_blank"} to know how to properly install the modules.

In the R740, there are 24 memory slots in total.  12 slots per CPU channel.  In each CPU channel, 6 slots have white tabs and 6 have black tabs.  As per the manual, the white ones need to be filled out first and in numerical order.  That means that I just need to fill out all white tabs, which I did.  

<!-- This is now what it looks like with 12 DIMM's installed.
![384 GB](/assets/images/2025-04-26-memory-scale-up/384gb.jpeg "384 GB"){: .popup-img }{: width="256" } -->

<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-04-26-memory-scale-up/384gb.jpeg" alt="384 GB" title="384 GB">
</div>
<figcaption>This is now what it looks like with 12 DIMM's installed..</figcaption>
</figure> 

Powered on the system.  It got stuck in "Loading BIOS drivers...".  Restarted a few times and it kept getting stuck there or at "Configuring Memory...".  I gather that the (Power On Self Test) (POST) never completes.

<figure>
<div class="image-row">
<img class="popup-img" src="/assets/images/2025-04-26-memory-scale-up/dell-boot-up-configuring-memory-done.png" alt="Configuring Memory" title="Configuring Memory" width="256px;">
<img class="popup-img" src="/assets/images/2025-04-26-memory-scale-up/dell-boot-up-loading-bios-drivers.png" alt="Loading BIOS Drivers" title="Loading BIOS Drivers" width="256px">
</div>
<figcaption>Imagine the anxiety building up when the system doesn't want to start up.</figcaption>
</figure> 

<!--![Configuring Memory](/assets/images/2025-04-26-memory-scale-up/dell-boot-up-configuring-memory-done.png "Configuring Memory"){: .popup-img }{: width="256" } ![Loading BIOS Drivers](/assets/images/2025-04-26-memory-scale-up/dell-boot-up-loading-bios-drivers.png "Loading BIOS Drivers"){: .popup-img }{: width="256" }-->


Found a report of this issue and the recommended solution, from a [Dell Community Forum thread](https://www.dell.com/community/en/conversations/poweredge-hardware-general/new-dell-r740-stuck-on-configuring-memory/66fa73059983c060015be57e){:target="_blank"}.



I tried the “flea power drain” method, but the issue persisted.

I then used the “clear NVRAM” method, and the POST completed finally.

I entered the BIOS Setup, and enabled Memory Testing and then restarted the system.

Memory Test completed with no errors!

<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-04-26-memory-scale-up/memory-test-done.png" alt="Memory Test" title="Memory Test">
</div>
<figcaption>No errors! What a relief!</figcaption>
</figure> 
<!--![Memory Test](/assets/images/2025-04-26-memory-scale-up/memory-test-done.png "Memory Test"){: .popup-img }{: width="256" }-->

In iDRAC though, the Inventory doesn’t show the new DIMM’s yet.

I rebooted the iDRAC.

I rebooted the system and entered Lifecycle Manager and ran the Diagnostics Test.

The Memory tests passed!

<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-04-26-memory-scale-up/tests-passed.png" alt="Memory Tests Passed" title="Memory Tests Passed">
</div>
<figcaption>Great to see it passed the tests.</figcaption>
</figure> 
<!--![Memory Tests Passed](/assets/images/2025-04-26-memory-scale-up/tests-passed.png "Memory Tests Passed"){: .popup-img }{: width="256" }-->

Now iDRAC shows the DIMM’s in the Inventory page.  Each DIMM has a detailed info that can be shown by expanding the particular row of that DIMM.  The info here has all the details including dates, serial number, part number and others.

<figure>
<div class="image-row">
<img class="popup-img" src="/assets/images/2025-04-26-memory-scale-up/inventory-dimm-list.png" alt="Inventory list" title="Inventory list" width="256px;">
<img class="popup-img" src="/assets/images/2025-04-26-memory-scale-up/dimm-a3-details.png" alt="Inventory DIMM details" title="Inventory DIMM details" width="256px">
</div>
<figcaption>All the DIMM's are accounted for in the Inventory.</figcaption>
</figure> 
<!-- ![Inventory list](/assets/images/2025-04-26-memory-scale-up/inventory-dimm-list.png "Inventory list"){: .popup-img }{: width="256" } -->

<!-- ![Inventory DIMM details](/assets/images/2025-04-26-memory-scale-up/dimm-a3-details.png "Inventory DIMM details"){: .popup-img }{: width="256" }-->

The Memory page also shows all of them with some details like speed, size and type.

<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-04-26-memory-scale-up/memory-page-1.png" alt="Memory page" title="Memory page">
</div>
<figcaption>iDRAC System->Memory page shows all 12 DIMM's.</figcaption>
</figure> 

<!--![Memory page](/assets/images/2025-04-26-memory-scale-up/memory-page-1.png "Memory page"){: .popup-img }{: width="256" }-->

Booted up ESXi.  The console and UI show the increased memory size!

<figure>
<div class="image-row">
<img class="popup-img" src="/assets/images/2025-04-26-memory-scale-up/esxi-console-382gb.png" alt="ESXi Console" title="ESXi Console" width="256px;">
<img class="popup-img" src="/assets/images/2025-04-26-memory-scale-up/esxi-ui-382gb.png" alt="ESXi UI" title="ESXi UI" width="256px">
</div>
<figcaption>My ESXi Host now with 384GB of RAM!</figcaption>
</figure> 

<!--![ESXi Console](/assets/images/2025-04-26-memory-scale-up/esxi-console-382gb.png "ESXi Console"){: .popup-img }{: width="256" }

![ESXi UI](/assets/images/2025-04-26-memory-scale-up/esxi-ui-382gb.png "ESXi UI"){: .popup-img }{: width="256" }
-->

I think that this is a nice memory upgrade from 128GB to 384GB and that I got them at a reasonable, if not lower price.  I’m happy about it.  I can now try the [Nested](https://williamlam.com/nested-virtualization){:target="_blank"} ESXi setup again at some point.  128GB was not enough for the Nested setup, I found out.








