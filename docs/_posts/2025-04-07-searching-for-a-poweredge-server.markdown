---
layout: post
title:  "Searching for a PowerEdge server"
date:   2025-04-07 17:48:00 -0400
categories: deephackmode.io update
---
It was Black Friday weekend 2024 when I was searching the Dell website for deals.  Before that, I wasn't sure what stuff to get for myself, and so I thought why not check out datacenter server hardware used for running Hypervisors like ESXi.  That’s how I came across the “PowerEdge” servers.  They are expensive though, like in the few thousands of dollars, and so I thought of searching for a used one instead.

I searched in the Facebook marketplace (FBm) for a “PowerEdge”.  Found one being sold by a guy in Cary, NC.  PowerEdge R720 for $200.  2 cpus, with 128gb ram and no hard drives!  

Did my research.  Found a sub-reddit where homelab stuff is being discussed.  I gather that R720’s are worth nothing any more.  People are just giving them away to get rid of them.  At least, that's what I gathered.

Googled it also and found that it was End-Of-Life (EOL) since 5/25/2018. [See this page.](https://www.topgun-tech.com/end-of-service-life/dell-emc/poweredge/){:target="_blank"} 

There was someone in FBm selling a R730 for $150, but it only has 32GB and no drives.  Also found out that R730’s were EOL’d back in 2018 too.  

Researched, and researched more.  Read `r/homelab` and `r/homelabsales`.

These servers are noisy, I gather.  I think I would place them in the garage.

The obsession has begun.

I got excited, basically.  I am now hooked.

I started planning, or day-dreaming, to install ESXi on it.  Perhaps, explore ESXi and install TKGi and TKGm on them.  Maybe install NSX and AVI as well.

Friday 12/6, I was off on lieu day for working the Sunday before.  I did a search again in FBm.  I found a posting for a Dell R640 w/ the 2P 5118 Xeon, 128GB, 2x4TB HDD.  It's being sold for $500.

Researched even more.  That posting came with pics.  Interestingly, the hardware captured in the photos doesn’t match the specs that was posted.  The pictured hardware is in 2U form with 8x3.5 front drive bays.  The R640 comes in 1U form.  So I presume the server is actually a R740!  

The poster also has another one R640 posting.  This one has 6126 Xeon, which has higher clock rate but same number of cores 12.  Also has 128GB.  Only has 4TB HDD in total (4x1TB HDDs).

<!--![FBm Post1](/assets/images/FBPost1.png "FBm Post1"){: width="250" }{: .popup-img }   ![FBm Post2](/assets/images/FBpost2.png "FBm Post2"){: width="250" }{: .popup-img }-->
<figure>
<div class="image-row">
<img class="popup-img" src="/assets/images/2025-04-07-searching-for-a-poweredge-server/FBpost1.png" alt="FBm Post1" title="FBm Post1">
<img class="popup-img" src="/assets/images/2025-04-07-searching-for-a-poweredge-server/FBpost2.png" alt="FBm Post2" title="FBm Post2">
</div>
<figcaption>Screenshots of the actual FBm listing by the seller</figcaption>
</figure> 

The service tag of the server was shown in one of the pics.
<!--
![FBm Pic1](/assets/images/FBpic1.jpeg "FBm Pic1"){: width="550" }{: .popup-img }
![FBm Pic2](/assets/images/FBpic2.jpeg "FBm Pic2"){: width="550" }{: .popup-img }
![FBm Pic3](/assets/images/FBpic3.jpeg "FBm Pic3"){: width="550" }{: .popup-img }
![FBm Pic4](/assets/images/FBpic4.jpeg "FBm Pic4"){: width="550" }{: .popup-img }
![FBm Pic5](/assets/images/FBpic5.jpeg "FBm Pic5"){: width="550" }{: .popup-img }
![FBm Pic6](/assets/images/FBpic6.jpeg "FBm Pic6"){: width="550" }{: .popup-img }
-->

<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-04-07-searching-for-a-poweredge-server/FBpic1.jpeg" alt="FBm Pic1" title="FBm Pic1">
<img class="popup-img" src="/assets/images/2025-04-07-searching-for-a-poweredge-server/FBpic2.jpeg" alt="FBm Pic2" title="FBm Pic2">
<img class="popup-img" src="/assets/images/2025-04-07-searching-for-a-poweredge-server/FBpic3.jpeg" alt="FBm Pic3" title="FBm Pic3">
<img class="popup-img" src="/assets/images/2025-04-07-searching-for-a-poweredge-server/FBpic4.jpeg" alt="FBm Pic4" title="FBm Pic4">
<img class="popup-img" src="/assets/images/2025-04-07-searching-for-a-poweredge-server/FBpic5.jpeg" alt="FBm Pic5" title="FBm Pic5">
<img class="popup-img" src="/assets/images/2025-04-07-searching-for-a-poweredge-server/FBpic6.jpeg" alt="FBm Pic6" title="FBm Pic6">
</div>
<figcaption>Pictures of the server from the FBm listing</figcaption>
</figure> 

I looked up the service tag in Dell website.  It is a “OEMR XL R740”.  So, my presumption is getting closer to being true.  Unless the seller posted the wrong pic.  It still has ProSupport but will end on Jan 30 2025, which was in the future, at that time.  Point is that the hardware is still new, relatively.

[Dell OEMR page](https://www.dell.com/support/home/en-us/product-support/servicetag/0-eGhnQ1YvcmplRzdMKzNNMTNDS0ZSZz090/overview){:target="_blank"}

<!--![Dell page](/assets/images/Dell-OEMR.png "Dell-OEMR"){: width="640" }{: .popup-img }-->
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-04-07-searching-for-a-poweredge-server/Dell-OEMR.png" alt="Dell page" title="Dell page">
</div>
<figcaption>Pictures of the server from the FBm listing</figcaption>
</figure> 

This really sounds like a great deal!  That's what I thought.

I texted the guy Chris and told him that I’m interested on the 6126 and if I could get it for $400 that day Friday.  No response.

Saturday, I followed up.  He responded in the evening though.  He said I can pick it up on Tuesday, but he won’t do $400.  He sent the address to his leased storage unit at Youngsville, NC.

I posted in Reddit for price check on R640’s.  Most of them said that it's about the right price for a 640.

Sunday came, I applied for the Homelab license for vSphere and ESX.  I got them approved the same hour.  It seems automated.  Just need to do the process every 90 days.

I’ve been wondering why Chris could be selling a R740 with R640 details.  If I get there, will I be able to at least see it power up to see the specs?

If it really is a R740, then getting it at that price would be a great deal! Right?  I think so!

Monday.  Still doing research.  More research.  Found out that OEMR means that the server was debranded so that it would look & feel like what a vendor or customer would like it to be.  Tried to find a VGA cable.  Found out that a VGA cable is needed for display.  USB port is there for keyboard.  Got to know about IDRAC.

Here’s my check list, in my notes app, to go through when I get to see it physically:

**Dell R640 or R740**

- IDRAC password?
- BIOS password?
- 128 GB RAM?
- 2 Xeon Gold 6126 processors?
- 6 fans?
- Bezel?
- Service Tag card?
- Service Tag is 89BHTH3?
- Does it have 2 PSUs (2x750w)?
- 2U form with 4x3.5” HDDs + 4 blanks?
- Is it the same as in the picture?

Tuesday Dec 10th.  He followed up in the morning around 9.  Got $500 from the ATM.  Drove 26 miles (35 mins) to Youngsville NC.  On the way, I’ve been thinking why I am doing this.  Why haven’t I quit already?  What is this trying to teach me?

Arrived at Youngsville Storage.  Parked.  Texted Chris that I’m there.  He came over to get me from the parking.  There was a gate and he had to unlock it.  We walked to the unit from the parking.  He showed me the hardware.  It was already turned on and in BIOS screen.  I’ve been wondering if I will get to see it power up, and that was my dilemma, and now it’s resolved and all good!  

I got it!  It is the same item that was in the pics, so I am pretty confident that this is a PowerEdge R740!  I got a used R740 with 128GB RAM and 4TB HDD, for 500 bucks in 2024!  I guess I am one lucky SOB!  Chris was great, btw.  It was a smooth transaction.  Thing was heavy maybe around 55lbs.  I carried it to the parking lot with Chris helping open the gate and my 4runner's rear door.  Drove 26 miles back to home.

Took it inside.  Took pics and sent those to my brother.  I remember back in the day when he used to bring home new PC hardware every time and I’ll just enthusiastically watch him install and use software on them.

<!-- 
![At Home Pic1](/assets/images/AtHome1.jpeg "At Home Pic1"){: width="550" }{: .popup-img }

![At Home Pic2](/assets/images/AtHome2.jpeg "At Home Pic2"){: width="550" }{: .popup-img }

![At Home Pic3](/assets/images/AtHome3.jpeg "At Home Pic3"){: width="550" }{: .popup-img }

![At Home Pic4](/assets/images/AtHome4.jpeg "At Home Pic4"){: width="550" }{: .popup-img }

![At Home Pic5](/assets/images/AtHome5.jpeg "At Home Pic5"){: width="550" }{: .popup-img }

![At Home Pic6](/assets/images/AtHome6.jpeg "At Home Pic6"){: width="550" }{: .popup-img }

![At Home Pic7](/assets/images/AtHome7.jpeg "At Home Pic7"){: width="550" }{: .popup-img }
 -->

<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-04-07-searching-for-a-poweredge-server/AtHome1.jpeg" alt="At Home Pic1" title="At Home Pic1">
<img class="popup-img" src="/assets/images/2025-04-07-searching-for-a-poweredge-server/AtHome2.jpeg" alt="At Home Pic2" title="At Home Pic2">
<img class="popup-img" src="/assets/images/2025-04-07-searching-for-a-poweredge-server/AtHome3.jpeg" alt="At Home Pic3" title="At Home Pic3">
<img class="popup-img" src="/assets/images/2025-04-07-searching-for-a-poweredge-server/AtHome4.jpeg" alt="At Home Pic4" title="At Home Pic4">
<img class="popup-img" src="/assets/images/2025-04-07-searching-for-a-poweredge-server/AtHome5.jpeg" alt="At Home Pic5" title="At Home Pic5">
<img class="popup-img" src="/assets/images/2025-04-07-searching-for-a-poweredge-server/AtHome6.jpeg" alt="At Home Pic6" title="At Home Pic6">
<img class="popup-img" src="/assets/images/2025-04-07-searching-for-a-poweredge-server/AtHome7.jpeg" alt="At Home Pic7" title="At Home Pic7">
</div>
<figcaption>Some more hardware pictures</figcaption>
</figure> 

Bought 2 power cables and a VGA cable from the nearest Walmart. $7.99 each cable. 

Brought it up to the bedroom.

Connected the VGA cable, power cables, and usb mouse & keyboard too.  Then, struggled to find the Power Button!  Checked the manual, and it’s on the right control-panel. LOL.

Powered it on.  The fans were loud, like a plane is about to take off.  "Welcome to Dell Airlines".

<!-- ![Dell booting up](/assets/images/Dell-booting.png "Dell booting up"){: width="550" }{: .popup-img } -->
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-04-07-searching-for-a-poweredge-server/Dell-booting.png" alt="Dell booting up" title="Dell booting up">
</div>
<figcaption>Dell OEMR XL R740 booting up...</figcaption>
</figure>

It's running!