---
layout: post
title: "Multiple Service Tiers – SP1"
date: 2009-08-05 15:55:35
categories: ["Archive"]
tags: ["Multiple", "NAV 2009 SP1", "Service", "Service Tier", "Web Services"]
permalink: /2009/08/05/multiple-service-tiers-sp1/
---

Right around the release of Microsoft Dynamics NAV 2009, I wrote a blog entry with some .bat files on how to create multiple Service Tiers when working with NAV 2009.

The blog post is here: [http://blogs.msdn.com/freddyk/archive/2008/10/29/multiple-service-tiers.aspx](http://blogs.msdn.com/freddyk/archive/2008/10/29/multiple-service-tiers.aspx "http://blogs.msdn.com/freddyk/archive/2008/10/29/multiple-service-tiers.aspx")

Now NAV 2009 SP1 is about to be released, it is time for a small update. One of the files of the package is a CustomSettings.template file, which really is just the CustomSettings.config with a few values replaced with variable names, so that we can replace those automagically.

Now in SP1, the CustomSettings.config has changed – new keys have been added and we also support named instances in the database.

SP1 will actually run with the old config file, so we could just ignore the entire thing and continue as if nothing happened – the .bat files will still work in SP1.

However – if we want to take advantage of the named instances in SQL Server or we want to have the additional keys available for modifying we need to change something.

I have created a new CustomSettings.template based on the SP1 config file – copy the config file and change the following keys:

    

```
<add key="DatabaseServer" value="#DBSERVER#"></add>
<add key="DatabaseInstance" value="#DBINSTANCE#"></add>
<add key="DatabaseName" value="#DATABASE#"></add>
<add key="ServerInstance" value="#INSTANCE#"></add>
```

and extended the createservice.bat file to also allow a database instance to be specified, meaning that the usage is now:

```
CreateService name [dbserver] ["dbinstance"] ["dbname"] [demand|auto|disabled] [both|servicetier|ws]
```

The new .zip file is available for download [here](http://www.freddy.dk/MultipleServiceTiers-SP1.zip).

Enjoy

_Freddy Kristiansen  
__PM Architect  
Microsoft Dynamics NAV_
