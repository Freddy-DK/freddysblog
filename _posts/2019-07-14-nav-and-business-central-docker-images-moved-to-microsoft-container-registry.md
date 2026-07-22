---
layout: post
title: "NAV and Business Central Docker images moved to Microsoft Container Registry…"
date: 2019-07-14 23:07:45
categories: ["CI/CD", "Docker", "NavContainerHelper"]
tags: ["Docker", "NAV on Docker", "NavContainerHelper"]
permalink: /2019/07/14/nav-and-business-central-docker-images-moved-to-microsoft-container-registry/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

If you are using public Microsoft Dynamics NAV or Microsoft Dynamics 365 Business Central on Docker, this blog post contains important information.

First of all, I am sorry that I didn’t publish this blog post earlier. It was written 2 months ago and I actually thought that I published it, but apparently not. It was still in my list of draft blog posts:-( So – I wrapped it up – and here you are…

# Docker hub

If you have been using public Microsoft Dynamics NAV or Dynamics 365 Business Central Docker images, you will know that they have been published to Docker hub on **microsoft/dynamics-nav** and **microsoft/bcsandbox**. You will probably also have seen blog posts or other information describing that images are moving to the Microsoft Container Registry for better performance and better security.

We are currently not publishing images to Docker hub and we will likely remove the existing repositories. The latest images on Docker hub are from April 16th and is Microsoft Dynamics NAV 2018 CU16.

# Microsoft Container Registry

As of May 1st 2019, all new public images will only be available on the Microsoft Container registry – **mcr.microsoft.com**.

## Dynamics 365 Business Central On Premises Containers

The new address for Dynamics 365 Business Central on Premises containers is:

**mcr.microsoft.com/businesscentral/onprem:<tag>**

where **<tag>** describes the version you need and is composed of:

<version>-<cu>-<country>-<platform>

or

<appversion>-<country>-<platform>

where

**<version>** is

-   **1810** for the first version of Business Central on Premises shipped October 2018
-   **1904** for the second version of Business Central on Premises, shipped April 2019
-   **latest** for the latest version, which at this time is 1904 (default)

**<cu>** is

-   **rtm** for the initial version
-   **cu1** for cumulative update 1
-   …
-   **latest** for the latest cumulative update (default)

**<country>** is the localization you want

-   **dk** for Denmark
-   **de** for Germany
-   **na** for North America
-   …
-   **w1** for international (default)

**<platform>** is

-   **ltsc2016** for Windows Server Core 2016 based image (default)
-   **ltsc2019** for Windows Server Core 2019 based image

**<appversion>** is the application version number composed of 4 numbers (Major, Minor, Build and Release)

### Examples

-   **mcr.microsoft.com/businesscentral/onprem:1810-cu5-dk-ltsc2019**
    -   Danish cu5 of Dynamics 365 Business Central fall 2018 release for Server 2019
-   **mcr.microsoft.com/businesscentral/onprem:1904-rtm-ltsc2016**
    -   Spring 2019 Dynamics 365 Business Central w1 for server 2016
-   **mcr.microsoft.com/businesscentral/onprem:ltsc2019**
    -   latest Dynamics 365 Business Central on Premises w1 for Server 2019
-   **mcr.microsoft.com/businesscentral/onprem**
    -   latest Dynamics 365 Business Central on Premises w1 for Server 2016
-   **mcr.microsoft.com/businesscentral/onprem:14.0.29537.0-dk-ltsc2019**
    -   Danish Dynamics 365 Business Central version 14.0.29537.0 (spring 2019) for Server 2019

## Dynamics 365 Business Central Sandbox Containers

The address of containers for Dynamics 365 Business Central Sandbox is:

**mcr.microsoft.com/businesscentral/sandbox:<tag>**

where **<tag>** describes the version you need and is composed of:

<update>-<country>-<platform>

or

<appversion>-<country>-<platform>

where

**<update>** is

-   **update25** for the April 2019 update
-   **update26** for the May 2019 update
-   **latest** for the latest update i.e. current version online (default)

**<country>** is

-   **dk** for Denmark
-   **ca** for Canada
-   **us** for United States
-   …
-   **w1** for international (default)

**<platform>** is

-   **ltsc2016** for Windows Server Core 2016 based image (default)
-   **ltsc2019** for Windows Server Core 2019 based image

**<appversion>** is the application version number composed of 4 numbers (Major, Minor, Build and Release)

### Examples

-   mcr.microsoft.com/businesscentral/sandbox:ltsc2019
    -   latest Dynamics 365 Business Central Sandbox container for server 2019
-   mcr.microsoft.com/businesscentral/sandbox
    -   latest Dynamics 365 Business Central Sandbox container for server 2016
-   mcr.microsoft.com/businesscentral/sandbox:dk-ltsc2016
    -   Latest danish Dynamics 365 Business Central Sandbox container for server 2016

## Microsoft Dynamics NAV Containers

The new address for Microsoft Dynamics NAV containers is:

**mcr.microsoft.com/dynamicsnav:<tag>**

**Note**: dynamicsnav is without a hyphen on mcr

where **<tag>** describes the version you need and is composed of:

<version>-<cu>-<country>-<platform>

or

<appversion>-<country>-<platform>

where

**<version>** is

-   **2016** for Microsoft Dynamics NAV 2016
-   **2017** for Microsoft Dynamics NAV 2017
-   **2018** for Microsoft Dynamics NAV 2018
-   **latest** refers to the latest version, which is 2018 (default)

**<cu>** is

-   **rtm** for the initial version
-   **cu1** for cumulative update 1
-   …
-   **latest** for the latest cumulative update (default)

**<country>** is the localization you want

-   **dk** for Denmark
-   **de** for Germany
-   **na** for North America
-   …
-   **w1** for international (default)

**<platform>** is

-   **ltsc2016** for Windows Server Core 2016 based image (default)
-   **ltsc2019** for Windows Server Core 2019 based image

**<appversion>** is the application version number composed of 4 numbers (Major, Minor, Build and Release)

### Examples

-   **mcr.microsoft.com/dynamicsnav:2017-cu5-dk-ltsc2019**
    -   Danish cu5 of Microsoft Dynamics NAV 2017 for server 2019
-   **mcr.microsoft.com/dynamicsnav**
    -   Latest Microsoft Dynamics NAV w1 for server 2016
-   **mcr.microsoft.com/dynamicsnav:ltsc2019**
    -   Latest Microsoft Dynamics NAV w1 for server 2019
-   **mcr.microsoft.com/dynamicsnav:11.0.33812.0-dk-ltsc2019**
    -   Danish Microsoft Dynamics NAV version 11.0.33812.0 (NAV 2018 cu19) for server 2019

I hope this is helpful

_**Freddy Kristiansen**_  
Technical Evangelist
