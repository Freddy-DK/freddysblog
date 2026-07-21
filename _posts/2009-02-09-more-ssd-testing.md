---
layout: post
title: "More SSD testing"
date: 2009-02-09 18:02:48
categories: ["Archive"]
tags: ["FusionIO", "Intel", "NAV 2009", "Performance", "SSD", "STEC"]
permalink: /2009/02/09/more-ssd-testing/
---

If you haven’t read my post about [Running Microsoft Dynamics NAV on SSD’s](http://blogs.msdn.com/freddyk/archive/2008/11/02/running-microsoft-dynamics-nav-on-ssd-s-solid-state-drives.aspx) – you should do so first.

After having posted the initial results, I was contacted by other vendors of SSD’s wanting to see whether we could do some additional testing on other hardware. In the interest of the end user, I accepted and once more allocated a slot in the performance lab in Copenhagen.

The new drives to test were:

-   STEC 3½” Zeus IOPS SSD 146GB
-   STEC 2½” MACH8 IOPS SSD 100GB
-   Intel 2½” SSDSA2SH032G1GN 32GB (actually 32GB wasn’t enough for the testing so we took two of those and striped them)

All of these drives looks like standard HDD’s with a SATA interface. Installation is plug and play and no driver installation.

### Disclaimer

Remember that the tests we have run here are scenario tests, designed to measure performance deltas on Microsoft Dynamics NAV to make sure that a certain build of NAV doesn’t suddenly get way slower than the previous version and gets shipped with poor performance.

Also again – I haven’t optimized the SQL server at all when doing these tests so you might not see the same performance gain if you switch your drives to SSD’s – or you might see more performance gain (if you know how to optimize for these things).

My testing is ONLY replacing a standard HDD (Seagate Barracuda 500GB, 7200 RPM SATA) with a SSD – and test the same scenarios.

The scenarios are being run for 180minutes each and the perf. testing starts after a 10 minutes warm up time. The tests I will be referring to here are all done simulating 50 users towards a service tier.

### Final build of NAV 2009

Between the time of the prior tests and the new tests we released the final version of NAV – so the tests in this post will be based on the RTM version of NAV. Also we got some new Lab equipment – so in order to be honest to the FusionIO tests – we retook all of these tests as well.

In the new list of tests you will see 5 results: HDD, FusionIO, STEC5 (3½”), STEC3 (2½”), Intel

I will use the same tests as in the original post.

### Here we go

[![clip\_image002](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MoreSSDtesting_BD32/clip_image002_thumb.gif)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MoreSSDtesting_BD32/clip_image002_2.gif)

As we can see if we compare this test to the test on the pre-release – this test is faster on the HDD than the prior test was on the FusionIO SSD.

This of course also means that the performance gain by using SSD’s in this test is smaller – but still a 21% performance enhancement by changing the drive to SSD isn’t bad at all.

[![clip\_image002](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MoreSSDtesting_BD32/clip_image002_thumb.jpg)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MoreSSDtesting_BD32/clip_image002_2.jpg)

Again 25% performance enhancement by changing to SSD’s and the difference between the different types of SSD’s is insignificant in comparison.

[![clip\_image002\[6\]](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MoreSSDtesting_BD32/clip_image002%5B6%5D_thumb.jpg)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MoreSSDtesting_BD32/clip_image002%5B6%5D.jpg)

Again 25-30% performance enhancement by changing to SSD’s and not a huge difference between the technologies.

Note, that as the numbers get lower – the measurement uncertainty can play a role in the results.

[![clip\_image002\[8\]](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MoreSSDtesting_BD32/clip_image002%5B8%5D_thumb.jpg)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MoreSSDtesting_BD32/clip_image002%5B8%5D.jpg)

This test is around 10 times faster than when we did the test on the pre-release and now the results are so fast that the uncertainty causes some results as being skyhigh. Analyzing the results actually reveals that there isn’t that much of a difference.

[![clip\_image002\[10\]](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MoreSSDtesting_BD32/clip_image002%5B10%5D_thumb.jpg)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MoreSSDtesting_BD32/clip_image002%5B10%5D.jpg)

Same picture again – significant performance enhancement changing to SSD’s – not a huge difference between the technologies.

### Wrap-up

Test results where not as clear as the last time – primarily because the RTM version solved some of the perf. problems and due to new hardware in the Perf. lab – but still tests show 20-30% performance increase.

I still think SSD’s are here to stay and I do think that people can take advantage of the increased performance they will get simply by changing the drives in their SQL Server. I haven’t tested what performance enhancements you would get from running the Service Tier on a box with SSD’s – but I wouldn’t expect a huge advantage if the service has sufficient RAM.

I will not be conducting any more tests – the primary reasons for this is, that I do not have the hardware anymore – meaning that I couldn’t do a re-run on the same hardware and compare all the different technologies – so any comparison would be unfair to one or the other.

Enjoy

_Freddy Kristiansen  
__PM Architect  
Microsoft Dynamics NAV_
