---
layout: post
title: "Installing NAV 2009 without A/D"
date: 2009-05-15 11:08:00
categories: ["Archive"]
tags: ["A/D", "Client Tier", "NAV 2009", "Service Tier", "Web Services", "XP"]
permalink: /2009/05/15/installing-nav-2009-without-ad/
---

The majority of installations of NAV 2009 will be in a network environment, you will have a domain server and an Active Directory with your users and your biggest worries will be how to setup a 3T environment on 3 different boxes, getting the delegation setup correctly all of that stuff.

But in a few cases we might run into situations, where there is only a LAN between a number of machines and no server.

In this post I will show what it takes to make one XP computer run the RoleTailored Client (or the Classic) on one XP computer and have another XP computer be the Database Tier, Service Tier – and of course run a client too.

### WorkXP1 and WorkXP2

First of all – I have installed two XP computers with XP SP3 and nothing else – no Office, no Visual Studio, no nothing – just plain good old XP.

In both XP’s I have used the **Set up a home or small office network** wizard to make the computers join a workgroup and be able to see each other

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/InstallingNAV2009withoutAD_95EE/image_thumb_1.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/InstallingNAV2009withoutAD_95EE/image_4.png)

The computers are named WorkXP1 and WorkXP2 – and there is one user on each computer XP1 on WorkXP1 and XP2 on WorkXP2.

So what I want to do, is to install the full NAV 2009 on WorkXP1 and only the Clients on WorkXP2 – and make WorkXP2 run with WorkXP1 as the Service Tier and Database tier for the RTC or the Classic. I will also see that I can in fact run Web Services from WorkXP2 as well.

I did not try to install the database tier separate from the Service Tier, as I do not think it is relevant in a scenario like this.

### Installing the full NAV2009 on WorkXP1

Note, that I cannot just insert the DVD and press Install Demo, because I don’t have Microsoft Outlook installed, so I select **Choose an installation option** and then Customize under Server (or Client or Database Components) and select the following options

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/InstallingNAV2009withoutAD_95EE/image_thumb_2.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/InstallingNAV2009withoutAD_95EE/image_6.png)

and then just accept the default setup values and install.

### Installing the Clients on WorkXP2

on the WorkXP2 we install both clients – Again **Choose an Installation option** and select customize under Client and Select to run Classic from My Computer.

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/InstallingNAV2009withoutAD_95EE/image_thumb_4.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/InstallingNAV2009withoutAD_95EE/image_10.png)

After selecting Next you will need to Click the RoleTailored Client link and specify the Service Tier computer (which is WorkXP1).

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/InstallingNAV2009withoutAD_95EE/image_thumb_3.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/InstallingNAV2009withoutAD_95EE/image_8.png)

### Installing…

If you wonder how – I am running this experiment as two virtual machines on a Windows 2008 Server running Hyper-V.

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/InstallingNAV2009withoutAD_95EE/image_thumb_6.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/InstallingNAV2009withoutAD_95EE/image_14.png)

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/InstallingNAV2009withoutAD_95EE/image_thumb_7.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/InstallingNAV2009withoutAD_95EE/image_16.png)

### Workgroup networking

As you might have guessed – starting the clients on WorkXP1 just works, and trying the same on WorkXP2 doesn’t.

We need to do 3 things in order to make this work…

### 1\. Create the XP2 user on WorkXP1

The way workgroups works is to authenticate the XP2 user on WorkXP1 using his own Username and Password, so you need to create all the users on the “server”, and note, then need to have the same password.

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/InstallingNAV2009withoutAD_95EE/image_thumb_8.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/InstallingNAV2009withoutAD_95EE/image_18.png)

After this you of course need to create the user in NAV (using the Classic Client on WorkXP1) and give him the right permissions (In my world everybody is SUPER).

Having done this, we try to start the Classic Client on the WorkXP2 machine and get the following result:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/InstallingNAV2009withoutAD_95EE/image_thumb_9.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/InstallingNAV2009withoutAD_95EE/image_20.png)

This means that the firewall is blocking access to the SQL Server.

### 2\. Open the firewall for SQLSERVR (if you want to run Classic)

On WorkXP1 open the Control Panel and open Windows Firewall. Click Add Program, browse and locate **SQLSERVR.EXE** under **C:\\Program Files\\Microsoft SQL Server\\MSSQL.1\\MSSQL\\Binn** add this to the list of exceptions:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/InstallingNAV2009withoutAD_95EE/image_thumb_10.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/InstallingNAV2009withoutAD_95EE/image_22.png)

At the top of the list of exceptions you will BTW. find DynamicsNAVServer – which means that the RoleTailored actually doesn’t need this setting.

Trying to connect from the Classic Client now will give a different error:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/InstallingNAV2009withoutAD_95EE/image_thumb_11.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/InstallingNAV2009withoutAD_95EE/image_24.png)

The reason for this is that XP by default is running Simple Sharing, meaning that everybody will login as Guest on a different computer and not trying to login with their own Username and Password.

### 3\. Disable Simple Filesharing on WorkXP1

On WorkXP1 in the Control Panel or any explorer window select Tools -> Folder Options, goto the View Tab and remove the checkmark from **Use simple file sharing**.

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/InstallingNAV2009withoutAD_95EE/image_thumb_12.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/InstallingNAV2009withoutAD_95EE/image_26.png)

After having done this – you should be able to connect using the Classic Client and also the RoleTailored Client should now work.

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/InstallingNAV2009withoutAD_95EE/image_thumb_13.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/InstallingNAV2009withoutAD_95EE/image_28.png)

It is probably not any different than what you would do with NAV 5.0 – but a number of people has asked me whether or not it is possible – and it is.

BTW – Web Services works in exactly the same way. If you start the Web Service listener on WorkXP1 – you can navigate to

[http://WorkXP1:7047/DynamicsNAV/WS/CRONUS%20International%20Ltd/Services](http://WorkXP1:7047/DynamicsNAV/WS/CRONUS%20International%20Ltd/Services) (this is the SP1 URL)

You might have to open the firewall for port 7047 if it doesn’t pick up that DynamicsNAVServer is listening on both ports.

Enjoy

_Freddy Kristiansen  
_PM Architect  
Microsoft Dynamics NAV
