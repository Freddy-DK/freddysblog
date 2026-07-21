---
layout: post
title: "NavContainerHelper – Start a NAV container and place the database files on a file share on the host computer"
date: 2018-03-20 03:46:11
categories: ["Docker", "NavContainerHelper"]
tags: ["Database", "Docker", "NAV on Docker", "NavContainerHelper", "Share"]
permalink: /2018/03/20/navcontainerhelper-start-a-nav-container-and-place-the-database-files-on-a-file-share-on-the-host-computer/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

If you haven’t read the [first post](https://freddysblog.com/2018/03/20/navcontainerhelper-1/) about the NavContainerHelper, you should do so.

The database files are placed inside the container by default. If you want to copy the database to a share on the Docker host, you can override the SetupDatabase.ps1 script by creating a file called SetupDatabase.ps1, specify that in myScripts.ps1 to New-NavContainer and share a folder on the host, which can host the DB files.

Example:

```
$SetupDatabaseScript = @'
if ($restartingInstance) {
    # Do nothing on restart
} else {
    $databaseFolder = "c:\navdbfiles"
    Write-Host "Using $databaseFolder as new location for database files"

    $mdfName = Join-Path $databaseFolder "$DatabaseName.mdf"
    $ldfName = Join-Path $databaseFolder "$DatabaseName.ldf"
    $filesExists = (Test-Path $mdfName -PathType Leaf) -and (Test-Path $ldfName -PathType Leaf)

    Write-Host "Take database [$DatabaseName] offline"
    Invoke-SqlCmd -Query "ALTER DATABASE [$DatabaseName] SET OFFLINE WITH ROLLBACK IMMEDIATE"

    if ($filesExists) {
        Write-Host "Database files for [$DatabaseName] already exists"
    } else {
        Write-Host "Move database files for [$DatabaseName]"
        (Invoke-SqlCmd -Query "SELECT Physical_Name as filename FROM sys.master_files WHERE DB_NAME(database_id) = '$DatabaseName'").filename | ForEach-Object {
            $FileInfo = Get-Item -Path $_
            $DestinationFile = "{0}{1}{2}" -f $databaseFolder, $DatabaseName, $FileInfo.Extension
            if (($DestinationFile -ne $mdfName) -and ($destinationFile -ne $ldfName)) { throw "Unexpected filename: $DestinationFile" }
            Copy-Item -Path $FileInfo.FullName -Destination $DestinationFile -Force
        }
    }

    Write-Host "Drop database [$DatabaseName]"
    Invoke-SqlCmd -Query "DROP DATABASE [$DatabaseName]"

    $Files = "(FILENAME = N'$mdfName'), (FILENAME = N'$ldfName')"
    Write-Host "Attach files as new Database [$DatabaseName]"
    Invoke-SqlCmd -Query "CREATE DATABASE [$DatabaseName] ON (FILENAME = N'$mdfName'), (FILENAME = N'$ldfName') FOR ATTACH"
}
'@

$additionalParameters = @("-v c:\temp\navdbfiles:c:\navdbfiles")
New-NavContainer -accept_eula `
                 -containerName "test" `
                 -auth NavUserPassword `
                 -imageName "microsoft/dynamics-nav" `
                 -additionalParameters $additionalParameters `
                 -myScripts @{"SetupDatabase.ps1" = $SetupDatabaseScript }
```

The SetupDatabase.ps1 script above will check whether the database files already exists and attach/reuse them if this is the case. If the databases files doesn’t exist, they will be copied to the specified folder and attached.

The additionalParameter sets up a shared folder. The local folder c:\\temp\\navdbfiles as c:\\navdbfiles in the container and shares the SetupDatabase.ps1 script from above.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
