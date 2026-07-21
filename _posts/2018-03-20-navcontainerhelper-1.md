---
layout: post
title: "NavContainerHelper"
date: 2018-03-20 03:23:33
categories: ["Docker", "NavContainerHelper", "Not Archived"]
tags: ["Docker", "NAV on Docker", "NavContainerHelper", "new-navcontainer"]
permalink: /2018/03/20/navcontainerhelper-1/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

# What are Containers? What is Docker?

If you are new to Docker and Containers, please read [this document](https://docs.microsoft.com/en-us/virtualization/windowscontainers/about/), which describes what Containers are and what Docker is.

If you want more info, there are a lot of [Channel9 videos on Containers as well](https://channel9.msdn.com/Search?term=containers#ch9Search&lang-en=en&pubDate=year)

If you have problems with Docker (not NAV related), the [Windows Containers Docker forum](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers) is the place you can ask questions (read the readme first).

# Get started – Install Docker

In order to run a NAV container, you need a computer with Docker installed, this will become your Docker host. Docker runs on Windows Server 2016 (or later) or Windows 10 Pro.

When using Windows 10, Containers are always using Hyper-V isolation with Windows Server Core. When using Windows Server 2016, you can choose between Hyper-V isolation or Process isolation. Read more about this [here](https://docs.microsoft.com/en-us/virtualization/windowscontainers/about/).

I will describe 3 ways to get started with Containers. If you have a computer running Windows Server 2016 or Windows 10 – you can use this computer. If not, you can deploy a Windows Server 2016 with Containers on Azure, which will give you everything to get started.

## Windows Server 2016 with Containers on Azure

In the Azure Gallery, you will find an image with Windows Server 2016 and Docker installed and pre-configured. You can deploy this image by clicking [this link](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Flive%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json).

**Note**, do not select Standard\_D1 (simply not powerful enough) – use at least Standard\_D2 or Standard\_D3.

In this VM, you can now run all the docker commands, described in this document.

## Windows Server 2016

Follow [these steps](https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/quick-start-windows-server) to install Docker on a machine with Windows Server 2016.

## Windows 10

Follow [these steps](https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/quick-start-windows-10) to install Docker on Windows 10.

# Get Started – Install NavContainerHelper

NavContainerHelper is a PowerShell module from the PowerShell Gallery, you can read more information about it [here](https://www.powershellgallery.com/packages/navcontainerhelper).

The module contains a number of PowerShell functions, which helps running and interactig with NAV containers.

On your Docker host, start PowerShell ISE and run:

```
install-module navcontainerhelper -force
```

Use

```
get-command -Module navcontainerhelper
```

to list all functions available in the module. Use

```
Write-NavContainerHelperWelcomeText
```

in order to list the functions in the module grouped into areas.

# Get started – run your first NAV container

Start PowerShell ISE and run this command:

```
New-NavContainer -accept_eula -containerName "test" -auth NavUserPassword -imageName "microsoft/dynamics-nav"
```

to run your first NAV container using NavUserPassword authentication. PowerShell will pop up a dialog and require you to enter a username and a password to use for the container.

New-NavContainer will remove existing containers with the same name before starting a new container. The container will be started as a process and the output of the function will be displayed in the PowerShell output window.

```
PS C:\Users\freddyk> New-NavContainer -accept_eula -containerName "test" -auth NavUserPassword -imageName "microsoft/dynamics-nav"
Creating Nav container test
Using image microsoft/dynamics-nav
NAV Version: 11.0.20783.0-w1
Generic Tag: 0.0.5.3
Creating container test from image microsoft/dynamics-nav
Waiting for container test to be ready
Initializing...
Starting Container
Hostname is test
PublicDnsName is test
Using NavUserPassword Authentication
Starting Local SQL Server
Starting Internet Information Server
Creating Self Signed Certificate
Self Signed Certificate Thumbprint 0A4F70380C95876A708018EA6883CA3A1F7FF72D
Modifying NAV Service Tier Config File with Instance Specific Settings
Starting NAV Service Tier
Creating DotNetCore NAV Web Server Instance
Creating http download site
Creating Windows user admin
Setting SA Password and enabling SA
Creating admin as SQL User and add to sysadmin
Creating NAV user
Container IP Address: 172.19.144.74
Container Hostname  : test
Container Dns Name  : test
Web Client          : http://test/NAV/
Dev. Server         : http://test
Dev. ServerInstance : NAV

Files:
http://test:8080/al-0.12.17720.vsix

Initialization took 59 seconds
Ready for connections!
Reading CustomSettings.config from test
Creating Desktop Shortcuts for test
NAV container test successfully created
```

The New-NavContainer will use the docker command to run the container, and will be building up the needed parameters for docker run dynamically, based on the parameters you specify to New-NavContainer.

You can also use docker run to run a container yourself

```
docker run -e ACCEPT_EULA=Y microsoft/dynamics-nav
```

**Note**, if you are running _Windows 10_ , you will have to add _–memory 4G_ as an extra parameter to the docker run command above (and in all docker run commands in this doc.)

This will start a container with a random name, a random password, using SSL with a self-signed certificate and using admin as the username. The prompt in which you are running the command will be attached to the container and the output will be displayed. You can add a number of extra parameters to specify password, database connection, license file, configurations, etc. etc.

The docker run command created by the New-NavContainer call above will be something like:

```
docker run --name test `
           --hostname test `
           --env auth=NavUserPassword `
           --env username="admin" `
           --env ExitOnError=N `
           --env locale=en-US `
           --env licenseFile="" `
           --env databaseServer="" `
           --env databaseInstance="" `
           --volume "C:\ProgramData\NavContainerHelper:C:\ProgramData\NavContainerHelper" `
           --volume "C:\ProgramData\NavContainerHelper\Extensionstest\my:C:\Run\my" `
           --restart unless-stopped `
           --env useSSL=N `
           --env securePassword= `
           --env passwordKeyFile="c:\run\my\aes.key" `
           --env removePasswordKeyFile=Y `
           --env accept_eula=Y `
           --detach `
           microsoft/dynamics-nav
```

**Note**, if you are running **Windows 10**, New-NavContainer will automatically add –memory 4G to the docker run command.

all parameters starting with –env means that docker is going to set an environment variable in the container. This is the way to transfer parameters to the NAV container. All the –env parameters will be used by the PowerShell scripts inside the container. All the non –env parameters will be used by the docker run command.

The –name parameter specifies the name of the container and the –hostname specifies the hostname of the contianer.

–volume parameters shared folders from the docker host to the container and –detach means that the container process will be detached from the process starting it.

The NAV container images supports a number of parameters and some of them are used in the above output. All of these parameters can be omitted and the NAV container image has a default behavior for them.

As you might have noticed, the New-NavContainer transfers the password to the container as an encrypted string and the key to decrypt the password is shared in a file and deleted afterwards. This allows you to use Windows Authentication with your domain credentials in a secure way.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
