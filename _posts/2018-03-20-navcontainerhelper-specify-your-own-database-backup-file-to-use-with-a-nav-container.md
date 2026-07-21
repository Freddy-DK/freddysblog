---
layout: post
title: "NavContainerHelper – Specify your own Database backup file to use with a NAV container"
date: 2018-03-20 03:44:00
categories: ["Docker", "NavContainerHelper"]
tags: ["backup", "bakfile", "Database", "Docker", "NAV on Docker", "NavContainerHelper"]
permalink: /2018/03/20/navcontainerhelper-specify-your-own-database-backup-file-to-use-with-a-nav-container/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

If you haven’t read the [first post](https://freddysblog.com/2018/03/20/navcontainerhelper-1/) about the NavContainerHelper, you should do so.

If you have a database backup file (.bak), you can specify that as parameter to the container. You can specify the bakfile using a secure URL. Read [this](https://freddysblog.com/2017/02/26/create-a-secure-url-to-a-file/) for information about how to create a secure url for a file.

```
$imageName = "microsoft/dynamics-nav:2018-rtm"
$navcredential = New-Object System.Management.Automation.PSCredential -argumentList "admin", (ConvertTo-SecureString -String "P@ssword1" -AsPlainText -Force)
New-NavContainer -accept_eula `
                 -containerName "test" `
                 -Auth NavUserPassword `
                 -imageName $imageName `
                 -Credential $navcredential `
                 -licenseFile "https://www.dropbox.com/s/abcdefghijkl/my.flf?dl=1" `
                 -additionalParameters @('--env bakfile="https://www.dropbox.com/s/abcdefghijkl/Demo%20Database%20NAV%20%2811-0%29.bak?dl=1"')
```

A second option is to place the .bak file in a folder, which you share to the container and then specify the container file path to the bakfile parameter in additionalParameters, like this:

```
$imageName = "microsoft/dynamics-nav:2018-rtm"
$navcredential = New-Object System.Management.Automation.PSCredential -argumentList "admin", (ConvertTo-SecureString -String "P@ssword1" -AsPlainText -Force)
New-NavContainer -accept_eula `
                 -containerName "test" `
                 -Auth NavUserPassword `
                 -imageName $imageName `
                 -Credential $navcredential `
                 -licenseFile "https://www.dropbox.com/s/abcdefghijkl/my.flf?dl=1" `
                 -additionalParameters @('--volume c:\temp\navdbfiles:c:\temp', '--env bakfile="c:\temp\Demo Database NAV (11-0).bak"')
```

A third optiopn is to specify the .bak file to the myscripts parameter and specify the path to the bakfile to be c:\\run\\my. All files specified in the myscripts parameter will be copied to the c:\\run\\my folder upon start of the container.

```
$imageName = "microsoft/dynamics-nav:2018-rtm"
$navcredential = New-Object System.Management.Automation.PSCredential -argumentList "admin", (ConvertTo-SecureString -String "P@ssword1" -AsPlainText -Force)
New-NavContainer -accept_eula `
                 -containerName "test" `
                 -Auth NavUserPassword `
                 -imageName $imageName `
                 -Credential $navcredential `
                 -licenseFile "https://www.dropbox.com/s/abcdefghijkl/my.flf?dl=1" `
                 -myScripts @("c:\temp\navdbfiles\Demo Database NAV (11-0).bak") `
                 -additionalParameters @('--env bakfile="c:\run\my\Demo Database NAV (11-0).bak"')
```

**Note**, when specifying a .bak file, the normal container initialization is still continuing and the NAV Super user will be created. This is not happening if you manually restore the .bak file to a SQL server and point out an external SQL Server database.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
