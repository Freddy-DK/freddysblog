---
layout: post
title: "Displaying Company information in Card pages"
date: 2009-11-29 12:03:32
categories: ["Archive"]
tags: ["Client Tier", "COMPANY", "NAV 2009 SP1"]
permalink: /2009/11/29/displaying-company-information-in-card-pages/
---

I received a question from a customer, who is running multiple companies and often have multiple instances of NAV open with different companies. On the main page, they do have information about what company is the active:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/DisplayingCompanyinformationinCardpages_A991/image_thumb.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/DisplayingCompanyinformationinCardpages_A991/image_2.png)

but when they have opened a number of Task Pages in the various instances of NAV, they cannot distinguish one from the other.

Example:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/DisplayingCompanyinformationinCardpages_A991/image_thumb_1.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/DisplayingCompanyinformationinCardpages_A991/image_4.png)

This image tells you the page and the Customer name – and you can easily identify the right page when Alt+TAB’ing between pages, but if you have multiple companies this doesn’t help you a lot.

### So what determines the caption?

The fields used in the caption on a page is determined by:

**DataCaptionExpr** on the page. This is an expression, which can use fields, functions etc. to build up a caption. If that isn’t defined, the client looks for

**DataCaptionFields** on the page. This is a collection of fields, which are used to build the caption by adding them together with a character 183 (middle dot) between them. If that isn’t defined, the client looks for

**DataCaptionFields** on the table, which basically is the same as DataCaptionFields on the page.

In a standard NAV, there is no DataCaptionExpr nor DataCaptionFields defined on the Customer Card, but on the Customer table you find:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/DisplayingCompanyinformationinCardpages_A991/image_thumb_2.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/DisplayingCompanyinformationinCardpages_A991/image_6.png)

In order to add the Company name behind the caption you will need to change the DataCaptionExpr on the Customer Card to f.ex.

`"No." + ' · ' + Name + ' ['+COMPANYNAME+']'`

which would cause the Customer Card to look like

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/DisplayingCompanyinformationinCardpages_A991/image_thumb_3.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/DisplayingCompanyinformationinCardpages_A991/image_8.png)

You can of course select to change the expression to whatever you like – or maybe create some function, which automagically returns a caption, only real flipside is that you need to modify the card pages, on which you need this functionality. In the end this is probably not a very large number.

Enjoy

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
