---
layout: post
title: "Can I run NAV 2015 (and earlier) on Docker?"
date: 2017-11-29 12:37:22
categories: ["Docker", "NavContainerHelper", "Not Archived"]
tags: ["Docker", "Generic", "NAV 2013", "NAV 2013R2", "NAV 2015", "NAV on Docker", "navinstall"]
permalink: /2017/11/29/can-i-run-nav-2015-and-earlier-on-docker/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

NAV on Docker is here to stay, and a lot of partners have discovered how NAV on Docker can save a lot of time in their development processes. NAV ships and maintains Docker images for all NAV versions (all CUs, all localizations) since NAV 2016RTM. One of the questions I have been asked a lot is, whether we will ship images for older versions of NAV.

The answer to that question is **No**.

We are **not** going to build and publish 38 (RTM +37 CU’s) for NAV 2015 times 20 localizations totalling a whooping 760 images to the Docker hub for NAV 2015, but…

# Ask the right question

As you might have noticed, the question in the title of this blog post is not whether Microsoft will build and publish Images – it is whether you can run NAV 2015 on Docker – and the answer to that question is **Yes**.

If you didn’t attend any of the presentations on NAV on Docker (at Directions US, Directions EMEA or NAV Tech Days), you should watch the [NAV Tech Days presentation on YouTube](https://www.youtube.com/watch?v=9c5Yl51yXb8). In the presentation you will find a slide explaining the layers in the NAV on Docker image:

[![](https://msdnshared.blob.core.windows.net/media/2017/11/dockerarchitecture1-1024x292.png)](/assets/images/2017/can-i-run-nav-2015-and-earlier-on-docker/60431-dockerarchitecture1.png)

All NAV 2016, NAV 2017 and Devpreviews images on the Docker hub are specific images. They have been build and published for your convenience. They all build on top of the Generic image and the Generic image is actually capable of running any version of NAV, if you just hand it the DVD (in some cases a little more)

# What?

Yes, you read it right. Run the Generic image and share a folder containing the NAV DVD to C:\\NAVDVD in the container and the Generic image will install NAV and run it. The simplest docker run command became a little more complex. If you have a complete NAV 2015 CU37 W1 DVD placed in c:\\temp\\nav2015, you can try this command:

docker run -e accept\_eula=Y -v c:\\temp\\nav2015:c:\\navdvd microsoft/dynamics-nav:generic

and you should see this output:  
[![](/assets/images/2017/can-i-run-nav-2015-and-earlier-on-docker/fc4f0-dockerrunnavdvd-1.png)](/assets/images/2017/can-i-run-nav-2015-and-earlier-on-docker/fc4f0-dockerrunnavdvd.png)

The installation (on my laptop) adds 85 seconds to the 34 seconds it takes to initialize the NAV Container which gives a total of 2 minutes to start an isolated instance of NAV 2015 CU37 on my laptop.

# WOW!

I think that is amazing, but there is more. All the other parameters from docker run still works, you can ask for ClickOnce deployment of the clients, you can override scripts and copy the Windows Client and the Classic Client to a host directory, you can run without SSL, with Windows Authentication etc. etc.

As I have said earlier – docker run is the “raw” way of running NAV on Docker. I expect most people to be using the navcontainerhelper to launch NAV on Docker. It is easier, it uses secure transport of credentials and it of course also supports the generic image – in fact, you don’t need to specify the imagename if you specify a navdvdpath, then the imagename is defaulted to microsoft/dynamics-nav:generic.

If you already have the navcontainerhelper installed, you need to update your navcontainerhelper from the PowerShell Gallery to version 0.2.1.1 (see the release notes [here](https://www.powershellgallery.com/packages/navcontainerhelper)), using:

Update-Module navcontainerhelper -force

and if you have the NAV 2015 CU37 W1 NAV DVD in c:\\temp\\nav2015, you should be able to run this PowerShell command:

New-NavContainer -accept\_eula \`
                 -containerName test \`
                 -navDvdPath "c:\\temp\\nav2015" \`
                 -navDvdCountry w1 \`
                 -updateHosts \`
                 -doNotExportObjectsToText \`
                 -includeCSide

Enter the Windows Credentials for the host machine and you should get an output like this:  
[![](/assets/images/2017/can-i-run-nav-2015-and-earlier-on-docker/f37a5-newnavcontaineroutput-1.png)](/assets/images/2017/can-i-run-nav-2015-and-earlier-on-docker/f37a5-newnavcontaineroutput.png)

and on my desktop, I have shortcuts for accessing my NAV 2015:  
[![](/assets/images/2017/can-i-run-nav-2015-and-earlier-on-docker/5b03a-shortcuts-1.png)](/assets/images/2017/can-i-run-nav-2015-and-earlier-on-docker/5b03a-shortcuts.png)

2 shortcuts to access the container and 3 to access NAV, all using Windows Authentication.

You can use all other parameters for new-navcontainer, and those that makes sense should work.

# NAV 2015 CU7 and earlier…

The further you go back in time, the more challenges you will have. NAV 2015 CU7, NAV 2013R2 and NAV 2013 all uses .NET 3 and that is not installed on WindowsServerCore, so trying to run NAV 2013R2 on Docker will give you this error:

Installing NAV
This NAV version requires .NET 3 which is not on WindowsServerCore.
If you download microsoft-windows-netfx3-ondemand-package.cab from a Windows Server 2016 media and place it in the Prerequisite Components folder on the NAV DVD, then it wi
ll be installed automatically.

I did not want to pre-install .NET 3 on the NAV on Docker generic image, I am sure you can understand that, so if you follow the instruction in the error message, you should be good to run earlier versions as well:  
[![](/assets/images/2017/can-i-run-nav-2015-and-earlier-on-docker/7f5a3-prereqs-1.png)](/assets/images/2017/can-i-run-nav-2015-and-earlier-on-docker/7f5a3-prereqs.png)

Now you should be able to run NAV 2013R2 on Docker as well, worked for me:  
[![](/assets/images/2017/can-i-run-nav-2015-and-earlier-on-docker/289de-2013r2-1.png)](/assets/images/2017/can-i-run-nav-2015-and-earlier-on-docker/289de-2013r2.png)

and indeed, I can start the Windows Client from my shortcut:  
[![](/assets/images/2017/can-i-run-nav-2015-and-earlier-on-docker/78345-about2013r2-1.png)](/assets/images/2017/can-i-run-nav-2015-and-earlier-on-docker/78345-about2013r2.png)

CSide won’t run on my machine, because I do not have the prerequisites installed, but should run as well.

Web Client doesn’t work with NAV 2013 – I did not spend any cycles to try to get that to work.

# If something sounds too good to be true…

So what’s the caveat here?

Did you just get support for NAV 2013, NAV 2013R2 and NAV 2015 on Docker?

The answer to that question is **No** – versions supported on Docker for test and development are still NAV 2016 and up.

Having said that, it will probably work for a lot of the scenarios you need (development and test) and if it doesn’t, I will tell you how you can fix the problem and I hope you will contribute whatever solution you have to problems for the benefit of all partners using NAV on Docker.

Microsoft does not perform any testing of earlier versions on Docker (except for what we did, that lead to this blog post).

# What to do if something doesn’t work

You can post issues on [http://www.github.com/Microsoft/nav-docker/issues](http://www.github.com/Microsoft/nav-docker/issues), other partners might have had the same issue and found a solution. If not, clone the nav-docker github repository and have a look in the corresponding navinstall.ps1 script (run71navinstall.ps1 is the installer for NAV 2013 R2).

In the following, I modified the first line in the script to (let’s say this was a fix to my problem):

Write-Host "Installing NAV MY WAY"

If you are using the navcontainerhelper, add the new navinstall.ps1 (or the folder in which it is) to the -myscripts parameter. This causes your navinstall.ps1 (or all files in the folder) to be placed in the my folder and overrides the base navinstall.ps1. This looks like this:

New-NavContainer -accept\_eula \`
                 -containerName test \`
                 -navDvdPath "c:\\temp\\nav2013R2" \`
                 -navDvdCountry w1 \`
                 -updateHosts \`
                 -doNotExportObjectsToText \`
                 -includeCSide \`
                 -myScripts @("C:\\Users\\freddyk\\Documents\\GitHub\\Microsoft\\nav-docker\\Run\\71")

The output now shows (captured before it was done):  
[![](/assets/images/2017/can-i-run-nav-2015-and-earlier-on-docker/df9af-myway-1.png)](/assets/images/2017/can-i-run-nav-2015-and-earlier-on-docker/df9af-myway.png)

If you are using docker run, you have to create a my folder yourself, place the navinstall.ps1 there and share this folder to c:\\run\\my in the container. Here, I just share the folder in which my navinstall is as my my folder:

docker run -e accept\_eula=Y -v c:\\temp\\nav2013r2:c:\\navdvd -v C:\\Users\\freddyk\\Documents\\GitHub\\Microsoft\\nav-docker\\Run\\71:c:\\run\\my microsoft/dynamics-nav:generic

and the output is:  
[![](/assets/images/2017/can-i-run-nav-2015-and-earlier-on-docker/18182-myway21-1.png)](/assets/images/2017/can-i-run-nav-2015-and-earlier-on-docker/18182-myway21.png)

There you are – I did it MY WAY!

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
