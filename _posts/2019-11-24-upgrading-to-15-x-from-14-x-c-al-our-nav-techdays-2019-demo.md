---
layout: post
title: "Upgrading to 15.x from 14.x C/AL – our NAV TechDays 2019 demo"
date: 2019-11-24 15:26:05
categories: ["AL Development", "Docker", "NavContainerHelper", "PowerShell"]
tags: ["C/AL to AL", "Docker", "NavContainerHelper", "Upgrade"]
permalink: /2019/11/24/upgrading-to-15-x-from-14-x-c-al-our-nav-techdays-2019-demo/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

_**WARNING: VERY LONG BLOG POST AHEAD!**_

My session at [NAV Tech Days](https://navtechdays.com/) this year was together with [Nikola Kukrika](https://twitter.com/NikolaKukrika) and we did an end 2 end walk-through of how to upgrade a code customized C/AL solution in 14.x to 15.x (AL), converting the code, upgrading the data, explaining all the pitfalls and in some cases, how to cope with missing functionality.

I will add a link to the youtube video of our session when the video is out, but for now – this blog post serves the purpose of allowing you to do the exact same demo as I performed on stage.

# 3 different routes…

During the presentation, we discussed 3 different routes from 14.x to 15.x.

## Code customized C/AL to AL extension

Preparing your solution on C/AL to be converted directly into an AL extension seems like the right approach and brings you directly to the place you want to be. It was also discussed in [this blog post](https://freddysblog.com/2019/04/15/c-al-to-al-extension/) from April 2019. This requires more work up front. Making sure that you have moved all code customizations to event handlers, no breaking SQL changes, no removed fields or controls and all in all you need to have a very clean solution to do this.

At this time however, there is no good way of upgrading if you have added fields to existing tables. You would have to create new fields in your extension, move data manually and obsolete the old fields. A cumbersome process, which we will have functionality to avoid within the next few months (no date given, but I will update this blog post when possible).

In order to move a solution to 15.x and do data upgrade during the presentation, we decided not to do this.

## Code customized C/AL to code customized AL

This requires less up front preparations, but the number of preparations you have to do in AL is bigger and some customizations are even harder to do in AL. This was discussed in [this blog post](https://freddysblog.com/2019/04/15/c-al-to-al-code-customizations/) from April 2019. Doing this wouldn’t really get you anywhere. If the goal is to move your customizations to an extension, this will even get you into trouble with added tables.

At this time, there is no good way of moving a table from one app to another (and this is what you would have to do if moving from code customized AL to an AL extension later)

In order to move a solution to 15.x during the presentation and be prepared to move to an extension, we decided not to do this.

## Code customized C/AL to AL Hybrid

This was the model we went for during the presentation. This requires the same preparations as when moving to an AL Extension, but in order to upgrade, we leave the table extensions (the new fields) in the base app (the code customized AL part). In your own solution you could also be in the situation, where you would leave more things in the base app in order to make your move easier.

This is the model we were demoing at Tech Days and this is the following demo.

# Prerequisites

If you want to run this demo, you should have a computer running Windows 10 1809 or later (or Windows Server 2019). It should have 32Gb RAM. Latest Docker and latest [NavContainerHelper from the PowerShell Gallery](https://www.powershellgallery.com/packages/navcontainerhelper). SQL Server 2017 developer edition is installed on the computer, setup with dual authentication mode and listening on TCP as described [here](https://freddysblog.com/2019/11/04/using-sql-server-on-the-host/). I am running all script snippets in PowerShell ISE with elevated permissions (**Run as administrator**).

Furthermore, I am using [KDiff3](http://kdiff3.sourceforge.net/) for 3 way merging. There are a lot of other tools out there, that does the same thing.

# Settings

These settings and variables should reflect your setup if you want to run the demo script yourself.

$ErrorActionPreference = 'Stop'
$licenseFile = 'c:\\temp\\license.flf'

$dbcredentials = New-Object PSCredential -ArgumentList 'sa', (ConvertTo-SecureString -String 'P@ssword1' -AsPlainText -Force)
$dbServer = 'host.containerhelper.internal'
$dbInstance = ''
$dbFolder = "c:\\databases"
$dbName = "Prod"
$prodContainerName = "prod"
if (!(Test-Path $dbFolder)) { New-Item $dbFolder -ItemType Directory | Out-Null }

$rewardsAppCodeFile = 'c:\\ProgramData\\NavContainerHelper\\RewardsAppcode.txt'
Download-File -sourceUrl 'https://bcdocker.blob.core.windows.net/public/RewardsAppcode.txt' -destinationFile $rewardsAppCodeFile -dontOverwrite
Unblock-File $rewardsAppCodeFile

$bingMaps14appFile = 'c:\\ProgramData\\NavContainerHelper\\BingMaps14.app'
Download-File -sourceUrl 'http://aka.ms/bingmaps.app' -destinationFile $bingMaps14appFile -dontOverwrite
Unblock-File $bingMaps14appFile

$bingMaps15appFile = 'c:\\ProgramData\\NavContainerHelper\\BingMaps15.app'
Download-File -sourceUrl 'http://aka.ms/FreddyKristiansen\_BingMaps\_15.0.app' -destinationFile $bingMaps15appFile -dontOverwrite
Unblock-File $bingMaps15appFile

$appJsonTemplate = \[System.Net.WebClient\]::new().DownloadString('https://bcdocker.blob.core.windows.net/public/app15.json') | ConvertFrom-Json

$imageName14 = 'mcr.microsoft.com/businesscentral/onprem:1904-cu6-w1-ltsc2019'
$imageName15 = 'mcr.microsoft.com/businesscentral/onprem:1910-cu1-w1-ltsc2019'
$dev14containerName = 'bc14'
$dev15containerName = 'bc15'
$credential = New-Object pscredential -ArgumentList 'admin', (ConvertTo-SecureString -String 'P@ssword1' -AsPlainText -Force)

$bc14originalFolder = 'c:\\ProgramData\\NavContainerHelper\\TechDays\\bc14original'
$bc15originalFolder = 'c:\\ProgramData\\NavContainerHelper\\TechDays\\bc15original'
$bc14mybaseappFolder = 'c:\\ProgramData\\NavContainerHelper\\TechDays\\bc14mybaseapp'
$bc15mybaseappFolder = 'c:\\ProgramData\\NavContainerHelper\\TechDays\\bc15mybaseapp'
$bc15myappFolder = 'c:\\ProgramData\\NavContainerHelper\\TechDays\\bc15myapp'

**dbCredentials**, **dbServer**, **dbInstance** and **dbName** defines the database on your host. Setting **dbServer** to **host.containerhelper.internal** means that the container will assume that the database is on the host and **dbFolder** determines the location of the database files. The remaining scripts will not automatically work if you want to use a SQL server, which is on a foreign machine, as I copy mdf/ldf files and use database attach.

All the folders defined must be shared with the containers. Placing them under c:\\ProgramData\\NavContainerHelper will automatically ensure that they are shared.

The two images defined as 14 and 15 matches the objects in the RewardsAppCode. Changing the 14 image to another CU, you also need to change the code and you also need to change the 15 image.

# The Production Environment

This section sets up our “production” environment. Base on Business Central Spring 2019 CU6 (1904-cu6), this is the environment we want to lift to Business Central 15.x.

Using the techniques described in this blog post, we will place the database on the docker host (the machine on which this script is running) and then import our Rewards App objects and install the BingMaps app.

This means that we want to lift both code customizations in C/AL and an AL app to Business Central 15.x. During the Tech Days presentation, we did not include the BingMaps app.

$tempPath = Join-Path $env:TEMP (\[Guid\]::NewGuid().ToString())
Extract-FilesFromBCContainerImage -imageName $imageName14 -extract database -path $tempPath -force

$files = @()
Get-ChildItem -Path (Join-Path $tempPath "databases") | % {
    $DestinationFile = "{0}\\{1}{2}" -f $dbFolder, $dbName, $\_.Extension
    Copy-Item -Path $\_.FullName -Destination $DestinationFile -Force
    $files += @("(FILENAME = N'$DestinationFile')")
}

Remove-Item -Path $tempPath -Recurse -Force

Write-Host "Attaching files as new Database $dbName"
Invoke-SqlCmd -Query "CREATE DATABASE \[$dbName\] ON $(\[string\]::Join(", ",$Files)) FOR ATTACH"

New-BCContainer \`
    -accept\_eula \`
    -containerName $prodContainerName \`
    -imageName $imageName14 \`
    -updateHosts \`
    -auth 'UserPassword' \`
    -Credential $credential \`
    -databaseServer $dbServer \`
    -databaseInstance $dbInstance \`
    -databaseName $dbName \`
    -databaseCredential $dbcredentials

Import-NavContainerLicense \`
    -containerName $prodContainerName \`
    -licenseFile $licenseFile

New-NavContainerNavUser \`
    -containerName $prodContainerName \`
    -Credential $credential \`
    -ChangePasswordAtNextLogOn:$false \`
    -PermissionSetId 'SUPER'

Import-ObjectsToNavContainer \`
    -containerName $prodContainerName \`
    -objectsFile $rewardsAppCodeFile \`
    -sqlCredential $dbcredentials

Compile-ObjectsInNavContainer \`
    -containerName $prodContainerName \`
    -sqlCredential $dbcredentials

Publish-NavContainerApp \`
    -containerName $prodContainerName \`
    -appFile $bingMaps14appFile \`
    -skipVerification \`
    -sync \`
    -install

Start-Process "http://$prodContainerName/NAV"

The last line in the above script will open the Web Client and you can now setup the BingMaps integration (link on the role center) and enter some data into the rewards points field for at least one of the customers.

![14.x-2](/assets/images/2019/upgrading-to-15-x-from-14-x-c-al-our-nav-techdays-2019-demo/14.x-2.png)

With this we are ready to start the process…

# Create temporary work containers

We need two temporary work containers to achieve this task. One using the same version as our production environment and one using the version to which we want to upgrade. Both containers needs to be started using **includeAL** in order to extract the AL baseline and allow us to compare our solution to that baseline.

The 14.x container will also be started using **includeCSIDE** in order to extract the C/AL baseline (for creating deltas from our solution). In both containers I decided to use -useBestContainerOS, in order to use process isolation (if possible). Hyperv containers using servercore 2019 and C/AL export has been seen to cause issues.

Last but not least, the 14.x container is created with **runTxt2AlInContainer** set to the 15.x container in order to use the same formatting of AL files in both containers.

New-BCContainer \`
    -accept\_eula \`
    -accept\_outdated \`
    -imageName $imageName15 \`
    -auth UserPassword \`
    -Credential $credential \`
    -containerName $dev15containerName \`
    -licenseFile $licenseFile \`
    -updateHosts \`
    -useBestContainerOS \`
    -includeAL

New-NavContainer \`
    -accept\_eula \`
    -accept\_outdated \`
    -imageName $imageName14 \`
    -auth UserPassword \`
    -Credential $credential \`
    -containerName $dev14containerName \`
    -licenseFile $licenseFile \`
    -updateHosts \`
    -useBestContainerOS \`
    -includeAL -runTxt2AlInContainer $dev15containerName \`
    -includeCSide

# Converting my app

Next step is to convert the code customized C/AL solution to an AL Extension and remove the tableextensions.

Import-ObjectsToNavContainer \`
    -containerName $dev14containerName \`
    -objectsFile $rewardsAppCodeFile

Compile-ObjectsInNavContainer \`
    -containerName $dev14containerName

New-Item -Path $bc15myappFolder -ItemType Directory | Out-Null
Convert-ModifiedObjectsToAl \`
    -containerName $dev14containerName \`
    -startId 50100 \`
    -alProjectFolder $bc15myappFolder

$haveModifiedTables = Test-Path -Path (Join-Path $bc15myappFolder "tableextension\*.\*")
if ($haveModifiedTables) {
    Remove-Item -Path $bc15myappFolder -Filter "tableextension\*.\*" -Recurse -Force
}

The **haveModifiedTables** variable will in our case be true. If we didn’t have modified tables at this time, we would not have to use hybrid at all – we could upgrade directly to an extension.

# Create AL Project Folders

Now, we need to create 3 AL Project Folders. One with 14.x baseline, one with 14.x including our solution and one with 15.x baseline. With these three, we can do a 3 way merge to get a 15.x folder including our solution.

The only modified object we want to keep in the BaseApp is **table18**. If you are trying to do this with your own objects, you would have to include the other objects in the **myModifiedObjects** array for the AL file structure as described in [this blog post](https://freddysblog.com/2019/08/02/organizing-your-al-files/).

$myModifiedObjects = @("table18")
$alFileStructure = { Param (\[string\] $type, \[int\] $id, \[string\] $name)
    if ($myModifiedObjects.Contains("$type$id")) {
        $folder = "Modified"
    }
    elseif (($id -ge 50000 -and $id -le 99999) -or ($id -gt 1999999999)) {
        $folder = "My"
    }
    else {
        $folder = "BaseApp"
    }
    if ($type.StartsWith('.')) {
        "$folder\\$($name)$($type)"
    }
    else {
        "$folder\\$($name).$($type).al"
    }
}

Write-Host "Create 14.x baseline"
Create-AlProjectFolderFromNavContainer \`
    -containerName $dev14containerName \`
    -alFileStructure $alFileStructure \`
    -useBaseAppProperties \`
    -useBaseLine \`
    -alProjectFolder $bc14originalFolder

Write-Host "Create 15.x baseline"
Create-AlProjectFolderFromNavContainer \`
    -containerName $dev15containerName \`
    -alFileStructure $alFileStructure \`
    -useBaseAppProperties \`
    -useBaseLine \`
    -alProjectFolder $bc15originalFolder

Write-Host "Create 14.x mybaseapp"
Create-AlProjectFolderFromNavContainer \`
    -containerName $dev14containerName \`
    -alFileStructure $alFileStructure \`
    -useBaseAppProperties \`
    -alProjectFolder $bc14mybaseappFolder \`
    -runTxt2AlInContainer $dev15containerName

The structure of these folders is like described in [this blog post](https://freddysblog.com/2019/08/02/organizing-your-al-files/).

# Merging

Creating the 15.x folder with our solution, requires us to copy the BaseApp folder from the 15.x baseline and merge the modified folders. In our solution we do not want to keep any of our new objects, which are placed in the 14.x my folder.

![copy-1](/assets/images/2019/upgrading-to-15-x-from-14-x-c-al-our-nav-techdays-2019-demo/copy-1.png)

The script for doing this is here:

New-Item $bc15mybaseappFolder -ItemType Directory | Out-Null
Copy-item -Path "$bc15originalFolder\\\*" -Destination $bc15mybaseappFolder -Recurse -Force
Remove-Item -Path "$bc15mybaseappFolder\\Modified\\\*.\*" -Recurse -Force
& "C:\\Program Files\\KDiff3\\kdiff3.exe" "$bc14originalFolder\\Modified" "$bc14mybaseappFolder\\Modified" "$bc15originalFolder\\Modified" -o "$bc15mybaseappFolder\\Modified" -m

Read-Host "Press Enter When you are done merging"

and the 3 way merge using kdiff3 looks like this:

![Screenshot 2019-11-24 08.41.04](/assets/images/2019/upgrading-to-15-x-from-14-x-c-al-our-nav-techdays-2019-demo/screenshot-2019-11-24-08.41.04.png)

# Building the new BaseApp

With our 3 way merge completed, we can now build the baseapp using the Compile-AppInBCContainer function. Note that we are reusing the AppId, Name, Publisher and version from the Microsoft Base Application in order to ensure all references still works.

$baseAppFile = Compile-AppInBCContainer -containerName $dev15containerName -appProjectFolder $bc15mybaseappFolder -credential $credential

the **baseAppFile** variable contains the path of the new .app file, and this will reside in a folder, which is shared with the container.

# Building my app

When building the extension, which contains all the functionality from our code customized solution, we first need to copy the new base app to the .alPackages folder as we have a reference to it. We could also publish it to the 15.x work container, but there are really no reason to do this.

New-Item -Path (Join-Path $bc15myappFolder '.alPackages') -ItemType Directory | Out-Null
Copy-Item -Path $baseAppFile -Destination (Join-Path $bc15myappFolder '.alPackages')

# Create App.Json
$appjsonTemplate.id = \[GUID\]::NewGuid().ToString()
$appjsonTemplate.name = 'MyApp'
$appjsonTemplate.publisher = 'Freddy'
$appjsonTemplate.target = 'OnPrem'
$appjsonTemplate | ConvertTo-Json -Depth 99 | Set-Content (Join-Path $bc15myappFolder 'app.json')

$appFile = Compile-AppInBCContainer \`
    -containerName $dev15containerName \`
    -credential $credential \`
    -appProjectFolder $bc15myappFolder \`
    -appOutputFolder $bc15myappFolder

With this, we now have a new **baseAppFile** and an **appFile**. Now we can start the actual upgrade.

# The upgrade process

The upgrade process on a very high level looks like:

1.  Uninstall and unpublish all extensions from our production environment.
2.  Remove the 14.x production container.
3.  Create a new 15.x production container, connecting to the production database, invoking database conversion and setting DestinationAppsForMigration to the apps which your existing 14.x base app should migrate to.
4.  Publish 15.x system symbols
5.  Publish the apps from DestinationAppsForMigration
6.  Restart the service tier
7.  Sync the tenants
8.  Sync the apps from DestinationAppsForMigration
9.  Test the database Schema
10.  Invoke the data upgrade
11.  Publish other apps (BingMaps in our case)
12.  Sync other apps
13.  Start data upgrade of other apps
14.  Run the upgrade tests

# Uninstall and unpublish

This seems very easy, but we need to uninstall and unpublish apps in the right order (we cannot uninstall an app on which other apps depend). A script which does this looks like this:

$availableTenants = Invoke-ScriptInBCContainer -containerName $prodContainerName -scriptblock { Get-NAVTenant -ServerInstance $ServerInstance }
$availableTenants | % {
    $tenantId = $\_.Id
    $apps = Get-NavContainerAppInfo -containerName $prodContainerName -tenantSpecificProperties -sort DependenciesLast -tenant $tenantId
    $apps | % {
        $app = $\_
        if ($app.IsInstalled) {
            Invoke-ScriptInBCContainer -containerName $prodContainerName -scriptblock { Param($app, $tenantId)
                Write-Host "Uninstalling $($app.Name)"
                Uninstall-NavApp -ServerInstance $ServerInstance -Name $app.Name -Version $app.Version -Tenant $tenantId -Force
            } -argumentList $app, $tenantId
        }
    }
}

$apps = Get-NavContainerAppInfo -containerName $prodContainerName -sort DependenciesLast -tenant Default
$apps | % {
    Invoke-ScriptInBCContainer -containerName $prodContainerName -scriptblock { Param($app)
        Write-Host "Unpublishing $($app.Name)"
        Unpublish-NAVApp -ServerInstance $ServerInstance -Name $app.Name -Version $app.Version
    } -argumentList $\_
}
Invoke-ScriptInNavContainer -containerName $prodContainerName -scriptblock {
    Get-NAVAppInfo -ServerInstance $ServerInstance -SymbolsOnly | % {
        Write-Host "Unpublishing $($\_.Name) symbols"
        Unpublish-NAVApp -ServerInstance $ServerInstance -Name $\_.Name -Version $\_.Version
    }
}

# Remove 14.x production environment

Removing the “old” production environment is very simple as the database is located on the host, this is just removing the container.

Remove-NavContainer -containerName $prodContainerName

# Create a new 15.x production container

During the creation of the new 15.x container, we need to do a few things.

1.  Connect to the production database on the host. This is done by setting the parameters databaseServer, databaseInstance, databaseName and databaseCredential on New-BCContainer.
2.  Run the database conversion. This is done by overriding the SetupDatabase script and calling Invoke-NavApplicationDatabaseConversion before calling the default SetupDatabase behavior.
3.  Set the DestinationAppsForMigration setting in customSettings.config. We do this by overriding SetupConfiguration because the value contains commas and using the normal CustomNavSettings environment variable wouldn’t work.

We also need to give the container sufficient memory to perform the upgrade. 16G is probably wildly exaggerated for our very small app.

$SystemSymbolsPath = (Get-Item -path (Join-Path $bc15myappFolder '.alPackages\\Microsoft\_System\_\*.app')).FullName

$DestinationAppsForMigrationPaths = \[string\]::Join(',',@(
    "C:\\Applications\\System Application\\Source\\Microsoft\_System Application.app"
    $baseAppFile
    $appFile
    "C:\\Applications\\TestFramework\\TestLibraries\\Assert\\Microsoft\_Library Assert.app"
    "C:\\Applications\\BaseApp\\Test\\Microsoft\_Tests-Upgrade.app"
))

$ThirdPartyAppPaths = "$bingMaps15appFile"

# Calculate value DestinationAppsForMigration value for CustomConfig
$destinationAppsForMigrationJson = Invoke-ScriptInBCContainer -containerName $dev15containerName -scriptblock { Param($paths)
    $destinationAppsForMigration = @()
    $paths.Split(',') | % {
        if ($\_) {
            Write-Host "Destination App $\_"
            $info = Get-NavAppInfo -Path "$\_"
            $destinationAppsForMigration += @(\[ordered\]@{ "appId" = $info.AppId.Value; "name" = $info.Name; "publisher" = $info.Publisher })
        }
    }
    $destinationAppsForMigration | ConvertTo-Json -Depth 99 -Compress
} -argumentList $DestinationAppsForMigrationPaths

# Override script for SetupDatabase to Invoke NavApplicationDatabaseConversion before setting up encryption key
$setupDatabaseScript = @'
if (!$restartingInstance) {
    $databaseServerInstance = $databaseServer
    if ("$databaseInstance" -ne "") {
        $databaseServerInstance += "\\$databaseInstance"
    }
    Write-Host "Invoke Database Conversion"
    Invoke-NAVApplicationDatabaseConversion -databaseServer $databaseServerInstance -ApplicationDatabaseCredentials $databaseCredentials -databasename $databaseName -force | Out-Null
}
. "c:\\run\\SetupDatabase.ps1"
'@

# Override SetupConfiguration to set DestinationAppsForMigration. Cannot be done with CustomNavSettings as the value contains commas
$setupConfigurationScript = @"
. "c:\\run\\SetupConfiguration.ps1"
if (!\`$restartingInstance) {
    Set-NAVServerConfiguration -ServerInstance \`$ServerInstance -KeyName 'DestinationAppsForMigration' -KeyValue '$destinationAppsForMigrationJson' -WarningAction SilentlyContinue | Out-Null
    Set-NAVServerConfiguration -ServerInstance \`$ServerInstance -KeyName 'IntegrationRecordsTableId' -KeyValue '5151' -WarningAction SilentlyContinue | Out-Null
}
"@

New-BCContainer \`
    -accept\_eula \`
    -accept\_outdated \`
    -containerName $prodContainerName \`
    -imageName $imageName \`
    -updateHosts \`
    -auth UserPassword \`
    -Credential $credential \`
    -databaseServer $dbServer \`
    -databaseInstance $dbInstance \`
    -databaseName $dbName \`
    -databaseCredential $dbcredentials \`
    -enableTaskScheduler:$false \`
    -memoryLimit 16G \`
    -doNotUseRuntimePackages \`
    -myscript @( @{ 
        "SetupDatabase.ps1" = $setupDatabaseScript
        "SetupConfiguration.ps1" = $setupConfigurationScript
    })

# The data upgrade

Steps 5 to 13, which performs the actual upgrade, is performed by running a script inside the container.

Invoke-ScriptInBCContainer -containerName $prodContainerName -scriptblock { Param($SystemSymbolsPath, $DestinationAppsForMigrationPaths, $ThirdPartyAppPaths)
    function Invoke-DataUpgrade(\[switch\] $SkipCompanyInitialization, \[string\] $TenantId )
    {
        Write-Host "Running Data upgrade for tenant $tenantId"
        # We will collect information about all errors at once thanks to -ContinueOnError switch
        Start-NAVDataUpgrade -FunctionExecutionMode Serial -ServerInstance $ServerInstance -SkipCompanyInitialization:$SkipCompanyInitialization -Tenant $TenantId -SkipAppVersionCheck -ErrorAction Stop -Force

        # Wait for Upgrade Process to complete
        Get-NAVDataUpgrade -ServerInstance $ServerInstance -Tenant $TenantId -Progress -ErrorAction Stop

        # Make sure that Upgrade Process completed successfully.
        $errors = Get-NAVDataUpgrade -ServerInstance $ServerInstance -Tenant $TenantId -ErrorOnly -ErrorAction Stop

        if (!$errors) {
            # no errors detected - process has been completed successfully
            Write-Host "Data upgrade completed succesfully for $TenantId."
            return
        }

        # Stop the suspended process - we won't resume in here
        Stop-NAVDataUpgrade -ServerInstance $ServerInstance -Tenant $TenantId -Force -ErrorAction Stop

        $errorMessage = "Errors occured during Data Upgrade Process: " + \[System.Environment\]::NewLine
        foreach ($nextErrorRecord in $errors) {
            $errorMessage += ("Codeunit ID: " + $nextErrorRecord.CodeunitId + ", Function: " + $nextErrorRecord.FunctionName + ", Error: " + $nextErrorRecord.Error + \[System.Environment\]::NewLine)
        }
        Write-Error $errorMessage
    }

    Write-Host "Publishing system symbols - $SystemSymbolsPath"
    Publish-NAVApp -ServerInstance $ServerInstance -Path $SystemSymbolsPath -PackageType SymbolsOnly -SkipVerification

    # Publish Migration Apps
    $DestinationAppsForMigrationPaths.Split(',') | % {
        $appPath = $\_
        Write-Host "Publishing app - $appPath"
        Publish-NAVApp -Path $appPath -ServerInstance $ServerInstance -PackageType Extension -SkipVerification
    }

    # Restart the server to free up resources
    Write-Host "Restarting Service Tier"
    Restart-NAVServerInstance -ServerInstance $ServerInstance

    Write-Host "Synchronizing tenants"
    $availableTenants = Get-NAVTenant -ServerInstance $ServerInstance
    $availableTenants | % {
        $tenantId = $\_.Id
        Write-Host "Synchronizing tenant $tenantId"
        Sync-NavTenant -ServerInstance $ServerInstance -Tenant $tenantId -Force
    }

    # Sync Migration apps
    # We move tables and generate SystemIds here
    $DestinationAppsForMigrationPaths.Split(',') | % {
        $appPath = $\_
        $availableTenants | % {
            $tenantId = $\_.Id
            $info = Get-NavAppInfo -Path $appPath
            Write-Host "Synchronizing $($info.Name) for tenant $tenantId"
            Sync-NAVApp -Name $info.Name -ServerInstance $ServerInstance -Tenant $tenantId 
        }
    }

    # Verify if all extensions were moved and metedata is correct
    $availableTenants | % {
        $tenantId = $\_.Id
        Write-Host "Testing Database Schema for tenant $tenantId"
        Test-NAVTenantDatabaseSchema -ServerInstance $ServerInstance -Tenant $tenantId
    }

    # Invoke Data upgrade
    # This cmdlet will automatically install and upgrade all of the extensions listed in DestinationAppsForMigration
    $availableTenants | % {
        $tenantId = $\_.Id
        # Calling upgrade will install DestinationAppsForMigration during upgrade. OnInstall triggers will not be invoked in this mode.
        Invoke-DataUpgrade -Tenant $tenantId
    }

    #Publish and upgrade other extensions
    $ThirdPartyAppPaths.Split(',') | % {
        $appPath = $\_
        Write-Host "Publishing app - $appPath"
        Publish-NAVApp -Path $appPath -ServerInstance $ServerInstance -PackageType Extension -SkipVerification
    }

    # Sync all third party extensions
    $ThirdPartyAppPaths.Split(',') | % {
        $appPath = $\_
        $availableTenants | % {
            $tenantId = $\_.Id
            $info = Get-NavAppInfo -Path $appPath
            Write-Host "Synchronizing $($info.Name) for tenant $tenantId"
            Sync-NAVApp -Name $info.Name -ServerInstance $ServerInstance -Tenant $tenantId 
        }
    }

    # Upgrade all third party extensions
    $ThirdPartyAppPaths.Split(',') | % {
        $appPath = $\_
        $availableTenants | % {
            $tenantId = $\_.Id
            $info = Get-NavAppInfo -Path $appPath
            Write-Host "Upgrading $($info.Name) for tenant $($tenantId)"
            Start-NAVAppDataUpgrade -Name $info.Name -ServerInstance $ServerInstance -Tenant $tenantId -Force
        }
    }
} -argumentList $SystemSymbolsPath, $DestinationAppsForMigrationPaths, $ThirdPartyAppPaths

There you are – opening the Web Client and looking at our data reveals that, customizations, apps and data has been transferred:

![Screenshot 2019-11-24 11.48.54](/assets/images/2019/upgrading-to-15-x-from-14-x-c-al-our-nav-techdays-2019-demo/screenshot-2019-11-24-11.48.54.png)

# Running upgrade tests

Last but not least, lets import the test toolkit and run the upgrade tests.

Import-TestToolkitToBCContainer -containerName $prodContainerName -includeTestLibrariesOnly -doNotUseRuntimePackages
Run-TestsInBCContainer -containerName $prodContainerName -extensionId "d0e99b97-089b-449f-a0f5-a2ab994dbfd7" -detailed -credential $credential

Looking at the extensions installed reveals my new base app version 15.1 (named the same as Microsofts), the BingMaps app, MyApp and all the Test Apps:

![Screenshot 2019-11-24 11.50.59](/assets/images/2019/upgrading-to-15-x-from-14-x-c-al-our-nav-techdays-2019-demo/screenshot-2019-11-24-11.50.59.png)

# Conclusion

As described, there are still some obstacles before 14.x C/AL to 15.x AL works smoothly. As soon as we have a path, where we can do the above process without the base app, I will write a new blog post with the process without the base app. This blog post will stay as covering a hybrid upgrade model.

The **full script** of the process above can be downloaded [here](https://bcdocker.blob.core.windows.net/public/DemoScript.ps1) and if you want to know more about upgrade, you can find information here: [http://aka.ms/bcupgradedeck](http://aka.ms/bcupgradedeck).

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
