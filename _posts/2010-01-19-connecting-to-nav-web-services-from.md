---
layout: post
title: "Connecting to NAV Web Services from …"
date: 2010-01-19 11:47:58
categories: ["Archive", "Web Services"]
tags: ["NAV 2009 SP1", "Web Services"]
permalink: /2010/01/19/connecting-to-nav-web-services-from/
---

I promised to write some posts about how to connect to NAV Web Services from various other programming languages/platforms and I guess it is about time I kept my promise.

I will create a couple of posts on how to connect to NAV Web Services from:

-   PHP
-   Java
-   C# using Web Reference
-   C# using Service Reference
-   Javascript
-   Visual Basic using Web Reference
-   Visual Basic using Service Reference
-   Windows Mobile
-   Microsoft Dynamics NAV 2009SP1
-   and more…

### Scenario

For all these I will create some code that:

-   Connects to NAV Web Services System Service and enumerates the companies in the database
-   Constructs a URL to the Customer Page (based on the first company in the list)
-   Get the name of Customer number 10000
-   Get the Customers in GB that have Location Code set to RED or BLUE.

Here is an example of the output of the PHP web site:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfrom_9B3C/image_thumb.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfrom_9B3C/image_2.png)

Note, that the code is not necessarily perfect PHP or Java code and the intention is not to teach people how to code in these languages (I would be a bad teacher) but more to overcome some of the obstacles when trying to work with NAV.

### Authentication

The very first obstacle is authentication. As you probably know, NAV 2009 only supported SPNEGO (Negotiate) authentication and both PHP and Java doesn’t currently have any support natively for that.

In NAV 2009 SP1 we added a key in the Service Tier configuration file called WebServicesUseNTLMAuthentication. If you set this key to **true** the Web Services listener will only use NTLM authentication and as such be easier accessible from other systems.

The config file you need to modify is placed in the directory where the Service Tier executable is and is called CustomSettings.config. The section you are looking for is:

    <!–  
Turns on or off NTLM authentication protocol for Web Services  
false: Use SPNEGO (recommended)  
true: Use NTLM only  
–>  
<add key=”WebServicesUseNTLMAuthentication” value=”true”></add>

Note that .net works with both values of this settings.

Enjoy

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
