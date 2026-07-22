---
layout: post
title: "Multiple ways to run a NAV on Docker image (NAV on Docker #5)"
date: 2017-11-03 08:40:23
categories: ["Azure", "Demo Environments", "Docker", "NavContainerHelper"]
tags: ["Azure", "Azure Container Services", "Containers", "Docker", "GETNAV", "NAV", "NAV on Docker", "NavContainerHelper"]
permalink: /2017/11/03/multiple-ways-to-run-a-nav-on-docker-image/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

If you haven’t read [this](/2017/10/29/it-has-never-been-easier-nav-on-docker-2/) blog post, then please do so before continuing here.

A lot of examples (like the prior blog post) will use docker run to start a NAV on Docker container, but there are actually a lot of different ways to start these containers. Some of these methods will run the container locally, some will spin up the container on Azure.

In the end, what you get is NAV running somewhere and you can connect, demo, use and develop using it.

In the following, I will go through a number of different ways to start a NAV on Docker. All the examples in this blog post will be very simple, but they will explain how to define parameters used to configure the container and how to share PowerShell scripts with the container, which can do more advanced configurations.

Upcoming blog posts will then attempt to explain how to achieve various scenarios through configuring NAV on Docker and whether you choose one or the other way of running Docker, you should be able to achieve these requirements.

# docker run

Docker run is the raw way of spinning up a new container based on a Docker image. Most of the other ways listed below will end up in a docker run command somewhere behind the scenes. The format of Docker run is:

```
docker run [options] image[:tag] [command] [args]
```

When running the NAV on Docker images, we don’t use command and args. As you might have seen in other places, as one of the options, you need to accept the eula using

```
-e accept_eula=Y
```

\-e is really short for –env, meaning that what you do here is to set an environment variable called **accept\_eula** to **Y**. This is the way you transfer parameters to the container.Inside the image, a script will check that accept\_eula indeed is set to Y before starting the container.

Parameters that can be transferred as environment variables to version 0.0.3.1 of the Docker Images are: Accept\_Eula, Accept\_Outdated, PublicDnsName, Auth, Username, Password, SecurePassword, PasswordKeyFile, RemovePasswordKeyFile, LicenseFile, BakFile, DatabaseServer, DatabaseInstance, DatabaseName, UseSSL, ClickOnce, WebClient, HttpSite, SqlTimeout, PublicWebClientPort, PublicFileSharePort, PublicSoapPort, PublicODataPort, PublicWinClientPort, Locale – a seperate blog post will describe what each of those means.

Note, that the environment variables are specific to the NAV on Docker Image.

