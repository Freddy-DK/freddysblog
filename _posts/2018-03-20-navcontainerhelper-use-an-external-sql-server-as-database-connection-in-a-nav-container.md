---
layout: post
title: "NavContainerHelper – Use an external SQL Server as database connection in a NAV container"
date: 2018-03-20 04:01:15
categories: ["Docker", "NavContainerHelper"]
tags: ["Database", "Docker", "External", "NAV on Docker", "NavContainerHelper", "SQL container"]
permalink: /2018/03/20/navcontainerhelper-use-an-external-sql-server-as-database-connection-in-a-nav-container/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

If you haven’t read the [first post](https://freddysblog.com/2018/03/20/navcontainerhelper-1/) about the NavContainerHelper, you should do so.

If you have a created a SQL Server container using one of the methods described any of the blog posts:

-   [Create a SQL Server container with the CRONUS database from a NAV container image](https://freddysblog.com/2018/03/20/navcontainerhelper-create-a-sql-server-container-with-the-cronus-database-from-a-nav-container-image/)
-   [Create a SQL Server container and restore a .bak file](https://freddysblog.com/2018/03/20/navcontainerhelper-create-a-sql-server-container-and-restore-a-bak-file/)

Then you will have the variables $databaseServer, $databaseInstance, $databaseName and $databaseCredential pointing to a database you can use to start up a NAV container. These parameters can be given directly to New-NavContainer.

If you have created your external database through other means, please set these variables in PowerShell. Please try the following script to see whether a docker container can connect to your database:

```
$dbPassword = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto([System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($databaseCredential.Password))
$databaseServerInstance = @{ $true = "$databaseServer$databaseInstance"; $false = "$databaseServer"}["$databaseInstance" -ne ""]
docker run -it --name sqlconnectiontest microsoft/mssql-server-windows-developer powershell -command "Invoke-Sqlcmd -ServerInstance '$databaseServerInstance' -Username '$($databaseCredential.Username)' -Password '$dbPassword' -Database '$databaseName' -Query 'SELECT COUNT(*) FROM [dbo].[User]'"
```

If the above script fails, you will not succeed starting a NAV container with these credentials, before your connection test succeeds.

Please remove your connection test container using:

```
docker rm sqlconnectiontest -f
```

When you successfully have conducted the connection test above, you can start a NAV container using this script:

```
$navcredential = New-Object System.Management.Automation.PSCredential -argumentList "admin", (ConvertTo-SecureString -String "P@ssword1" -AsPlainText -Force)
New-NavContainer -accept_eula `
                 -containerName "test" `
                 -Auth NavUserPassword `
                 -imageName $imageName `
                 -Credential $navcredential `
                 -databaseServer $databaseServer `
                 -databaseInstance $databaseInstance `
                 -databaseName $databaseName `
                 -databaseCredential $databaseCredential
```

If your database doesn’t have a license file, you can upload a license file using:

```
Import-NavContainerLicense -containerName test -licenseFile "https://www.dropbox.com/s/abcdefghijkl/my.flf?dl=1"
```

If you do not import a license file, you are likely to get errors like this when trying to access the NAV container.

```
The following SQL error was unexpected:
Invalid object name 'master.dbo.$ndo$srvproperty'.
```

You can add users to the database using:

`New-NavContainerNavUser -containerName test -Credential $navcredential`

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
