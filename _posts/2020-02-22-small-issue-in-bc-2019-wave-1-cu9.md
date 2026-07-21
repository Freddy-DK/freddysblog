---
layout: post
title: "Small issue in BC 2019 Wave 1 CU9"
date: 2020-02-22 10:37:18
categories: ["Docker", "NavContainerHelper", "PowerShell"]
tags: ["Compile-NavApplicationObject", "Export-NavApplicationObject", "Import-NavApplicationObject"]
permalink: /2020/02/22/small-issue-in-bc-2019-wave-1-cu9/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

Business Central 2019 wave 1 CU9 was shipped in February and it contains a small bug, which can cause some time consuming troubleshooting. Therefore this blog post.

# Issue #859

[https://github.com/microsoft/navcontainerhelper/issues/859](https://github.com/microsoft/navcontainerhelper/issues/859) was the first place I saw this bug and normally I would just fix these bugs and assume that people would use the latest Docker image or the latest NavContainerHelper and automagically get the fix.

I have of course done so, however…

This bug actually isn’t restricted to Docker and the essence of the bug is, that standard NAV CmdLets like **Import-NavApplicationObject**, **Export-NavApplicationObject** and **Compile-NavApplicationObject** doesn’t work in 14.10. If you are running in Docker, the commands will silently hang (actually displaying a dialog, which isn’t possible). If you are running in pipelines and UI-less environments, you might see the same.

NavContainerHelper 0.6.4.28 will patch the bug when creating the container and you should only see a small change at the end of container creation.

![patch](/assets/images/2020/small-issue-in-bc-2019-wave-1-cu9/patch.png)

If you are not using Docker, you can still see in the issue above how to work around the problem. Easiest fix for non-Docker users would be to edit the **Microsoft.Dynamics.Nav.Ide.psm1** file and add a default value for the **SuppressElevationCheck** parameter.

Replace

    **\[string\] $SuppressElevationCheck)**

With

    **\[string\] $SuppressElevationCheck = ‘No’)**

We will get the bug fixed for 14.11.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