Beside the environment variable parameters, there are a number of other options you can specify to Docker. Read more [here](https://docs.docker.com/engine/reference/run/).

If all these options, I will mention those, we frequently use when running NAV on Docker.

-   **–hostname (-h)** specifies the hostname of the container. This is the name you will use to connect to the containers web client and the name you can ping. If you do not specify the hostname, the first 12 characters of the container Id will be used as the hostname.
-   **–name** specifies the name of the container. The name is used when referring to the container with other docker commands. If the name is not specified, a random name will be generated using a verb and a name (like friendly\_freddy)
-   **–memory (-m)** specifies the max. amount of memory the container can use. The default for this option depends on how you are running the container. When you run Windows Server Containers there are no implicit memory option and the container can basically use all memory available to the host. When you run HyperV containers, the default max. memory is 1Gb.
-   **–volume (-v)** specifies a folder from the host you want to share with the container and the path inside the container, where you want to mount this folder. –volume c:\\myfolder:c:\\run\\my shares the existing c:\\myfolder on the host to the container and the content is in c:\\run\\my.
-   **–restart** specifies the restart options for the container.

A very typical docker run command could look like:

```
docker run -e accept_eula=Y --name test -h test -m 4G -e useSSL=N -e licensefile=c:\run\my\license.flf -v c:\myfolder:c:\run\my --restart always -e exitonerror=N -e locale=en-us microsoft/dynamics-nav:devpreview
```

Which will accept the eula and run the dynamics-nav:devpreview container with 4Gb of memory, test as the name and hostname, http (not https), restart option set to always, locale to en-US, and use the licensefile, which is located in c:\\myfolder\\license.flf on the host computer.

Docker run is the raw unfiltered experience, where you can do everything.

# navcontainerhelper

The navcontainerhelper is an open source PowerShell module, which is available on the PowerShell Gallery.

Source: [https://www.github.com/microsoft/navcontainerhelper](https://www.github.com/microsoft/navcontainerhelper)

PowerShell Gallery: [https://www.powershellgallery.com/packages/navcontainerhelper](https://www.powershellgallery.com/packages/navcontainerhelper)

On a Windows 10 or Windows Server 2016, start Powershell and write:

```
install-module navcontainerhelper -force
Write-NavContainerHelperWelcomeText
```

You should now see:  
[![](/assets/images/2017/multiple-ways-to-run-a-nav-on-docker-image/5ac5e-navcontainerhelper-1.png)](/assets/images/2017/multiple-ways-to-run-a-nav-on-docker-image/5ac5e-navcontainerhelper.png)

In order to see the syntax for the functions, you can use

```
help new-navcontainer -detailed
```

and you will get  see the syntax for new-navcontainer, the description for all parameters and some examples.

Try

```
new-navcontainer -accept_eula -containerName test -imageName microsoft/dynamics-nav:devpreview
```

The navcontainerhelper will create a folder on the C:\\ drive called DEMO and will place all files underneath that folder. The demo folder will be shared to the container for transfer of files etc. If you do not specify a username and a password, then it will ask for your password and use the current Windows username as the username. If you specify your windows password, the container setup will use Windows Authentication integrated with the host. The password will be transferred to the container in a secure way and the random 128bit key generated to encrypt the password will be deleted, in order to avoid that your domain password is exploited.

The navcontainerhelper will also create shortcuts on the desktop for the Web Client, a container prompt and a container PowerShell prompt.

The navcontainerhelper module also allows you to add the -includeCSide switch in order to add the windows client and CSide to the desktop and export all objects to a folder underneath C:\\DEMO\\Extensions in order for the Object handling functions from the module to work.

Try:

```
new-navcontainer -accept_eula -containername test -imageName microsoft/dynamics-nav:devpreview -includecside
(start CSide, add a field, add the field to a page)
Convert-ModifiedObjectsToAl -containername test
```

and you should have a v2 app with your field and your control.

There are a lot more you can do with the navcontainerhelper. Documentation coming up…

The new-navcontainer will predefine a lot of the options for docker run, but it allows you to specify additional parameters by adding:

```
-additionalParameters @("-e usessl=Y", "-e accept_outdated=Y")
```

You will also find a new-csidedevcontainer function, which is really just new-navcontainer with -includecside.

# [http://aka.ms/getnav](http://aka.ms/getnav)

The aka-ms-getnav is described in [this blog post](/2017/11/03/1-800-getnav/) and is probably the easiest way to spin up a NAV on Docker container on an Azure VM.

The Azure VM will have the Nav Container Helper installed and a shortcut on the desktop for the Nav Container Helper. The idea is, that the primary container (navserver) is used to v2 extension development and you can create another container for Cside Development and/or for converting objects from v1 extensions to v2 extensions.

Nav Container Helper provides functions for converting objects from a container to Deltas or Al as described in the previous section. Much more on this in upcoming blog posts + documentation.

# Azure Container Services

Tobias Fenster blogged about how to setup NAV on Azure Container Instances [here](https://www.axians-infoma.de/navblog/use-azure-container-instances-nav/).

Some things have changed since then:

-   NAV on Docker images are now available on the public Docker hub
-   Windows Server 1709 has been released
-   Kubernetes is being integrated in Azure Container Services
-   \+ a lot of other innovations happening as you read this…

So, I am sure that we are going to hear a lot more about Azure Container Services. If you think about it, NAV on Docker running on Azure Container Services in a kubernetes cluster using Azure SQL backend is after all true server-less computing, setup in minutes.

For now, I will continue my journey to enlighten and educate people about how to optimize their processes by using NAV on Docker for development and test.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
