---
layout: post
title: "NavContainerHelper doesn’t work anymore…"
date: 2021-03-01 19:02:10
categories: ["BcContainerHelper", "NavContainerHelper"]
tags: ["BcContainerHelper", "NavContainerHelper"]
permalink: /2021/03/01/navcontainerhelper-doesnt-work-anymore/
---

With a total of over 300000 downloads, NavContainerHelper is no more. As of this weekend, NavContainerHelper cannot be used to create containers anymore. There has been a lot of blog posts here on my blog and others that indicated that this day would come and now it is here… – all specific images are gone and with those all the “old” generic images, which was used by NavContainerHelper.

I will contact PowerShell Gallery to get the entry removed.

RIP NavContainerHelper, Long Live BcContainerHelper.

# Shift to BcContainerHelper

The recipe for changing from NavContainerHelper to BcContainerHelper goes here:

1.  Remove all containers (Get-NavContainers | Remove-NavContainer)
2.  Uninstall all versions of NavContainerHelper (UnInstall-Module NavContainerHelper -allversions)
3.  Restart PowerShell and check that NavContainerHelper is gone (else repeat step 2)
4.  Reset Docker to factory settings (to remove all images)
5.  Remove c:\\ProgramData\\NavContainerHelper
6.  Install BcContainerHelper (Install-Module BcContainerHelper -force)

Hope this helps

**_Freddy Kristiansen_**  
Technical Evangelist
