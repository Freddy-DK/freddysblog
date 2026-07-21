---
layout: post
title: "Docker and Business Central"
date: 2021-10-30 22:57:16
categories: ["Uncategorized"]
permalink: /2021/10/30/docker-and-business-central/
---

Over the last few months, there has been quite a few blog posts and discussions on social media based on blogs posts from Docker, Microsoft and Mirantis indicating new pricing structure. In this blog post I will try to make the options for running Business Central on Docker clear.

The two major products used for running Business Central containers are **[Docker Desktop](https://docs.docker.com/desktop/)** and **[Mirantis Container Runtime](https://www.mirantis.com/software/container-runtime/)** (formerly known as **Docker Engine – Enterprise**). Both products include **[Docker Engine](https://docs.docker.com/engine/)** and adds a number of additional components and features which might be relevant for you. Docker Engine can however also be installed alone and is sufficient for running BcContainerHelper and the Business Central Generic image.

This blog post will NOT just tell you how to install Docker Engine, as that might not be the right option for you. Instead, I will walk through the different options you have and you will have to decide on what is best for you.

## Docker Desktop

On August 31st, Docker announced [updated product subscriptions](https://www.docker.com/blog/updating-product-subscriptions/), which means that for companies with more than 250 employees using Docker and companies with a revenue of more than 10mio US$, Docker Desktop is no longer free, but will have a per user pricing. Docker Desktop Subscription overview can be found [here](https://docs.docker.com/subscription/). The new subscription includes a grace period until January 31st 2022, after which you need to conform to the new licensing rules or uninstall Docker Desktop.

Official installation instructions for Docker Desktop can be found [here](https://docs.docker.com/desktop/windows/install/).

For most people running Windows Desktop, I recommend using Docker Desktop. Even It comes with an installer and a nice GUI with some tools for helping you out if Docker acts out.

Personally, I am subject to the licensing cost (Microsoft does have slightly more than 250 employees AND we have slightly more than $10mio in revenue) and even though **I do not see $5 a month as a problem for using an amazing tool like Docker Desktop**, I right now do not have a way to pay as the licensing will be calculated for all of Microsoft.

If Microsoft signs an enterprise agreement of some kind with Docker before January 31st, I will re-install Docker Desktop. Until then, I will be installing Docker Engine on my machine.

A comparison between Docker Desktop and Docker Engine can be found [here](https://www.docker.com/products/docker-desktop/alternatives).

## Mirantis Container Runtime (formerly Docker Engine – Enterprise)

Shortly after, on September 27th, [Microsoft announced](https://techcommunity.microsoft.com/t5/containers/updates-to-the-windows-container-runtime-support/ba-p/2788799) that support for Mirantis Container Runtime (formerly known as Docker Engine – Enterprise) will transition to Mirantis support services. On the same day, [Mirantis acknowledged](https://www.mirantis.com/blog/windows-server-container-users-mirantis-is-here-to-support-you/) this and mentioned a pricing structure for Mirantis Container Runtime and indicating that support for this would not be included in standard pricing and would be subject to additional pricing. Pricing is currently in beta and might be subject to change.

Microsoft will no longer maintain the [MicrosoftDockerProvider repository](https://github.com/OneGet/MicrosoftDockerProvider), which then also means that the [DockerMsftProvider PowerShell module](https://www.powershellgallery.com/packages/dockermsftprovider) will no longer be supported and if you are using this module, you will need to find a different way of installing Docker.

Official installation instructions for Mirantis Container Runtime can be found [here](https://docs.mirantis.com/mcr/20.10/install/mcr-windows.html).

Mirantis Container Runtime is positioned as enterprise scale container runtime and are primarily focusing on functionality targeting this segment. I have used Docker Engine – Enterprise on Windows Server, but primarily because it was available using the DockerMsftProvider PowerShell module and it was free on Windows Server.

I do not see a problem in the pricing model of Mirantis Container Runtime. In fact, **I think it is a very good deal** if you are using any of the components Mirantis Container Runtime provides. BcContainerHelper and the Business Central generic image isn’t utilizing any of the added functionality provided in Mirantis Container Runtime and going forward, I will be installing Docker Engine on Windows Server.

## Docker Engine

[Docker Engine](https://docs.docker.com/engine/) consists of Docker Daemon and Docker CLI (Command Line Interface). Both projects are Open Source, licensed under the [Apache 2.0 license](https://github.com/moby/moby/blob/master/LICENSE). Note that Docker Engine doesn’t come with a support plan. If you need a support plan, you need Docker Desktop or Mirantis Container Runtime.

The binaries for Docker Engine can be downloaded from Docker [here](https://download.docker.com/win/static/stable/x86_64/).

Looking at the Docker binaries, the .zip files basically consists of two files. Docker.exe (CLI) and DockerD.exe (Daemon). If you want to go even deeper, the source for the Docker Daemon is in [the moby project](https://github.com/moby/moby) and the Docker Client is in the [Docker CLI project](https://github.com/docker/cli). You can build both projects yourself if you like. The moby project is fairly straightforward, the CLI project is a bit more complex to get building.

Official installation instructions for Docker Engine can be found [here](https://docs.docker.com/engine/install/binaries/#install-server-and-client-binaries-on-windows).

Docker Engine can also be installed using Chocolatey, using the [Docker-Engine chocolatey package](https://community.chocolatey.org/packages?q=docker-engine) (as I learned from Chris Blank during my session @ Directions EMEA, thanks:-))

**Docker Engine contains all functionality needed for running BcContainerHelper and the Business Central generic image.**

Personally, I am not using the Chocolatey packages. Instead I have created a PowerShell script, which installs or updates Docker Engine on my machine. This script was created based on the information on the Docker Docs page.

Note that this script is NOT supported and I only recommend you to use this script if you understand what it does and can fix issues if any occurs on your machine.

**This PowerShell script can be found [here](https://github.com/microsoft/nav-arm-templates/blob/master/InstallOrUpdateDockerEngine.ps1).**

And, BTW – if something fails and you need to reset your docker installation, you can use [this script](https://github.com/microsoft/nav-arm-templates/blob/master/CleanupAfterDocker.ps1), which is explained in [this blog post](https://freddysblog.com/2018/12/11/clean-up-after-yourself-docker-your-mom-isnt-here/) from December 2018 – you cannot just remove the c:\\programdata\\docker folder manually.

Hope this helps

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
