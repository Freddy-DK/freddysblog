---
layout: post
title: "The Service Tier"
date: 2008-10-29 15:30:03
categories: ["Archive"]
tags: ["C#", "NAV 2009", "Service", "Service Tier", "Web Services"]
permalink: /2008/10/29/the-service-tier/
---

My first technical blog post is going to describe some details about the service tier, that some people might find interesting (and some people might think that this is common knowledge:-))

### What is the Service Tier?

Very briefly – the Service Tier is the middle tier in a Microsoft Dynamics NAV 2009 installation. This is where all database access is performed and all business logic is executed, meaning also that this is where the application is running. The Database Tier needs to be SQL Server 2005 or higher and the Client Tier needs to be the Role Tailored Client. When installed, the Service Tier does nothing but to wait for a connection from a Role Tailored Client, so even if the Service Tier is started it really doesn’t consume a lot of resources until some Clients connects to it.

The Service Tier can also be configured as a Web Service listener making part of your application accessible from any Web Service Consumer. It is not recommended that you expose Web Services Directly on the Internet due to security – but you could easily create a high level proxy for some Web Services and expose those to the Internet.

The Role Tailored Client doesn’t connect to the SQL Server directly at all – and the SQL Server could in fact be hidden behind a firewall and be inaccessible from the clients.

When a Client connects to the Service Tier he is authenticated using Windows Authentication and the Service Tier will impersonate the user when connecting to the Database Tier. The Service Tier of course needs permission to do that, we cannot have random computers on the network running around impersonating users – more on this topic later.

### What is installed?

When you install Microsoft Dynamics NAV 2009 (The Demo install), the installer will create 2 services for you:

_**Microsoft Dynamics NAV Business Web Services**  
_Service Name: _MicrosoftDynamicsNavWs  
_This is the Web Service listener. By default this is set to start Manually and without starting this service – you will not be able to do anything with Microsoft Dynamics NAV 2009 Web Services.

**_Microsoft Dynamics NAV Server  
_**Service Name: _MicrosoftDynamicsNavServer  
_This is the Service Tier. By default this service is set to start Automatically.

Both services are set to run as the NETWORK SERVICE user, and by default the Web Service listener has a dependency on the HTTP protocol.

This is what you can see when looking at properties for the service in the services list (Control Panel -> Administrative Tools -> Services).

Another interesting thing pops up if you query the Service Controller for info about the services:

[![image](/assets/images/2008/the-service-tier/image_thumb_2-1.png)](/assets/images/2008/the-service-tier/image_6-1.png)

Notice that the services are running with a flag called WIN32\_SHARE\_PROCESS – meaning that even though you start both services, your task list will only reveal one process running:

[![image](/assets/images/2008/the-service-tier/image_thumb.png)](/assets/images/2008/the-service-tier/image_2.png)

Why is this important?

First of all – the two processes share caches of metadata, memory consumption and everything – meaning that you will save some memory when running in the same process.

Secondly if you at some point want to restart your service tier in order to flush these caches – you actually need to restart both services to achieve that goal.

### Configuration of the Service Tier

The default installation path of Microsoft Dynamics NAV 2009 varies according to version and language of the operating system – but on my machine it is under:

C:\\Program Files\\Microsoft Dynamics NAV\\60

Note that if you install on a 64bit computer, it will be installed C:\\Program Files (x86) because the Service Tier in NAV 2009 is 32bit only.

In this directory you will find a folder called Service (if you installed the demo) and in this directory a configuration file called:

CustomSettings.config

Among the keys in the config file you will find the following 5 keys:

**DatabaseServer / DatabaseName  
**These values are used when the Service Tier connects to the Database Tier – this is the location of your SQL Server and the name of the Database you want to connect to.

After changing these values, you need to restart the Service Tier (both services if started) in order to make the Service Tier connect to a new database.

It seems cumbersome if you are working as a single user using multiple databases – but one Service Tier has one database connection – that is just the way it is.

If you have multiple databases, you want to install multiple Service Tiers – and I will create a post on how to do this. You can also install multiple Service Tiers connecting to the same databases and you can do that on the same computer if you would want that.

**ServerInstance / ServerPort / WebServicePort  
**These are the values that differentiates multiple service tiers on one box.  
The default ServerInstance installed by the installer is DynamicsNAV, the default serverport is 7046 and the default WebServicePort is 7047.

Again – for one Service Tier, these values are fine.

On the Client computer you will connect to a Service Tier using the Select Server dialog, in which you specify the URL of the Service Tier as:

localhost/DynamicsNAV

or

localhost:7046/DynamicsNAV

as you can see – the server URL contains the computername of the Service Tier computer, the port and the Service  Tier instance and if you have multiple Service Tiers, one of these has to be different.

Also the WebService listener uses the Instance in the URL needed to connect to WebServices

[http://localhost:7047/DynamicsNAV/WS//Services](http://localhost:7047/DynamicsNAV/WS//Services)

More about this in a later post on web services.

BTW. Microsoft recommends that you use the Instance name to differentiate between service tiers and not ports.

### Is the Service Tier interpreting C/AL code?

As you probably already know, the answer to this question is No. In NAV 2009

The majority of the Service Tier is written in C# and runs managed code and the application is also converted into C# and at runtime executed as compiled managed Code.

The way this happens is, that whenever you compile an object in C/SIDE, the object is behind the scenes compiled to C# (by the classic client) and the C# code is stored in the Object Metadata table (in the BLOB field called User Code). Furthermore the Object Timestamp field in the Object Tracking table is updated allowing the Service Tier to pickup these changes. When the Service Tier needs to run code from an object – the C# code from the object is written to disk and through some magic compiled into a module, which is loaded dynamically, allowing the Service Tier to replace single code units, pages etc. on the fly.

The directory in which the Service Tier stores the C# files can be found in:

MicrosoftMicrosoft Dynamics NAV60Serversource

where is

**C:\\Documents and Settings\\All Users\\Application Data** on my XP and my Windows 2003 Server and  
**C:\\Users\\All Users** on my Vista box

and the process ID can be found in the Task Manager by choosing to show the PID column.

Having the C# files available in this directory actually allows you to debug this code as described in Claus Lundstrøm’s post:

[http://blogs.msdn.com/clausl/archive/2008/10/14/debugging-in-nav-2009.aspx](http://blogs.msdn.com/clausl/archive/2008/10/14/debugging-in-nav-2009.aspx "http://blogs.msdn.com/clausl/archive/2008/10/14/debugging-in-nav-2009.aspx")

but note that:

-   You will be debugging the C# code – and you cannot see the AL code
-   You will have to install SP1 on VS2005 or VS2008 in order to debug
-   You are debugging the Service Tier meaning that you debug every connection (can be pretty confusing:-))

Enjoy

_Freddy Kristiansen  
__PM Architect  
Microsoft Dynamics NAV_
