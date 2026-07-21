---
layout: post
title: "NavContainerHelper – Create a SQL Server container with the CRONUS database from a NAV container image"
date: 2018-03-20 03:49:50
categories: ["Docker", "NavContainerHelper"]
tags: ["CRONUS", "Database", "Docker", "NAV on Docker", "NavContainerHelper", "SQL container"]
permalink: /2018/03/20/navcontainerhelper-create-a-sql-server-container-with-the-cronus-database-from-a-nav-container-image/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

If you haven’t read the [first post](/2018/03/20/navcontainerhelper-1/) about the NavContainerHelper, you should do so.

The NAV container images contains SQL Express with the CRONUS Demo Database. If we want to get a copy of the databases from a NAV container image, we can override the navstart.ps1 script with a script, which basically just starts the SQL Server, takes the database offline and copies the database files to a folder.

Example:

```
$navstartScript = @'
Write-Host "Extracting databases..."
$startTime = [DateTime]::Now

. "c:\run\HelperFunctions.ps1"
. "c:\run\SetupVariables.ps1"

# start the SQL Server
Write-Host "Starting Local SQL Server"
Start-Service -Name $SqlServiceName -ErrorAction Ignore -WarningAction Ignore

Write-Host "Take database [$DatabaseName] offline"
Invoke-SqlCmd -Query "ALTER DATABASE [$DatabaseName] SET OFFLINE WITH ROLLBACK IMMEDIATE"

$databaseFolder = "c:\navdbfiles"
Write-Host "Using $databaseFolder as new location for database files"

$mdfName = Join-Path $databaseFolder "$DatabaseName.mdf"
$ldfName = Join-Path $databaseFolder "$DatabaseName.ldf"

(Invoke-SqlCmd -Query "SELECT Physical_Name as filename FROM sys.master_files WHERE DB_NAME(database_id) = '$DatabaseName'").filename | ForEach-Object {
    $FileInfo = Get-Item -Path $_
    $DestinationFile = "{0}{1}{2}" -f $databaseFolder, $DatabaseName, $FileInfo.Extension
    if (($DestinationFile -ne $mdfName) -and ($destinationFile -ne $ldfName)) { throw "Unexpected filename: $DestinationFile" }
    Write-Host $DestinationFile
    Copy-Item -Path $FileInfo.FullName -Destination $DestinationFile -Force
}
$timespend = [Math]::Round([DateTime]::Now.Subtract($startTime).Totalseconds)
Write-Host "Extracting databases took $timespend seconds"
Write-Host "Ready for connections!"
'@

$hostFolder = "c:\temp\navdbfiles"
$imageName = "microsoft/dynamics-nav"

$additionalParameters = @("-v ${hostFolder}:c:\navdbfiles")
$tempcredential = New-Object System.Management.Automation.PSCredential -argumentList "admin", (ConvertTo-SecureString -String "P@ssword1" -AsPlainText -Force)
New-NavContainer -accept_eula `
                 -containerName "temp" `
                 -imageName $imageName `
                 -Credential $tempcredential `
                 -memoryLimit "1g" `
                 -shortcuts None `
                 -additionalParameters $additionalParameters `
                 -myScripts @{"navstart.ps1" = $navstartScript}
Remove-NavContainer -containerName "temp"
```

The navstart script above will work with all standard NAV container images to extract the database files to a folder, which is shared as c:\\navdbfiles. Below is the script, which will start the container, get the database files and remove the container again. On my Windows 2016 laptop, this takes ~20 seconds.

In this example, it gets the .mdf and .ldf files from the latest cumulative update of the latest released version of NAV with W1 localization. Adding the lines below will spin up a SQL Server Developer container and attach the database files extracted by the lines above.

```
$databaseCredential = New-Object System.Management.Automation.PSCredential -argumentList "sa", (ConvertTo-SecureString -String "P@ssword1" -AsPlainText -Force)
$databaseName = "CRONUS"
$attach_dbs = (ConvertTo-Json -Compress -Depth 99 @(@{"dbName" = "$databaseName"; "dbFiles" = @("c:\temp\${databaseName}.mdf", "c:\temp\${databaseName}.ldf") })).replace('"',"'")
$dbPassword = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto([System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($databaseCredential.Password))
$dbserverid = docker run -d -e sa_password="$dbPassword" -e ACCEPT_EULA=Y -v "${hostFolder}:C:/temp" -e attach_dbs="$attach_dbs" microsoft/mssql-server-windows-developer
$databaseServer = $dbserverid.SubString(0,12)
$databaseInstance = ""
```

After this you have a SQL Server Container with your NAV database. Variables $databaseServer, $databaseInstance, $databaseName and $databaseCredential are the parameters used to start up a NAV Container to use this new database server.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
