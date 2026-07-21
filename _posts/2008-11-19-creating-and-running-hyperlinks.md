---
layout: post
title: "Creating and Running Hyperlinks"
date: 2008-11-19 00:06:23
categories: ["Archive"]
tags: ["Bookmark", "Client Tier", "ISSERVICETIER", "Personalization", "URL"]
permalink: /2008/11/19/creating-and-running-hyperlinks/
---

In the developer help for NAV 2009 (nav\_adg.chm), there is a description for creating and running Hyperlinks – I will not try to repeat all the information in the documentation – so please read the documentation before reading this post.

There are however a couple of thing, which are not described in detail.

My next post is about the Search demo, which was shown at Convergence yesterday (Partner day) – I will describe how this demo is done, in a 2 step walkthrough (first is to get it to work on all operating systems and second is to make it work as a Windows Vista Gadget – stay tuned)

### Bookmark

<table border="1" width="847" cellspacing="0" cellpadding="2"><tbody><tr><td valign="top" width="76">Bookmark</td><td valign="top" width="350">This positions the cursor on a single record in a table.<p></p><p>Only automatically generated bookmarks should be used. If you enter an incorrect bookmark, you will get an error message.</p></td><td valign="top" width="419"><b>dynamicsnav://localhost/DynamicsNAV/CRONUS International Ltd./runpage?page=22&amp;bookmark=120000000089083237343</b></td></tr></tbody></table>

But how do you get your hands on this automatically generated bookmark?

It is actually described in another section of the documentation – **Walkthrough: Creating a Link in a Report.**

`FORMAT(RecordRef.RECORDID,0,10)`

The usage of value 10 in this expression is a RoleTailored client feature only that will format `RECORDID` into a text representation that is compatible with the URL handler of reports and pages. Note that this function only works if ISSERVICETIER = TRUE – if you run a code unit in the classic client, trying to use the FORMAT(xx,0,10) it will not return a bookmark for the Role Tailored Client.

### Personalization ID

<table border="1" width="850" cellspacing="0" cellpadding="2"><tbody><tr><td valign="top" width="130">Personalization ID</td><td valign="top" width="292">This is the unique identification used in personalization to store settings in the User Metadata table. If a personalization ID is not found, the page is launched without personalization.</td><td valign="top" width="423"><b>dynamicsnav://localhost/DynamicsNAV/CRONUS International Ltd./runpage?page=22&amp;personalization=0000232e-0000-001a-0008-0000836bd2d2</b></td></tr></tbody></table>

What is this personalization ID and how do you get to that?

The Personalization ID is the way we distinguish the different views of things like the Sales Order List View. In Susans Role Center, there are 6 List Places, which all use the same underlying List Place (9305)

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/CreatingandRunningHyperlinks_55D8/image_thumb.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/CreatingandRunningHyperlinks_55D8/image_2.png)

In fact, this is the reason for these List Places to be grouped together – that they have a common page number, and if we didn’t have the personalization ID, all these list places would share personalization – and in a Role Tailored User Experience, there are differences in which actions you typically would promote in a list place with shipped not invoices sales orders and a list place with ready to ship sales orders.

BTW – if you wonder where the last 4 come from – the are auto generated from the stacks in Susans Activities – and the reason for this is, that in order for navigation to work, we need to have a node in the navigation pane for every possible list place we can have in the navigation area.

The Personalization ID for these views are:

<table border="0" width="484" cellspacing="0" cellpadding="2"><tbody><tr><td valign="top" width="198">Sales Orders</td><td valign="top" width="284">0000232E-0000-0002-0008-0000836BD2D2</td></tr><tr><td valign="top" width="198">Shipped Not Invoiced</td><td valign="top" width="284">0000232E-0000-0007-0008-0000836BD2D2</td></tr><tr><td valign="top" width="198">Sales Orders – Open</td><td valign="top" width="284">00002364-0000-0006-0008-0000836BD2D2</td></tr><tr><td valign="top" width="198">Ready to Ship</td><td valign="top" width="284">00002364-0000-000C-0008-0000836BD2D2</td></tr><tr><td valign="top" width="198">Partially Shipped</td><td valign="top" width="284">00002364-0000-000B-0008-0000836BD2D2</td></tr><tr><td valign="top" width="198">Delayed</td><td valign="top" width="284">00002364-0000-000A-0008-0000836BD2D2</td></tr></tbody></table>

and how in earth did I find these ID´s?

Simple enogh – In a VPC, Classic Client I open the User Personalization table. Then I personalize these list places one after the other, and every time I have personalized a list place I refresh my table view and a new record pops up:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/CreatingandRunningHyperlinks_55D8/image_thumb_1.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/CreatingandRunningHyperlinks_55D8/image_4.png)

Sorted by Personalization ID.

For Task Pages – the personalization ID will typically just be the same as the Page ID.

But what can you use this for?

Very little as the matter of fact – if you launch a RunPage url with a list place as paramter, the list place will open as a task page, so you will need to specify which personalization you want – else you will create a personalization set with the same ID as the list place (9305 in this case). The personalizations stored under this personalization ID will never be used by the RTC (unless you launch that URL again), since we always specify the above ID’s.

So you should think that the following URL

“DynamicsNAV:////CRONUS International Ltd./RunPage?Page=9305&personalization=0000232E-0000-0007-0008-0000836BD2D2”

would open the Shipped Not Invoiced List Place in a Task Page.

That is unfortunately only partially true – you will open a Task Page with a List of Sales Orders and the Personalizations in this list are Shipped Not Invoiced – but you will NOT inherit the filters from Shipped Not Invoiced and the caption is also not what you would expect.

So now told what the personalization ID is and how to use it, but I am afraid it is only for limited usage right now.

If you want to launch a listplace you need to do like:

“DynamicsNAV:////CRONUS International Ltd./navigate?node=Home/Sales Orders/Ready to Ship”

Only problem with this URL is, that it always opens a new Client (eating one extra license).

I will investigate whether there are other ways of getting to a list place with filters and personalization – but for now, don’t specify pesonalization ID when launching pages via URL’s unless you have a good reason for doing so.

Enjoy

_Freddy Kristiansen  
__PM Architect  
Microsoft Dynamics NAV_
