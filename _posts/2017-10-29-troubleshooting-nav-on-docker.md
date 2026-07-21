---
layout: post
title: "Troubleshooting NAV on Docker"
date: 2017-10-29 23:35:48
categories: ["Docker", "Not Archived"]
tags: ["Docker", "NAV on Docker", "Troubleshooting"]
permalink: /2017/10/29/troubleshooting-nav-on-docker/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

With the release of official NAV images on Docker Hub, we will probably see a larger uptake of people trying this great technology. In this post I have included the issues people are typically running into when trying out NAV on Docker. The first two topics are only relevant on Docker CE (Community Edition), which currently is the only Docker version you can install on Windows 10.

This list is by no means exhaustive and I will update this post if I discover other things, that people run into. Note that this troubleshooting post is not intended to cover advanced usage of NAV Containers, just cover the basic things to get you going. If you can start the simplest NAV Container and the advanced NAV Container setup doesn’t work, it is my assumption that the problem is somewhere in the configuration parameters or scripts.

# NAV on Docker Container doesn’t start!

## 1\. Is Docker setup for Windows Containers?

When running Docker CE for Windows 10, you have to switch Docker to Windows Containers (default is Linux Containers). Right click the Docker icon and select _Switch to Windows Containers_, like explained [here](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers).

## 2\. Are you giving the container enough memory?

By far, the most common error people face when running the NAV on Docker image is forgetting to allocate memory when running Hyper-V isolation (which is the **only option on Windows 10** at this time). When running Hyper-V isolation, Docker by default gives each container 1GB of memory and that is not enough for the NAV+SQL+IIS inside the NAV Container. The container will fail during initialization in random places, sometimes indicating that the problem is memory or that the NAV Service Tier is not running.

The solution to this problem is to insert a memory allocation (-m 4G or –memory 4G) in the docker run command. 4GB is more than enough and depending on what you are going to use NAV for, less could do. The full docker run command could look like:

```
docker run --env accept_eula=Y --memory 4G microsoft/dynamics-nav
```

A caveat here is of course to make sure that you have enough memory left on the host computer. You could also try with 3G or 2500M instead of 4G.

