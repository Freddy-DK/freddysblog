---
layout: post
title: "Moving the generic images!"
date: 2020-12-06 07:47:03
categories: ["BcContainerHelper", "Docker"]
tags: ["Generic Image", "SQL Server 2017", "SQL Server 2019"]
permalink: /2020/12/06/moving-the-generic-images/
---

**UPDATE:** The tag of the SQL 2017 image is no longer {0}-0.1.0.25 – it is now {0}-sql2017.

A few years ago, when we started out shipping containers for NAV, we didn’t know that the name NAV was going away and Business Central would emerge.

In the Docker world, this becomes clear in a lot of ways.

-   The Generic image is called mcr.microsoft.com/dynamicsnav:<hostosversion>-generic
-   The PowerShell module was called NavContainerHelper
-   The first repo on Docker hub was microsoft/dynamics-nav
-   The projects on github are called microsoft/navcontainerhelper, microsoft/nav-docker and microsoft/nav-arm-templates
-   and I could probably continue…

This blog post announces the new location and tag of the generic image:

**mcr.microsoft.com/businesscentral:{0}**

where {0} is your host OS Version number, also explained in docker hub here: [https://hub.docker.com/\_/microsoft-businesscentral](https://hub.docker.com/_/microsoft-businesscentral)

The other pages on the docker hub: [microsoft/dynamics-nav](https://hub.docker.com/r/microsoft/dynamics-nav), [mcr/bconprem](https://hub.docker.com/_/microsoft-businesscentral-onprem) and [mcr/bcsandbox](https://hub.docker.com/_/microsoft-businesscentral-sandbox) all point to this new location and all references to the old outdated specific images should now be gone.

# SQL 2019

The new generic image has also been updated to contain SQL Express 2019 and the latest version (generic tag 1.0.1.2) also has support for setting ApplicationInsightsKey in customSettings.config (for single-tenant) or while mounting the tenants (for multi-tenantcy). Note that it was always possible to set ApplicationInsightsKey in app.json in extensions for specific app insights.

# SQL 2017

If your code somehow has a dependency on SQL 2017 or you find that something doesn’t work with the new images, I did also publish the “old” SQL 2017 images to the new location in:

**mcr.microsoft.com/businesscentral:{0\]-sql2017**

where {0} is your host OS version number. The SQL 2017 are updated with the same fixes images will only get emergency fixes as the SQL 2019 as per December 16th (1.0.1.2), but will likely not be maintained going forward. It is only there to help in case of emergency.

# How to use the new generic image

Easiest thing is probably just to update to BcContainerHelper 1.0.15 or later, then you will automatically switch to use the latest generic image in the new location, no need to change anything.

But… if you want to control which generic image you use for creating containers, you have 3 options:

1.  Specify **genericImage** in **bcContainerHelper.config.json** as described here: [https://freddysblog.com/2020/10/10/bccontainerhelper-configuration/](https://freddysblog.com/2020/10/10/bccontainerhelper-configuration/) this means that all PowerShell Sessions on this computer will use this generic image.
2.  Set **$bcContainerHelperConfig.genericImageName** (also described in the above blog) means that this PowerShell session will use this generic image.
3.  Specify **\-useGenericImage** to New-BcContainer (or **\-baseImage** to New-BcImage) means that this container/image will be created using that generic imag. (f.ex. use “$(Get-BestGenericImageName)-dev” to use insider generic images.

But again, if you just update BcContainerHelper, you probably won’t even notice that the image has moved and you are now using SQL 2019.

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
