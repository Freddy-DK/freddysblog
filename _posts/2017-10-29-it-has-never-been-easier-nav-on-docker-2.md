---
layout: post
title: "It has never been easier… (NAV on Docker #2)"
date: 2017-10-29 09:20:13
categories: ["Docker"]
tags: ["Containers", "Docker", "NAV 2016", "NAV 2017", "NAV on Docker"]
permalink: /2017/10/29/it-has-never-been-easier-nav-on-docker-2/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

As of this morning, all shipped cumulative updates (+ all localizations) of Microsoft Dynamics NAV 2016 and 2017 are available as Docker images in the Docker Hub, ready for execution on a Windows system with Docker installed.

# How do I use them?

This means, that if you have a computer running Windows 10 Professional edition or Windows Server 2016, you can install Docker (follow [this](https://blogs.msdn.microsoft.com/webdev/2017/09/07/getting-started-with-windows-containers/) blog post until the _Set up an ASP.NET…_ section or for Windows Server 2016, you can install the EE version by following [this](https://docs.docker.com/engine/installation/windows/docker-ee/) blog post) and run:

```
C:>docker run -e accept_eula=Y -m 4G microsoft/dynamics-nav
```

in a Command Prompt running as **Administrator** and get the W1 country version of the latest cumulative update of the latest version of NAV running on your computer within a few minutes (the first time will take more time for downloading base components)

Or you can run any other cumulative update (of 2016 or 2017), just by specifying the version, the CU and the country version you want as a tag:

```
C:>docker run -e accept_eula=Y -m 4G microsoft/dynamics-nav:2016-cu5-dk
```

**Note:** One of my next posts will be a troubleshooting blog post – what can you do if things doesn’t work.

# Where is my NAV!

When you execute the docker run command, Docker will start downloading the image if it doesn’t already exist. After downloading the image from the docker hub, you should be seeing something like this:  
[![](/assets/images/2017/it-has-never-been-easier-nav-on-docker-2/c75f6-dockerrun2-e1509263643176-1.png)](/assets/images/2017/it-has-never-been-easier-nav-on-docker-2/c75f6-dockerrun2-e1509263643176.png)

and if you open up your internet browser and type in the Web Client URL, you should be prompted with a login dialog, where you can login with the NAV Admin Username/Password displayed:  
[![](/assets/images/2017/it-has-never-been-easier-nav-on-docker-2/bc554-navlogin-1.png)](/assets/images/2017/it-has-never-been-easier-nav-on-docker-2/bc554-navlogin.png)

So you are clearly running NAV somewhere but if you investigate your system, you won’t find this version of NAV installed. You also might not be running IIS or SQL Server, so how in earth can NAV run on my box with database, Web Client and all?

If you investigate your Hyper-V Manager you also won’t see any Virtual Machines, so…

# Is it magic?

When every other attempt to explain something fails, it must be magic, right?

Well fortunately, there are good explanations to what you are seeing. You are running NAV in a Container (aka NAV on Docker). It is kind of a Virtual Machine, but then again it isn’t. For now, the only thing you really need to know is, that what’s inside the container is isolated from the host, meaning that you can run any number of containers (resources allowing) side by side without any conflicts.

Yes, that also means that you can run different versions, different cumulative updates and different localization’s of NAV very efficiently side by side.

# OK, I get it!, but is this it?

Now you might be thinking: So, I can run the NAV Web Client towards the NAV Demo Database – big deal – but I need much more than that, can I …???

And you certainly can get much more than that.

In fact, you can:

-   use your own database in the Container
-   connect to your own SQL Server
-   integrate NAV with your AD (and use Windows Authentication)
-   avoid the SSL warning when running locally
-   use the Classic Development Environment and customize NAV
-   use the Windows Client
-   split Web Client and Service Tier
-   and a lot lot more

You can really configure the service tier inside the container the way you want. Some things can be done just by specifying a parameter, some things might require PowerShell code.

The idea is that I will write up a series of blog posts on how to achieve a number of these things.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
