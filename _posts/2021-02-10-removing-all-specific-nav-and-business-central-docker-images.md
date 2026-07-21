---
layout: post
title: "Removing all specific NAV and Business Central Docker Images"
date: 2021-02-10 16:01:39
categories: ["BcContainerHelper", "Docker"]
tags: ["Artifacts", "BcContainerHelper", "Generic Image", "Get-BcArtifactUrl", "Get-NavArtifactUrl", "NavContainerHelper", "Specific image"]
permalink: /2021/02/10/removing-all-specific-nav-and-business-central-docker-images/
---

On October 27th 2017, I posted the first announcement which introduced NAV on Docker. For approx. 3 years we published Docker images first to Docker Hub and later to Microsoft Container Registry for both Windows Server 2016 and Windows Server 2019. Many 1000 images was pushed to the container registries until we during the summer of 2020 changed strategy to use artifacts together with the generic image.

The various versions of specific images on different container OS’ amounts to more than 100Tb of docker images and now is the time to cleanup…

# Registries

In November 2020, we removed these two registries from the Docker Hub:

-   microsoft/dynamics-nav
-   microsoft/bcsandbox

and the 3 registries which are about to be removed are:

-   mcr.microsoft.com/businesscentral/sandbox (35267 tags)
-   mcr.microsoft.com/businesscentral/onprem (9393 tags)
-   mcr.microsoft.com/dynamicsnav (21567 tags)

and the only registry we will have left is the registry, which holds the generic images:

-   mcr.microsoft.com/businesscentral

# Docker Images and NavContainerHelper -> Artifacts and BcContainerHelper

If you are using BcContainerHelper and you have shifted to artifacts, you don’t need to do anything.

If you are still using NavContainerHelper you really need to switch, read more about BcContainerHelper here: [BcContainerHelper | Freddys blog](https://freddysblog.com/2020/08/11/bccontainerhelper/)

If you are still using the Specific Docker Images, you need to learn about artifacts and make the change asap, this blog post might be a good place to start: [Changing the way you run Business Central in docker… | Freddys blog](https://freddysblog.com/2020/06/25/changing-the-way-you-run-business-central-in-docker/).

The Specific Docker Images can be gone any day now.

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
