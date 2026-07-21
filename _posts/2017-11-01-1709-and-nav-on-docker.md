---
layout: post
title: "1709 and NAV on Docker"
date: 2017-11-01 12:34:47
categories: ["Docker", "Not Archived"]
tags: ["1709", "Docker", "NAV", "NAV on Docker", "Windows 10", "Windows Server 1709", "Windows Server 2016", "windowsservercore"]
permalink: /2017/11/01/1709-and-nav-on-docker/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

As of yesterday evening Windows Server version 1709 (with and without Containers) is available on Azure:  
[![](/assets/images/2017/1709-and-nav-on-docker/1709.png?w=917&h=556)](/assets/images/2017/1709-and-nav-on-docker/1709.png)

I thought it would be a good idea to describe how this relates to the NAV on Docker images recently published on the Docker Hub.

Before diving into the details, please read [this blog post](https://docs.microsoft.com/en-us/windows-server/get-started/semi-annual-channel-overview) which talks about the Windows as a Service model and how Microsoft builds, deployed and services Windows. The model talks about the Long-term Servicing Channel (LTSC) and the Semi-Annual Channel.

The available release channels are different for each Windows Server installation option:

  
| Installation option | Semi-Annual Channel (Windows Server) | Long-term Servicing Channel (Windows Server 2016) |
| --- | --- | --- |
| Nano Server | Yes | No |
| Server Core | Yes | Yes |
| Server with Desktop | No | Yes |

Meaning, that if you deploy a Windows Server 1709 you will get a Server Core and will only be able to get a command prompt.

# No Desktop!

With No Desktop, I mean no meaningful desktop. You actually can remote desktop into an Azure VM with Windows Server 1709, you just get a desktop with a command prompt:  
[![](/assets/images/2017/1709-and-nav-on-docker/90263-corerdp-1.png)](/assets/images/2017/1709-and-nav-on-docker/90263-corerdp.png)

and if you press Ctrl+Alt+End you get a good old “last century style” menu:[![](/assets/images/2017/1709-and-nav-on-docker/cc4b6-coremenu-1.png)](/assets/images/2017/1709-and-nav-on-docker/cc4b6-coremenu.png)

The Task Manager you get from selecting Task Manager in the menu gives you valuable info about your server and services.  
[![](/assets/images/2017/1709-and-nav-on-docker/2ddb3-coretaskmanager-1.png)](/assets/images/2017/1709-and-nav-on-docker/2ddb3-coretaskmanager.png)

The Task Manager does provide one cute extra feature, which is very useful: File -> Run New Task

[![](/assets/images/2017/1709-and-nav-on-docker/46fa7-corenewtask-1.png)](/assets/images/2017/1709-and-nav-on-docker/46fa7-corenewtask.png)

Launch that in order to start another command or PowerShell prompt.

# microsoft/windowsservercore

If you look at the tags on [Docker Hub](https://hub.docker.com/r/microsoft/windowsservercore/tags/) for windowsservercore, you will find 2 major versions:

-   1709
-   10.0.14393

As described in [this blog post](https://freddysblog.com/2017/10/31/what-is-docker-what-are-containers/), there are 2 ways of running the NAV on Docker images.

-   Hyper-V containers, with kernel level/**hyperv isolation** (meaning the image has its own kernel)
-   Windows Server containers, with **process isolation** (meaning the image shares the kernel with the host).

As you can imagine, shared Kernel is only possible if the host kernel is the same major version as the container base image. Fortunately, both Windows 10 1709 and Windows Server 1709 does support running “old” 14393 containers under Hyper-V.

Read [this blog post](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility) for deeper description on the version compatibility. This gives us this compatibility matrix (gray column is the base image of the NAV on Docker images):

  
| 
Host OS Container base image

 |      windowsservercore 10.0.14393.x |      windowsservercore 1709 |
| --- | --- | --- |
| Windows 10 | Yes (with Hyper-V) | No |
| Windows 10 1709 | Yes (with Hyper-V) | Yes (with Hyper-V) |
| Windows Server 2016 | Yes | No |
| Windows Server 1709 | Yes (with Hyper-V) | Yes |

As you know, we are supporting NAV on Docker for development and test purposes, meaning we are in the right column.

# So why should we look at 1709?

Well, there are multiple reasons. Beside the obvious ones, that we are Microsoft, and we should follow the versions, there also have been significant improvements to the containers feature in 1709 and images have been on a major diet:  
[![](/assets/images/2017/1709-and-nav-on-docker/c5d39-images-1.png)](/assets/images/2017/1709-and-nav-on-docker/c5d39-images.png)

I am also confident that Azure Container Services is running 1709 as host and the more serious we get with Azure Container Services, the more urgent 1709 will become.

So yes, of course we will be creating 1709 images going forward, but as you probably can understand, we are not in a rush, but we will be looking into this and discuss what we need to do.

You can of course use 1709 if you want to, but for now you will not get any significant benefits if all you want to do is to run NAV on Docker images.

But… – this is not the last you heard of 1709.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
