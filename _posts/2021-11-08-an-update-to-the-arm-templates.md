---
layout: post
title: "An update to the ARM templates"
date: 2021-11-08 05:05:46
categories: ["Azure", "BcContainerHelper", "Docker"]
tags: ["ARM templates", "Docker"]
permalink: /2021/11/08/an-update-to-the-arm-templates/
---

It’s been a while since I last blogged about the ARM templates. Mostly because they just live a life of their own and just works. An average of 20 times a day, some partner somewhere in the work is using [https://aka.ms/getbc](https://aka.ms/getbc) to create an Azure VM with Business Central running in a container (or one of the other ARM templates) directly from a browser. On top of this are all the partners who are automating the process of creating Azure VMs with a PowerShell script. A few things are changing, but functionality stays the same.

## Configurable Container Name

![](/assets/images/2021/an-update-to-the-arm-templates/containername.png)

Since the making of the ARM templates, the container name has been navserver. With the latest update, the default container name changes to bcserver for getbc and getbcext. For getnav and getnavext, it remains navserver. If your code or scenarios rely on the name being navserver, you should change the setting to navserver.

## Docker engine instead of Docker EE

As described in [this blog post](/2021/10/30/docker-and-business-central/), the DockerMsftProvider with Docker EE will cease to exist in approx. one year from now. With this update, the Azure VMs created by getbc and the other ARM templates will use Docker Engine (installed by [this installation script](https://github.com/microsoft/nav-arm-templates/blob/master/InstallOrUpdateDockerEngine.ps1)) if docker wasn’t already present on the base image. Note that _Windows Server 2019 with Containers_ already contains docker, and that will not be replaced with Docker Engine. I don’t know what the future of that base image is.

## Windows Server 2022 support

![](/assets/images/2021/an-update-to-the-arm-templates/os.png)

Windows Server 2016 was removed and Windows Server 2022 was added as supported VM platform.

## Just-in-time traefik image

When adding support for Windows Server 2022, I found that the [traefik image](https://hub.docker.com/_/traefik) currently only exists for Windows Server 2019. Running the 1809 traefik container in Windows Server 2022 using hyperv didn’t work, so with this update, a traefik image is build using the same windows servercore image as the one used for Business Central (optimizing the amount of images to pull). The code for building the traefik image can be found [here](https://github.com/microsoft/nav-arm-templates/blob/4e71e6119532ae6fbf6050de7940854bf1676ec0/SetupVm.ps1#L133).

## Updated list of VM Sizes

The list of VM Sizes you can select has been updated to include more sizes and I have tested that all included sizes are available in all datacenters.

## [https://aka.ms/GetBuildAgent](https://aka.ms/GetBuildAgent)

As many partners will know, GetBuildAgent can be used to create an Azure VM, which includes a self-hosted agent for Azure DevOps. With this update, you can use the same template for creating agents for **GitHub Actions** and you also have a field determining the **number of build agents** you want to run on the Azure VM.

When creating agents for GitHub actions you need to specify a **token**, an **organization url** and an **agent url**. In the information popup with each field, you can see where to get the corresponding information.

![](/assets/images/2021/an-update-to-the-arm-templates/token.png)

## Windows 11

The ARM templates also have support for creating Windows 11 Azure VMs. I have however disabled this option due to [this bug](https://github.com/microsoft/navcontainerhelper/issues/2178). When I have a solution for the bug, I will enable Windows 11 as option for the ARM templates.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
