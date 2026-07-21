---
layout: post
title: "NavContainerHelper – Create a SQL Server container and restore a .bak file"
date: 2018-03-20 03:57:03
categories: ["Docker", "NavContainerHelper"]
tags: ["backup", "bakfile", "Database", "Docker", "NAV on Docker", "NavContainerHelper", "SQL container"]
permalink: /2018/03/20/navcontainerhelper-create-a-sql-server-container-and-restore-a-bak-file/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

If you haven’t read the [first post](https://freddysblog.com/2018/03/20/navcontainerhelper-1/) about the NavContainerHelper, you should do so.

The following script sample, will create a new SQL Server container and restore a NAV 2018 database backup file (Demo Database NAV (11-0).bak) placed on the host in a folder called c:\\temp\\navdbfiles. The folder c:\\temp\\navdbfiles on the host is shared as c:\\temp inside the container.

```
$hostFolder = "c:\temp\navdbfiles"
$databaseCredential = New-Object System.Management.Automation.PSCredential -argumentList "sa", (ConvertTo-SecureString -String "P@ssword1" -AsPlainText -Force)
$dbPassword = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto([System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($databaseCredential.Password))
$dbserverid = docker run -d -e sa_password="$dbPassword" -e ACCEPT_EULA=Y -v "${hostFolder}:C:/temp"
$databaseServer = $dbserverid.SubString(0,12)
$databaseInstance = ""

$databaseName = "Demo Database NAV (11-0)"
$databaseServerInstance = @{ $true = "$databaseServer$databaseInstance"; $false = "$databaseServer"}["$databaseInstance" -ne ""]
$RelocateData = New-Object Microsoft.SqlServer.Management.Smo.RelocateFile("${databaseName}_Data", "c:\temp\${databaseName}_Data.mdf")
$RelocateLog = New-Object Microsoft.SqlServer.Management.Smo.RelocateFile("${databaseName}_Log", "c:\temp\${databaseName}_Log.ldf")
Restore-SqlDatabase -ServerInstance $databaseServerInstance -Database $databaseName -BackupFile "C:\temp\$databaseName.bak" -Credential $databaseCredential -RelocateFile @($RelocateData,$RelocateLog)
```

After this, the SQL Server is ready to use from a NAV container and the variables $databaseServer, $databaseInstance, $databaseName and $databaseCredential are the parameters used to start up a NAV container to use this new database server.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
