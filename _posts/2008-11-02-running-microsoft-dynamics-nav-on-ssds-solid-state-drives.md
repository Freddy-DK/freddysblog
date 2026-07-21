---
layout: post
title: "Running Microsoft Dynamics NAV on SSD’s (Solid State Drives)"
date: 2008-11-02 12:41:39
categories: ["Archive"]
tags: ["FusionIO", "NAV 2009", "Performance", "SSD"]
permalink: /2008/11/02/running-microsoft-dynamics-nav-on-ssds-solid-state-drives/
---

Solid State Drives are here. Laptops are sold with SSD’s (a bit expensive) and also the server market is seeing SSD’s coming out promising incredible performance from your storage.

But what does this mean for NAV?

I contacted FusionIO ([www.fusionio.com](http://www.fusionio.com)), one of the major players in the high end SSD market and borrowed a drive for my server. The purpose of this was to test the performance of the drive in our NAV performance lab in Copenhagen. I also wanted to test whether the drive was reliable and easy to use / install.

The installation was a piece of cake: Open the server, Insert the card. Close the server, Install the driver – done!

Regarding reliability (after all I did get a beta version of the drivers) – I haven’t experienced one single problem with the server since installing the drive – so I guess that one gets a checkmark as well.

### Executive summary

The solid state drive is dramatically faster in a number of the cold start scenarios – around 20-25% faster than the same test run on hard drives.

In other tests we see no a big difference which can be either because the SQL server makes extensive usage of caching or the test scenario is heavier on the CPU on the service tier.

In a few tests there is a small performance drop (typically in the magnitude of < 10ms for the test) – I expect this to be measurement inaccuracy.

Some reports, which are heavy on the database will also experience dramatic performance gain – again around 20-25%.

But the real difference is when we run 100 user tests – the picture is very clear, performance gain on a lot of the scenarios are 40%+

Buying a solid state drive for a company running 1-10 users seems overkill, it won’t hurt performance, but the more users you have on the system, the more performance gain you will get out of a drive like this.

Of course you will see the same picture of you have a smaller number of users but huge database or if you for other reasons have a large number of transactions.

Remember though that these solid state drives for servers is fairly new technology and priced at around $30/Gb (slightly more than a hard drive:-)) – prices always vary and these prices will probably be adjusted as we go along.

### Initial testing

Before going to Copenhagen, I did a small test on the performance database (which contains a LOT of sales lines). I ran a SQL statement which calculated the SUM of all sales lines in the system.

On the harddrives this took approx. 45 seconds the first time (the second time, the cache would come into effect and the time was 1 second)

On the solid state drive – it took approx. 2 seconds the first time (and of course 1 second the second time).

But this of course doesn’t tell anything about NAV performance on these drives…

The server runs Windows 2003 server 64bit version with SQL Server 64bit – it has 8Gb of RAM, 3 \* 500GB Hard drives (SATA – 7500rpm – one for system, one for data and one for SQL Server Log) and one 80GB FusionIO drive (which in the tests runs both Data and Logs for the databases on this drive).

The specs for the FusionIO drive are (from FusionIO’s marketing material):  
– 700 MB/s read speed  
– 600 MB/s write speed  
– 87,500 IOPS (8K packets) transaction rate  
– Sustain 3.2 GBps bandwidth with only four ioDrives  
– 100,000 IOPS (4K packets) with a single ioDrive  
– PCI-Express x4 Interface

BTW – these specifications are some of the fastest I have found on the market.

Intel launches a drive which is 250Mb/s read and 170Mb/s writes with 35000 IOPS and the small laptop 2.5” SSD’s from Imation specs at around 125Mb/s write and 80Mb/s reads with 12500 IOPS.

These drives will probably increase performance on demo laptops a lot – but are not suited for servers.

Texas Memory Systems have launched a series of SSD’s that matches the performance and as FusionIO – they are primarily focused on the server market – in fact if you look at the document about SQL Server performance on their drives ([http://www.texmemsys.com/files/f000174.pdf](http://www.texmemsys.com/files/f000174.pdf)) you will find a Danish AX customer (Pilgrim A/S) who is live on this technology and he states:

_“Don’t be scared. The technology has proven itself superior in many circumstances. You just have to know when to apply it. For applications with smaller databases but heavy load, it’s a life saver”._

The purpose of this blog is not to point at any one particular provider of SSD’s – but I do want to mention that if you go for this – beware that performance specs on these things vary a lot – and from what I have seen, performance costs money.

### Details

Note that I did absolutely nothing to improve the SQL performance on this machine – that is why we will run tests on this server both on drives and on solid state. The first rerun is the test run on the build in hard drives and the second rerun is on the solid state drive.

The Client and Service Tier computers are kept the same and only the location of the attached database it altered in order to know the difference in performance when switching to solid state.

Note also that these tests are NAV 2009 tests – I do think the picture for NAV Classic is similar when running multiple users though, since the multi user tests doesn’t include UI (that would be the Client tests) and really just measure the app and stack performance.

### Details – Reporting scenarios

Of the report tests I will show two test results – a performance test doing Adjust Cost Item Entry and a Customer Statement. The first one hits the SQL Server performance a lot and shows very good performance on the 2nd rerun (the SSD) and the Customer Statement is more heavy on the Service Tier than on the SQL Server

[![clip\_image002](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningNAVonSSDsSolidStateDrives_757E/clip_image002_thumb.gif)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningNAVonSSDsSolidStateDrives_757E/clip_image002_2.gif)

This report is a batch job adjusting the Cost on Item entries – performance gain – around 25%

[![clip\_image004](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningNAVonSSDsSolidStateDrives_757E/clip_image004_thumb.gif)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningNAVonSSDsSolidStateDrives_757E/clip_image004_2.gif)

Customer statement report – the performance database doesn’t contain a lot of data, which affects the customer statement, the report with the given data isn’t hard on SQL.

### Details – Client scenarios

Of the Client tests the major performance advantage comes when doing a cold start (this is where the Service Tier needs warm up – and this of course hits the SQL Server a lot). This shows us, that when running single user scenarios in a pre-heated (warm start) environment, we don’t hit the SQL server a lot (unless in some report or other special scenarios) – we probably knew this already.

[![clip\_image006](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningNAVonSSDsSolidStateDrives_757E/clip_image006_thumb.gif)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningNAVonSSDsSolidStateDrives_757E/clip_image006_2.gif)

This scenario is starting up the Client and opens an existing sales order – cold start – performance gain around 15%

The same scenario in a warm start shows no performance gain at all – probably because everything is cached.

[![clip\_image008](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningNAVonSSDsSolidStateDrives_757E/clip_image008_thumb.gif)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningNAVonSSDsSolidStateDrives_757E/clip_image008_2.gif)

Again Cold start scenario – 40% faster – the same scenario in warm start is only a fraction faster.

### Details – multiple user tests

Now this is the area, where the solid state drive should be faster – this is where the SQL server gets hit more and where the caching on the SQL server cannot contain everything, and I think the perf tests do show, that this is the case.

I will let the number speak for themselves on the important performance scenarios:

[![clip\_image010](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningNAVonSSDsSolidStateDrives_757E/clip_image010_thumb.gif)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningNAVonSSDsSolidStateDrives_757E/clip_image010_2.gif)

40% faster

[![clip\_image012](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningNAVonSSDsSolidStateDrives_757E/clip_image012_thumb.gif)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningNAVonSSDsSolidStateDrives_757E/clip_image012_2.gif)

40% faster

[![clip\_image014](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningNAVonSSDsSolidStateDrives_757E/clip_image014_thumb.gif)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningNAVonSSDsSolidStateDrives_757E/clip_image014_2.gif)

40% faster

[![clip\_image016](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningNAVonSSDsSolidStateDrives_757E/clip_image016_thumb.gif)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningNAVonSSDsSolidStateDrives_757E/clip_image016_2.gif)

80% faster

[![clip\_image018](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningNAVonSSDsSolidStateDrives_757E/clip_image018_thumb.gif)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningNAVonSSDsSolidStateDrives_757E/clip_image018_2.gif)

50% faster

[![clip\_image020](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningNAVonSSDsSolidStateDrives_757E/clip_image020_thumb.gif)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningNAVonSSDsSolidStateDrives_757E/clip_image020_2.gif)

50% faster

I think all of these are showing dramatic performance gain – of 40% or more when running  with the solid state drive, and I think that this shows that the technology can do a lot for some NAV customers.

I also do think that AX will we similar or better results running Solid State especially with large number of users (which have been confirmed by the report from Texas Memory Systems).

I do think the SSD technology has arrived and when pricing gets right – they will become very popular on the market. I think we will see them take over the laptop market – but I also do think that these tests shows that some providers are ready for the server marked as well. I think the major obstacle right now is that people somehow trust their data on hard drives more than on a SSD – but I think that will change as we get to know the technology better.

**A special thanks to the performance team in Copenhagen for helping out and a special thanks to FusionIO for lending me a IODrive on which I could perform the test.**

Enjoy

_Freddy Kristiansen  
__PM Architect  
Microsoft Dynamics NAV_
