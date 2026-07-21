---
layout: post
title: "Windows Server 2019 and SQL Server 2017"
date: 2018-11-15 11:27:08
categories: ["Docker"]
tags: ["1709", "1803", "1809", "SQL Server 2016", "SQL Server 2017", "Windows Server 2016", "Windows Server 2019"]
permalink: /2018/11/15/windows-server-2019-and-sql-server-2017/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

Windows Server 2019 (1809) finally shipped (again) and with that we have a new LTSC (Long Term Servicing Channel). This means that we will begin creating images for Windows Server 2019 as soon as our infrastructure is ready for that.

# \-UseBestContainerOS

While waiting for 1809 images, you can use -UseBestContainerOS when running images with NavContainerHelper and it will automatically detect the best generic image to use for your host OS, whether that is Windows Server 2016, Windows Server 2019, Windows 10 1709, Windows 10 1803 or Windows 10 1809.

It will even detect that you are running an insider version of Windows Server and fall back to Windows Server 2019 generic image.

# SQL Server 2017

With the release of Windows Server 2019, I tried to just rebuild the generic image based on the new Windows Server Core 1809 image. SQL Server 2016 has always been used in the images, but it did cause problems with 1809, so I decided to switch to SQL Server 2017, which installs and works.

A few issues though encountered during testing:

## DAC missing

SQL Server 2016 came with a version of Dac Framework pre-installed. SQL Server 2017 doesn’t. Dac was used for importing .bacpac files in the container. I have changed this code to do a Just-In-Time download and install of Dac Framework 18.0 when needed.

## Export-NavApplication

When switching to multitenant setup, the container uses Export-NavApplication. This worked out of the box for SQL Server 2016, but for some reason SQL Server 2017 failed. After a lot of investigations, I found that setting -ServiceAccount on Export-NavApplication to **NT AUTHORITYSYSTEM** and adding the user to the database solved the issue. This also required the Export-NavContainerDatabasesAsBacpac in NavContainerHelper to remove this user on export.

## Cmdlets expecting “old” version of DLLs

The Cmdlets for Service Tier administration in old NAV and Business Central versions are expecting earlier versions of SQL Server DLL’s:

-   Microsoft.SqlServer.Smo
-   Microsoft.SqlServer.SmoExtended
-   Microsoft.SqlServer.ConnectionInfo
-   Microsoft.SqlServer.SqlEnum

By placing dependentAssembly sections the AssemblyBinding section in PowerShell.exe.config in the container, specifying that any attempt to load old versions of these DLLs, should take the new version:

## This should make all CmdLets work with the new SQL Server.

## Export-NavContainerDatabasesAsBacpac

This function is placed in the NavContainerHelper, but it also used to install a newer version of the Dac Framework (than the one included in SQL Server 2016). This function has been changed to also use the new version (18.0) and it will just re-use the version installed in the container if possible. This fix is included in NavContainerHelper 0.4.1.0.

# So, what’s there now

You will need **NavContainerHelper 0.4.1.0** for supporting Windows Server 2019 and SQL Server 2017.

While writing this blog post, the generic images have been updated:

-   **microsoft/dynamics-nav:generic-ltsc2016** is the generic image for Windows 10 1607, Windows ServerCore 1607 or Windows Server 2016. Based on Windows ServerCore ltsc2016 and the foundation of all **ltsc2016** NAV and Business Central containers
-   **microsoft/dynamics-nav:generic-1709** is the generic image for Windows 10 1709 release or Windows ServerCore 1709. Based on Windows ServerCore 1709.
-   **microsoft/dynamics-nav:generic-1803** is the generic image for Windows 10 1803 release or Windows ServerCore 1803. Based on Windows ServerCore 1803.
-   **microsoft/dynamics-nav:generic-ltsc2019** is the generic image for Windows 10 1809, Windows ServerCore 1809 or Windows Server 2019. Based on Windows ServerCore 1809.

**microsoft/dynamics-nav:generic** still points to **microsoft/dynamics-nav:generic-ltsc2016**.

The version number of the new Generic image is **0.0.8.0**.

# What will be there tomorrow

All future images (NAV and Business Central) will be build on the new ltsc2016 image with SQL Server 2017.

# What will be there in a week from now

All NAV and Business Central images have been rebuilt to use the new ltsc2016 image with SQL Server 2017.

# And finally, what will be there in a few weeks

All images are available as ltsc2019 as well as ltsc2016. In order to download the 1809 version you will have to append -ltsc2019 if using docker run. New-NavContainer will automatically detect that your host is Server 2019 and download the right image for that.

Remember that you can use ltsc2019 already today by specifying **\-useBestContainerOS** to **New-NavContainer** or you can extract the files from a ltsc2016 container using **Extract-FilesFromNavContainerImage** and use them to run ltsc2019 image.

All dates are subject to change…

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
