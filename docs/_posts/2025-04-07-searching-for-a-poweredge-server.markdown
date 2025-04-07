---
layout: post
title:  "Searching for a Poweredge server"
date:   2025-04-07 17:48:00 -0400
categories: deephackmode.io update
---
It was around Black Friday 2024 when I was searching the Dell website for deals.  I wasn't sure what to get for myself and so I thought why not check out servers used for ESXi hosts.  That’s how I came across the “Poweredge” servers.  They are expensive though, like in the thousands of dollars.

I searched in FB marketplace for a “Poweredge”.  Found one being sold by a guy in Cary NC.  Poweredge R720 for $200.  2 cpus, with 128gb ram but no hard drives!  

Did my research.  Found a reddit where homelab is being discussed.  I gather that R720’s are worth nothing any more.  People are just giving them away.  

Googled it also and found that it is EOL since 2018. 5/25/2018 - https://www.topgun-tech.com/end-of-service-life/dell-emc/poweredge/ 

There was someone in FBm selling a R730 for $150, but only has 32gb and no drives.  Also found out that R730’s were EOL’d back in 2018 too.  

Researched, and researched more.  Read r/homelab and r/homelabsales.

These servers are noisy, I read.  I think I would place them in the garage.

The obsession has begun.

I got excited, basically.  

I started planning to install ESXi on it.  Explore ESXi and install TKGi and TKGm on them.  Maybe install NSX and AVI as well.

Friday 12/6, I was off on lieu day for working the Sunday before.  I did a search again in FBm.  I found the postings for Dell R640 w/ the 2P 5118 Xeon, 128gb, 2x4tb hdd.  It's being sold for $500.

![FBm Post](/assets/images/FBpost1.png "FBm Post"){: width="250" }
![FBm Post](/assets/images/FBpost2.png "FBm Post"){: width="250" }

Researched even more.  That posting came with pics.  Interestingly, the hardware captured in the photos doesn’t match the specs that was posted.  The pic’d hardware is in 2U form w/ 8x3.5 front drive bays.  The R640 comes in 1U form.  So I presume the server is actually a R740!  



The poster also has another one R640 posting.  This one has 6126 Xeon, which has higher clock rate but same number of cores 12.  Also has 128gb.  Only has 4tb hdd in total (4x1tb hdds).

The service tags of these servers were shown in the pics.

I looked up the service tags in Dell website.  They are “OEMR XL R740”.  So, my presumption is getting more accurate.  Unless they posted the wrong pic.   It still has ProSupport but will end next month Jan 30 2025.

https://www.dell.com/support/home/en-us/product-support/servicetag/0-eGhnQ1YvcmplRzdMKzNNMTNDS0ZSZz090/overview 

[attach photo here]

This really sounds like a great deal.

I texted the guy Chris and told him that I’m interested on the 6126 and if I could get it for $400 that day Friday.  No response.

Saturday, I followed up.  He responded in the evening though.  He said I can pick it up on Tuesday, but he won’t do $400.  He sent the address to his leased storage at Youngsville, NC.

I posted in Reddit for price check on R640’s.

Sunday came, I applied for the Homelab license for VCF and ESX.  I got them approved the same hour.  It seems automated.  Just need to do the process every 90 days.

I’ve been wondering why could he be selling a R740 with R640 details.  If I get it, will I be able to at least see it power up to see the specs?

If it really is a R740, then getting it at that price would be a great deal! Right?

Monday.  Still doing research.  More research.  Found out that OEMR means that the server was debranded so that it would look & feel like what the customer would like.  Tried to find a VGA cable.  Found out that a VGA cable is needed for display.  USB port is there for keyboard.  Got to know about IDRAC.

Here’s my check list when I get to see it physically:

**Dell R640**

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

Arrived at Youngsville Storage.  Parked.  Texted Chris that I’m there.  He came over to get me from the parking.  There was a gate and he had to open it.  We walked to the unit from the parking.  He showed me the unit.  It was already turned on and in BIOS screen.  I’ve been wondering if I will get to see it power up, and that was my dilemma, and now it’s resolved and all good!  

I got it!  Chris was great.  It was a smooth transaction.  Thing was heavy maybe 55lbs.  I carried it to the parking lot with Chris helping open the gate and my 4runner's rear door.  Drove 26 miles back to home.

Took it inside.  Took pics and sent those to my brother in the Philippines.  I remember he used to bring home new hardware every time and I’ll just watch him do things enthusiastically.

Bought 2 power cables and a VGA cable from nearest Walmart. $7.99 each cable. 

Brought it up to the bedroom.

Connected the VGA cable, power cables, and usb mouse & keyboard too.  Then, struggled to find the Power Button!  Checked the manual, and it’s on the right control-panel. LOL.

Powered it on.