Also note [this bug](https://github.com/docker/for-win/issues/767) in Docker for Windows, which is under investigation, where a container can start with one memory setting and not with another.

## 3\. Is Docker properly installed?

The NAV image on Docker is a fairly complicated image and you can specify a lot of different parameters to configure NAV to suit your needs. Before filing an issue with a dump from the output on [http://www.github.com/microsoft/nav-docker/issues](http://www.github.com/microsoft/nav-docker/issues) please try the following:

```
docker run -it --name myserver --memory 4G microsoft/windowsservercore cmd
```

This should start a windowsservercore image and place you in a Cmd prompt inside the Container. Press Exit to exit the container. If this fails, then Docker is not properly installed on the machine. Use

```
docker rm myserver -f
```

to delete the container again.

## 4\. Is the network working?

Docker is doing an amazing job on the network side, typically it just works. I have however seen a number of people (including one of my machine) having one problem on the name resolution. Try to create a container with a specified hostname:

```
docker run -itd --name myserver --hostname myserver microsoft/windowsservercore cmd
```

Now, you should be able to ping myserver:

```
ping /4 myserver
Pinging myserver [172.19.152.52] with 32 bytes of data:
Reply from 172.19.152.52: bytes=32 time<1ms TTL=128
Reply from 172.19.152.52: bytes=32 time<1ms TTL=128
Reply from 172.19.152.52: bytes=32 time<1ms TTL=128
Reply from 172.19.152.52: bytes=32 time<1ms TTL=128
```

The problem I have on my machine is, when I connect to the Microsoft Corporate network, then Docker name resolution will stop working:

```
ping /4 myserver
Pinging myserver.redmond.corp.microsoft.com [10.137.86.122] with 32 bytes of data:
Request timed out.
Request timed out.
Request timed out.
Request timed out.
```

The only solution I have found to this problem is to add an entry to the hosts file in c:\\windows\\system32\\drivers\\etc to specify that 172.19.152.52 is myserver. The new-navcontainer function in the navcontainerhelper PowerShell module has a parameter called -updateHosts, which will do just that – update the hosts file.

Use

```
docker rm myserver -f
```

to delete the container again.

## 5\. Is the docker run syntax correct?

The docker syntax is:

```
docker
```

Note that the command is first and the image name last.

You cannot add options after the image name or insert options before the command.

## 6\. Does the LicenseFile parameter really point to a license file?

If you are using a secure Url to specify the license file, does it really yield a license file – or is it the Url to a page in which you can download the license file. If you are using DropBox (Copy DropBox Link) to create a secure Url, you will need to replace dl=0 with dl=1 like explained in [this](/2017/02/26/create-a-secure-url-to-a-file/) blog post.

If you are specifying a file path to the License file, then you need to make sure, that the license file is accessible from inside the container given that file path. If f.eks. you have placed the file in **c:\\temp\\license.flf** on the docker host – and you specify that the licensefile is in c:\\hosttemp\\license.flf, then you need to insert the -v c:\\temp:c:\\hosttemp as a parameter to your docker run command, for the temp folder to be accessible from inside the container as c:\\hosttemp.

```
docker run -e accept_eula=Y -e licensefile=c:\hosttemp\license.flf -v c:\temp:c:\hosttemp -m 4G microsoft/dynamics-nav
```

## 7\. Are you running an Anti-Virus program, which prevents Docker from working?

I have only heard this from one partner, but wanted to include it here as more people might run into this.

As described in [this](https://github.com/moby/moby/issues/30296) github issue, Anti-Virus software can prevent Docker from working and there is not a whole lot Docker or Microsoft can do about that. In the specific github issue, upgrading the Anti-Virus program to the latest version actually solved the problem, but there are links indicating people having problems with other Anti-Virus programs.

Microsoft also published an [article](https://msdn.microsoft.com/en-us/windows/hardware/drivers/ifs/anti-virus-optimization-for-windows-containers) about ways to configure/optimize your Anti-Virus program for Docker on Windows.

Best mitigation for this issue is really to contact your Anti-Virus program provider and ask them what they are going to do about the problem.

## 8\. Are you running Windows Server 1709?

If you are running Windows Server 1709 you need to run the normal NAV on Docker images using Hyper-V isolation.

```
docker run -e accept_eula=Y --isolation hyperv --memory 4G microsoft/dynamics-nav:devpreview
```

When we start shipping 1709 images, you will be able to use those with process isolation. For NAV on Docker images, there is no benefit of running Windows Server 1709 at this time.

## 9\. Are you using a container host name, which is more than 15 characters long?

If you are using a very long name as the container host name, you might end up in problems

## 10\. Are you using transparent networks

If you want to run your containers using a transparent network you might have trouble connecting or getting an IP address through DHCP. This is basically caused by security mechanisms in the hypervisor which prevents a virtual network interface to get traffic which isn’t meant for it, i.e. which isn’t coming from or directed at its MAC address. Because a container in a transparent network has its own MAC address, the network interface on the switch won’t let the traffic from or to the container through and networking will fail. By enabling MAC address spoofing in Hyper-V or promiscuous mode in VMWare you tell your hypervisor that all network traffic should be made available to the network interface, which results in the network traffic from or for the container also getting through.

You can find information on how to enable MAC address spoofing on Hyper-V here ([https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/container-networking#tips–insights](https://na01.safelinks.protection.outlook.com/?url=https%3A%2F%2Fdocs.microsoft.com%2Fen-us%2Fvirtualization%2Fwindowscontainers%2Fmanage-containers%2Fcontainer-networking%23tips--insights&data=04%7C01%7CFreddy.Kristiansen%40microsoft.com%7C5af81cdcb4104687354708d53e02b111%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C636483103648397855%7CUnknown%7CTWFpbGZsb3d8eyJWIjoiMC4wLjAwMDAiLCJQIjoiV2luMzIiLCJBTiI6Ik1haWwifQ%3D%3D%7C-1&sdata=T8gQMCSzo5sfUabd4CPGCxF5tn5BC6Tpl44UwiUSceE%3D&reserved=0)) and on VMWare promiscuous mode here ([https://kb.vmware.com/s/article/1004099](https://na01.safelinks.protection.outlook.com/?url=https%3A%2F%2Fkb.vmware.com%2Fs%2Farticle%2F1004099&data=04%7C01%7CFreddy.Kristiansen%40microsoft.com%7C5af81cdcb4104687354708d53e02b111%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C636483103648397855%7CUnknown%7CTWFpbGZsb3d8eyJWIjoiMC4wLjAwMDAiLCJQIjoiV2luMzIiLCJBTiI6Ik1haWwifQ%3D%3D%7C-1&sdata=z5opV%2B%2FNsyOwQMM308FSOZp%2B3QjEGVcJvZ2L8VxHlyk%3D&reserved=0))

# NAV on Docker container starts, but you cannot connect?

First of all you need to realize that NAV on Docker is “just” the Service Tier and optionally the WebClient and SQL Server as well.

## 1\. You cannot connect to Web Client using Windows Authentication

Remember that you can always revert back to use NavUserPassword for the container if you cannot make Windows Authentication work.

### a. You might need to run Windows Update on the host.

Yes, it is not a joke. At the time where I am writing this, the NAV on Docker images are using WindowsServerCore 10.0.14393.1770.

When deploying a Windows Server 2016 with Containers on Azure, you might get version 10.0.14393.1715. If that is the case, then Windows Auth doesn’t work for the Web Client.

### b. Is the host computer domain joined and not connected?

If you are running a domain joined computer in cached mode (i.e. not connected to the domain), then Windows Authentication to the Web Client might not work. Try to connect to your domain physically or through a VPN and see if that solves the problem.

### c. Did you clear browser data / Run In-private?

Almost sounds like a farce, but if the Web Client please try to remove all browser data or run in private to see if the problem is with previously cached stuff in the browser.

## 2\. You cannot connect to the Classic Development Environment (CSide)

### a. Are you missing the prerequisites?

Whether you are running CSide using Click-Once, through the navcontainerhelper or you manually copied the binaries to a shared folder, you need to have the pre-requisites installed on the computer on which you are running CSide.

If CSide complains about missing DLL’s or giving an ODBC error, you probably need to install these pre-requisites:

1.  [https://download.microsoft.com/download/2/E/6/2E61CFA4-993B-4DD4-91DA-3737CD5CD6E3/vcredist\_x86.exe](https://download.microsoft.com/download/2/E/6/2E61CFA4-993B-4DD4-91DA-3737CD5CD6E3/vcredist_x86.exe)
2.  [https://download.microsoft.com/download/3/A/6/3A632674-A016-4E31-A675-94BE390EA739/ENU/x64/sqlncli.msi](https://download.microsoft.com/download/3/A/6/3A632674-A016-4E31-A675-94BE390EA739/ENU/x64/sqlncli.msi)

# You can connect to NAV on Docker, but it doesn’t work?

Now this sounds like a very very wide topic and I can in no way explain everything that goes wrong. I can however explain some of the things you might experience, where Docker causes things not to work or work differently

## 1\. You cannot modify tables when running CSide through ClickOnce

When you are running CSide through ClickOnce, there are a number of things you cannot do and changing tables is one of them. Running CSide through ClickOnce is primarily intended for viewing purposes.

The NavContainerHelper offers a way to share the binaries from the container to the Host and this way be able to run Classic Development towards the container.

## 2\. There are errors in the event-log

If you are running a financials localization under Docker, you might see an error indicating that AzureActiveDirectoryClientId is wrong and yes, it is. On Docker we currently do not support AAD authentication and since Financials is intended to only run AAD, you might see things like this for now.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
