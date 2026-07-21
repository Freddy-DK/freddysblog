---
layout: post
title: "NavContainerHelper 0.3.1.0 and a new Docker Generic build 0.0.6.6"
date: 2018-09-03 10:35:02
categories: ["Docker"]
tags: ["Docker", "Generic Image", "NAV on Docker", "NavContainerHelper"]
permalink: /2018/09/03/navcontainerhelper-0-3-1-0-and-a-new-docker-generic-build-0-0-6-6/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

Over the weekend, a new version of the NavContainerHelper ([version 0.3.1.0](https://www.powershellgallery.com/packages/navcontainerhelper/0.3.1.0)) has been uploaded to the PowerShell Gallery with a few bug fixes and a few extra functions. Also a new version of the Generic image (microsoft/dynamics-nav:generic-0.0.6.6) has been published and all NAV images (2016, 2017 and 2018) are being rebuild.

Business Central Sandbox images are automatically based on the newest generic image. Older images are not rebuild.

# Hosting endpoints on different ports

Up until now, the NAV Containers have always had fixed endpoints with the option of mapping these endpoints to different public ports on the host. A number of people found this hard to setup, and we have found out, that the Client Services Port (the port, which the Windows Client connects to) cannot be used with port mapping.

So, we decided to make it easier:-)

All docker images with generic build 0.0.6.5 or newer have some new parameters (environment variables), which allow you to change the endpoint port on the container. With this, you do not need to do any port mappings, instead you can just specify the port on which the container is listening.

The environment variables are:

**WebClientPort** defines the port on which the Web Client will be listening. Default is 80 or 443 (depending on whether or not you are using SSL).

**ClientServicesPort** defines the port on which the Service Tier is hosting Client Services (Windows Client connection port). Default is 7046.

**SoapServicesPort** defines the port on which the Service Tier is hosting Soap Services. Default is 7047.

**ODataServicesPort** defines the port on which the Service Tier is hosting OData Services and API Services. Default is 7048.

**DeveloperServicesPort** defines the port on which the Service Tier is hosting Developer Services (VS Code connection port). Default is 7049.

So, when using Docker Run, you will have to use:

\--env ClientServicesPort=7146

to make the container listen for Client Services on another port.

If you are using New-NavContainer in NavContainerHelper, you can use

\-ClientServicesPort 7146

You do not need to add this to the additionalParameters array.

The “old” way of using port mapping still works like before.

# New Functions

**Add-FontsToNavContainer** will copy one or more fonts from the container host to a container. A number of fonts (including some Chinese and Korean fonts) are not included in WindowsServerCore and as such are missing when printing reports which includes characters in the languages. This function will copy the fonts from the container host and register them in the container.

**Generate-SymbolsInNavContainer** shouldn’t really be needed, but it will re-generate all symbols in the container.

# Time Zone

A strange bug (which must be in Docker) resulted in an error when trying to invoke a CodeUnit inside a Container. The reason was, that the Container seems to be started with a time zone, which didn’t exist. Somehow the name and id of the timezone was translated into the local language, but when requesting time zones in the container,  the time zones were not translated. The original bug is [here](https://github.com/Microsoft/navcontainerhelper/issues/153).

# New generic image

The new generic image comes with a few updates, but most significantly is really the updated WindowsServerCore dependency. The new image comes with an updated WindowsServerCore and all images since NAV 2016 RTM are being rebuilt with this. This might be the last time where we automatically rebuild all images. In the future, we might just build new images on new versions of the generic layer.

Note that you can still run the old versions if you specify the accept\_outdated flag.

# Windows Server 2019 Insider Preview

Windows Server 2019 is available as Insider builds. Read more [here](https://blogs.windows.com/windowsexperience/2018/08/28/announcing-windows-server-2019-insider-preview-build-17744).

All NAV images are still based on ltsc2016 (Windows Server Long Time Servicing Channel 2016) as this is the image, which is compatible with most operating systems. We will likely adopt the next ltsc release when it is out and build our images for both versions, which will also work with Windows 10 at that time.

The generic image is available for 1709 (microsoft/dynamics-nav:generic-0.0.6.6-1709) and 1803 (microsoft/dynamics-nav:generic-0.0.6.6-1803), but the individual images will not be build for these versions.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
