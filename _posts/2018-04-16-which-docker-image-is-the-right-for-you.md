---
layout: post
title: "What Docker Image is right for you?"
date: 2018-04-16 16:07:25
categories: ["AL Development", "Docker"]
tags: ["bcsandbox", "bcsandbox-master", "Docker", "dynamics-nav", "NAV on Docker", "support"]
permalink: /2018/04/16/which-docker-image-is-the-right-for-you/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

The last year has been quite a journey for people using Docker images for Microsoft Dynamics NAV or Dynamics 365 Business Central. Images have been available in various places and private registries, navdocker, developer preview, navinsider, microsoft/dynamics-nav are just some of the terms you have run into.

This blog post should demystify and explain clearly which Docker Image is the right for you in a given situation – and where to get it.

# Developing for Microsoft Dynamics NAV

If you are developing for Microsoft Dynamics NAV, you will find Docker images on the public Docker Hub under [microsoft/dynamics-nav](https://hub.docker.com/r/microsoft/dynamics-nav/).

In the public Docker hub, you will find all cumulative updates to NAV 2016, 2017 and 2018 in all country versions and you can use the images simply by specifying the right tag. The tagging strategy used in microsoft/dynamics-nav is:

microsoft/dynamics-nav:version-cu-country

where

-   version is 2016, 2017 or 2018 (default is 2018)
-   cu is rtm, cu1, cu2, … (default is latest)
-   country is w1, dk, de, nl, na, … (default is w1)

Image name examples:

microsoft/dynamics-nav
microsoft/dynamics-nav:2018-cu3-de
microsoft/dynamics-nav:2017
microsoft/dynamics-nav:2016-dk

You will also be able to run earlier versions of Microsoft Dynamics NAV using the generic image as explained [here](https://freddysblog.com/2017/11/29/can-i-run-nav-2015-and-earlier-on-docker/).

# Developing for Dynamics 365 Business Central

If you are developing for the current version of Dynamics 365 Business Central, you will find Docker images on the public Docker Hub under [microsoft/bcsandbox](https://hub.docker.com/r/microsoft/bcsandbox/).

You will find the current version of Dynamics 365 Business Central using

microsoft/bcsandbox:build-country

where

-   build is the build number (default is current version)
-   country is w1, dk, us, ca, de, … (default is w1)

Image name examples:

microsoft/bcsandbox
microsoft/bcsandbox:us
microsoft/bcsandbox:dk
microsoft/bcsandbox:12.0.21229.0-us

You should normally never use the image with a specific build number unless instructed to do so. This is primarily used when you spin up Container Sandbox images from within Dynamics 365 Business Central (page search for Sandbox).

# Maintaining an app in AppSource for Dynamics 365 Business Central

If you have published an app in AppSource, you should continuously test that the app works with the next version of Dynamics 365 Business Central. You will be able to get insider builds of Business Central from a private registry called bcinsider.azurecr.io and the credentials for this private registry is available through [Microsoft Collaborate](http://developer.microsoft.com/en-us/dashboard/collaborate).

The insider builds are normally updated daily and the Dynamics 365 Business Central servers are updated monthly to this version. When the Dynamics 365 Business Central servers are updated, the image will also be deployed on the public docker hub under microsoft/bcsandbox (see previous section).

The image name follows the same tagging strategy as the public Dynamics 365 Business Central images:

bcinsider.azurecr.io/bcsandbox:build-country

where

-   build is the build number (default is latest version)
-   country is w1, dk, us, ca, de, … (default is w1)

Image name examples:

bcinsider.azurecr.io/bcsandbox
bcinsider.azurecr.io/bcsandbox:us
bcinsider.azurecr.io/bcsandbox:dk
bcinsider.azurecr.io/bcsandbox:12.1.21581.0-nl

You should never use the image with a specific build number unless instructed to do so. Setup Continuous Integration and Continuous Deployment by pulling the daily update of the Dynamics 365 Business Central Sandbox Container image.

# Developing for a future release of Dynamics 365 Business Central

Much like the strategy for Windows and other Microsoft services, Dynamics 365 Business Central will receive major updates semi-annually. If you are developing an app for AppSource targetting the next major update or if you need cutting edge functionality directly from the lab, you will be able to get insider builds of Business Central from a private registry called bcinsider.azurecr.io and the credentials for this private registry is available through [Microsoft Collaborate](http://developer.microsoft.com/en-us/dashboard/collaborate).

The insider builds are normally updated daily and the Dynamics 365 Business Central servers are updated semi annually to this version. When the Dynamics 365 Business Central servers are updated, the image will also be deployed on the public docker hub under microsoft/bcsandbox and will receive updates monthly (see previous section).

Please be aware that these insider builds might be more unstable than builds from the previous sections. You might see new functionality being developed over multiple days and upgrade procedures between versions might not be working smoothly. Please only use builds from this branch if you have a reason to do so.

The image name follows the same tagging strategy as the public Dynamics 365 Business Central images, but with a different namespace:

bcinsider.azurecr.io/bcsandbox-master:build-country

where

-   build is the build number (default is latest version)
-   country is w1, dk, us, ca, de, … (default is w1)

Image name examples:

bcinsider.azurecr.io/bcsandbox-master
bcinsider.azurecr.io/bcsandbox-master:us
bcinsider.azurecr.io/bcsandbox-master:dk
bcinsider.azurecr.io/bcsandbox-master:12.1.21581.0-nl

You should never use the image with a specific build number unless instructed to do so.

# Support?

If you encounter issues with Microsoft Dynamics NAV or with the current release of Dynamics 365 Business Central, you must report issues through the Dynamics Support team. You can open Support Request to CSS through PartnerSource portal ([https://mbs2.microsoft.com/Support/SupportRequestStep1.aspx](https://mbs2.microsoft.com/Support/SupportRequestStep1.aspx)) or contact your Service Account Manager (SAM) in the local subsidiary to understand what is included in your contract as of support incident and PAH (Partner Advisory Hours). Your SAM might also direct you step-by-step how to open a support request or how to get credentials if this is the first time you or your company are engaging Support.

If you encounter issues which are specific to the insider builds of Dynamics 365 Business Central, you should report these on [Github AL issues](https://github.com/Microsoft/AL/issues).

In the near future, there will be a blog post on the [NAV team blog](https://community.dynamics.com/business/b/businesscentraldevitpro) explaining the above in more detail.

If you have issues running the simplest NAV on Docker container (docker run -e accept\_eula=Y -m 3G microsoft/dynamics-nav) you should troubleshoot your infrastructure. A lot of frequently encountered issues can be solved be reading [this blog post](https://freddysblog.com/2017/10/29/troubleshooting-nav-on-docker/). You can also download a Container Host Debug PowerShell script here: [http://aka.ms/debug-containerhost.ps1](http://aka.ms/debug-containerhost.ps1) to troubleshoot issues with the container host.

If you have issues running NAV on Docker or Business Central Sandbox Containers, which you think might be related to problems in the Container images, please report these on [Github nav-docker issues](https://github.com/Microsoft/nav-docker/issues).

If you have issues running NAV on Docker or Business Central Sandbox Containers using navcontainerhelper, which you think might be related to problems in navcontainerhelper, please report these on [Github navcontainerhelper issues](https://github.com/Microsoft/navcontainerhelper/issues).

If you have issues running NAV on Docker or Business Central Sandbox Containers in Azure VMs using the ARM templates ([http://aka.ms/getnav](http://aka.ms/getnav), [http://aka.ms/bcsandbox](http://aka.ms/bcsandbox), etc.), which you think might be related to problems in the ARM templates, please report these on [Github nav-arm-templates issues](https://github.com/Microsoft/nav-arm-templates/issues).

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
