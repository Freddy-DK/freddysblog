---
layout: post
title: "Mounting a database from my online environment using SQL Server on the host"
date: 2019-11-12 11:40:41
categories: ["Demo Environments", "Docker", "NavContainerHelper", "PowerShell"]
tags: ["Business Central Sandbox", "Database", "Docker", "Multitenancy", "NavContainerHelper"]
permalink: /2019/11/12/mounting-a-database-from-my-online-environment-using-sql-server-on-the-host/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

This blog post is really a combination between the last two blog posts, [https://freddysblog.com/2019/11/04/using-sql-server-on-the-host/](/2019/11/04/using-sql-server-on-the-host/) and [https://freddysblog.com/2019/11/12/mounting-a-database-backup-from-my-online-environment-inside-a-container/](/2019/11/12/mounting-a-database-backup-from-my-online-environment-inside-a-container/). As stated in the last blog post, you can only use databases of less than 10Gb in size inside the container due to SQL Express. This blog post will explain how to get past that problem.

# **Please Note:** Not for production

_**Please note** that following this blog post will not give you an environment you can use for production. First of all, there might be extensions installed in your online environment, which you cannot install locally and furthermore Docker is not supported for production even if Docker would have been supported for production, we would definitely not support sandbox containers in production (as they simulate online and there are a number of things that doesn’t work in sandbox containers)._

# Prepare yourself

Please follow the steps outlined in [this blog post](/2019/11/12/mounting-a-database-backup-from-my-online-environment-inside-a-container/) to download your database backup in .bacpac format. You should also find out what image to use and again, I recommend using the latest sandbox image for the same reasons as described.

# Create a multitenant container using SQL Server on the host

Like described in this blog post, we need to extract the database from the container and restore that on the SQL Server on the host.

Note that the database name is the **containername****\-app**. This serves the purpose that I can remove the container and all databases starting with **containername-** in order to start from scratch. Without this, I would only be able to have one multitenant container using the same SQL Server.

```
$containerName = "test"
$DatabaseFolder = "c:\databases"
$DatabaseName = "$containerName-app"

if (!(Test-Path $DatabaseFolder)) {
    New-Item $DatabaseFolder -ItemType Directory | Out-Null
}

if (Test-Path (Join-Path $DatabaseFolder "$($DatabaseName).*")) {
    Remove-BCContainer $containerName
    $databases = Invoke-Sqlcmd "SELECT name FROM master.sys.databases" | Where-Object { $_.name.StartsWith("$ContainerName-") }
    $databases | ForEach-Object {
        $dbname = $_.name
        Write-Host "Dropping database $dbname"
        Invoke-SqlCmd -Query "ALTER DATABASE [$dbname] SET OFFLINE WITH ROLLBACK IMMEDIATE" 
        Invoke-Sqlcmd -Query "DROP DATABASE [$dbname]"
        Write-Host "Removing Database files $($databaseFolder)\$($dbname).*"
        Remove-Item -Path (Join-Path $DatabaseFolder "$($dbname).*") -Force
    }
}

$imageName = Get-BestBCContainerImageName -imageName "mcr.microsoft.com/businesscentral/sandbox:us"
docker pull $imageName

$dbPath = Join-Path $env:TEMP ([Guid]::NewGuid().ToString())
Extract-FilesFromBCContainerImage -imageName $imageName -extract database -path $dbPath -force

$files = @()
Get-ChildItem -Path (Join-Path $dbPath "databases") | ForEach-Object {
    $DestinationFile = "{0}\{1}{2}" -f $databaseFolder, $DatabaseName, $_.Extension
    Copy-Item -Path $_.FullName -Destination $DestinationFile -Force
    $files += @("(FILENAME = N'$DestinationFile')")
}

Remove-Item -Path $dbpath -Recurse -Force

Write-Host "Attaching files as new Database $DatabaseName"
Write-Host "CREATE DATABASE [$DatabaseName] ON $([string]::Join(", ",$Files)) FOR ATTACH"
Invoke-SqlCmd -Query "CREATE DATABASE [$DatabaseName] ON $([string]::Join(", ",$Files)) FOR ATTACH"
```

After restoring the database to the host, we can create the container and much like in the previous blog post, we will create a container and specify database connection.

This time we will also specify **\-multitenant**, but as you might know, this won’t change anything. When you specify a database connection, then I will assume that the database is ready – and not try to switch it into multitenancy mode. So we need a little more.

We need to override a few scripts. The **SetupDatabase** script (in order to prepare app and tenant databases) and the **SetupTenant** script (in order to mount the default tenant).

The last file we need to override is **HelperFunctions.ps1**. Here we will just use a newer version of HelperFunctions.ps1 directly from the nav-docker github repository, which fixes an issue with Copy-NavDatabase when running on a SQL Server on the host.

With the next version of the generic image, we don’t need to override HelperFunctions, but then again, it doesn’t hurt.

```
$setupDatabaseScript = @'
. 'C:\Run\SetupDatabase.ps1'
if (!$restartingInstance) {
    Copy-NavDatabase -databaseserver $DatabaseServer -databaseInstance $databaseInstance -databaseCredentials $databaseCredentials -sourcedatabaseName $databaseName -destinationDatabaseName "$(hostname)-$tenantId"
    Copy-NavDatabase -databaseserver $DatabaseServer -databaseInstance $databaseInstance -databaseCredentials $databaseCredentials -sourcedatabaseName $databaseName -destinationDatabaseName "$(hostname)-tenant"
}
'@

$setupTenantScript = @'
. 'c:\Run\SetupTenant.ps1'
if (!$restartingInstance) {
    Mount-NavDatabase -databaseserver $DatabaseServer -databaseInstance $databaseInstance -databaseCredentials $databaseCredentials -ServerInstance $ServerInstance -TenantId $TenantId -DatabaseName "$(hostname)-$tenantId"
}
'@

$credential = New-Object pscredential 'admin', (ConvertTo-SecureString -String 'P@ssword1' -AsPlainText -Force)
$dbcredentials = New-Object PSCredential -ArgumentList 'sa', $credential.Password
New-BCContainer `
    -accept_eula `
    -containerName $containerName `
    -imageName $imageName `
    -updateHosts `
    -auth UserPassword `
    -Credential $credential `
    -licensefile C:\temp\license.flf `
    -databaseServer 'host.containerhelper.internal' `
    -databaseInstance '' `
    -databaseName $DatabaseName `
    -databaseCredential $dbcredentials `
    -multitenant `
    -myScripts @(
        "https://raw.githubusercontent.com/microsoft/nav-docker/master/generic/Run/HelperFunctions.ps1"
        @{ "SetupDatabase.ps1" = $setupDatabaseScript }
        @{ "SetupTenant.ps1" = $setupTenantScript }
    )
New-NavContainerNavUser `
    -containerName $containerName `
    -Credential $credential `
    -ChangePasswordAtNextLogOn:$false `
    -PermissionSetId SUPER `
    -tenant 'Default'
```

**With this, we will have a multitenant container using SQL Server on the host.**

Not quite as easy as when running with the databases inside the container, but absolutely doable.

_**Note:** The clever reader will notice that I do not Remove the application part from the tenant database and I do not remove the tenant part of the database from the app database. This is because the cmdlets for doing this only works with Windows Authentication and won’t work towards SQL Server on the host. For the purpose here, it doesn’t really matter – the container works just fine as is. If you need to clean up the app and tenant databases, you will have to extract the PowerShell cmdlets from the container and run them locally._

# Restore the .bacpac

Now, we “just” need to restore and mount the .bacpac from our online environment. Restoring needs to be done on the host – and NOT in the container. You can do this using SSMS or you can use PowerShell. When using PowerShell, you need to install the Dac Framework on the host and you can find it here: [https://go.microsoft.com/fwlink/?linkid=2108813](https://go.microsoft.com/fwlink/?linkid=2108813).

we also need to make sure that the imported database is placed in the same folder as the other databases (c:\\databases) and I can do this by reconfiguring the server through the smo dll.

The following script will restore the .bacpac and place the files in $databaseFolder:

```
$dacdll = Get-Item "C:\Program Files\Microsoft SQL Server\*\DAC\bin\Microsoft.SqlServer.Dac.dll"
if (!($dacdll)) {
    throw "Dac Framework is not installed, download and install from here: https://go.microsoft.com/fwlink/?linkid=2108813"
}
$smodll = Get-Item -Path "C:\Program Files\Microsoft SQL Server\*\SDK\Assemblies\Microsoft.SqlServer.Smo.dll"
if (!($smodll)) {
    throw "Unable to locate Microsoft.SqlServer.smo.dll. Is SQL Server installed on this machine?"
}

Add-Type -path $dacdll.FullName
Add-Type -Path $smodll.FullName

$tenantBacpac = "C:\ProgramData\NavContainerHelper\Production_20191110_02.bacpac"
$tenantId = "mydata"
$databaseName = "$containerName-$tenantId"
$server = New-Object Microsoft.SqlServer.Management.Smo.Server($env:ComputerName)
$defaultFile = $server.Properties["DefaultFile"].Value
$defaultLog = $server.Properties["DefaultLog"].Value
$server.Properties["DefaultFile"].Value = $DatabaseFolder
$server.Properties["DefaultLog"].Value = $DatabaseFolder
$server.Alter()
try {
    $conn = "Data Source=localhost;Initial Catalog=master;Connection Timeout=0;Integrated Security=True;"
    Write-Host "Restoring Database from $tenantBacpac as $databaseName"
    $AppimportBac = New-Object Microsoft.SqlServer.Dac.DacServices $conn
    $ApploadBac = [Microsoft.SqlServer.Dac.BacPackage]::Load($tenantBacpac)
    $AppimportBac.ImportBacpac($ApploadBac, $DatabaseName)
}
finally {
    $server.Properties["DefaultFile"].Value = $defaultFile
    $server.Properties["DefaultLog"].Value = $defaultLog
    $server.Alter()
}
```

# Publish/Remove extensions

Like with the other blog post, we need to install extensions in the container (if we have the source) or remove them from the database:

```
Publish-BCContainerApp -containerName $containerName -appFile "C:\Users\freddyk\Downloads\Freddy Kristiansen_BingMaps_15.0.56.0.app" -skipVerification -sync -install
$removeApps = @("MyApp")
$removeApps | ForEach-Object {
Invoke-Sqlcmd -Query "USE [$databaseName]
GO
DELETE FROM [dbo].[NAV App Published App]
WHERE Name = '$_'
DELETE FROM [dbo].[NAV App Installed App]
WHERE Name = '$_'
GO"
}
```

# Mount and sync the database

Now, the database should be ready for mounting and sync’ing and the method to do this is similar to doing it in the container (just specifying database server, instance and credentials)

```
Invoke-ScriptInBCContainer -containerName $containerName -scriptblock { Param($tenantId, $databaseServer, $databaseInstance, $DatabaseName, $databaseCredential)
    Mount-NavTenant `
        -ServerInstance $ServerInstance `
        -id $tenantId `
        -databaseserver $databaseServer `
        -databaseinstance $databaseInstance `
        -databasename $databaseName `
    ​    -databaseCredential $databaseCredential `
        -EnvironmentType Sandbox `
        -OverwriteTenantIdInDatabase `
        -Force
    Sync-NavTenant `
        -ServerInstance $ServerInstance `
        -Tenant $tenantId `
        -Force
} -argumentList $tenantId, 'host.containerhelper.internal', '', $DatabaseName, $dbcredentials
```

# Create the super user

Last but not least, create the super user with which you can login to the tenant:

```
New-NavContainerNavUser `
    -containerName $containerName `
    -credential $credential `
    -changePasswordAtNextLogOn:$false `
    -permissionSetId SUPER `
    -tenant $tenantId
```

and there you are:

![Screenshot 2019-11-12 11.36.55](/assets/images/2019/mounting-a-database-from-my-online-environment-using-sql-server-on-the-host/screenshot-2019-11-12-11.36.55.png)

again – a LOT of PowerShell and you can probably stitch things together to work for you…

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
