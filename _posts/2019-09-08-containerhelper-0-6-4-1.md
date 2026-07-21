---
layout: post
title: "ContainerHelper 0.6.4.1"
date: 2019-09-08 08:55:45
categories: ["CI/CD", "Docker", "NavContainerHelper"]
tags: ["Docker", "NAV on Docker", "NavContainerHelper", "new-navcontainer"]
permalink: /2019/09/08/containerhelper-0-6-4-1/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

Now and then, I decide to blog about a version of NavContainerHelper. The version shipped today ([0.6.4.1](https://www.powershellgallery.com/packages/navcontainerhelper/0.6.4.1)) is one of these.

# What’s  new

-   Backup/Restore databases in containers
-   Speed up repetitive container generation
-   Flush cache folders used by the ContainerHelper
-   Run-Tests have become more resilient

and some small bugfixes + enhancements.

# Backup/Restore databases in containers

**Backup-BCContainerDatabases** has existed for a long time and the functionality has only changed slightly. Now, the default is to backup all tenant databases if the container is multi-tenant.

If you don’t specify the bakFolder, then the backup will be placed in the container folder (_c:\\ProgramData\\NavContainerHelper\\Extensions\\<containername>_) and be removed when the container is deleted.

If you specify a simple name in bakFolder, the backup will be placed in a version specific folder, which can be reused in other containers with the same version, here: _c:\\ProgramData\\NavContainerHelper\\<containerversion>-bakfolders\\<bakfolder>_.

If you specify a full path in bakFolder, the backup will be placed there.

**Restore-DatabasesInBCContainer** can either restore a single database in a container or it can restore the databases backed up by Backup-BCContainerDatabases.

When restoring a single database backup file using the -bakFile parameter, the database is restored to a name and location determined by yourself and if you want to mount/use the database, you need to reconfigure the service tier.

When restoring all databases in a container using the -bakFolder parameter, the service tier is stopped, the .bak file(s) in the folder will be restored in the container and the service tier is restarted.

New parameter **\-bakFolder on New-BCContainer** can be used to start a container using the databases from a .bakFolder created by Backup-BCContainerDatabases or create the databases itself.

# Scenarios for backup restore

## Create/Restore snapshot of databases in the container

After creating your container, you can run

Backup-BCContainerDatabases -containername <containername>

To create a snapshot of the databases. When you later want to restore the snapshot, you just run

Restore-DatabasesInBCContainer -containername <containername>

and you can of course do this as many times you want.

## Speed up repetitive container generation

When you create containers for build scenarios or in other examples recreate the same containers again and again, you can add -bakFolder to New-BCContainer, which causes the databases in the container to be backed up after container generation is completed.

The database backups will be stored in a location determined by the bakFolder parameter (_rules from Backup-BCContainerDatabases apply_). If you run the exact same New-BCContainer again, the databases from this bakFolder will be restored and used in the new container.

Try to run this script twice:.

$containername = 'test'
$navdockerimage = 'mcr.microsoft.com/businesscentral/sandbox:dk'
$license = 'c:\\temp\\build.flf'
$credential = New-Object pscredential 'admin', (ConvertTo-SecureString -String 'P@ssword1' -AsPlainText -Force)
Measure-command {
    New-NavContainer -accept\_eula \`
                     -containername $containername \`
                     -auth UserPassword \`
                     -Credential $credential \`
                     -updateHosts \`
                     -assignPremiumPlan \`
                     -imageName $navdockerimage \`
                     -includeTestToolkit \`
                     -licenseFile $license \`
                     -finalizeDatabasesScriptBlock { 
                         Setup-NavContainerTestUsers -containerName $containername -password $credential.Password -credential $credential
                     } \`
                     -bakFolder "mysetup"
}

On my machine, the first run takes ~4 minutes and the second run takes ~1 minute and 40 seconds. The following things are skipped during the second run, as they are assumed to already be in the database backup:

-   Importing license file is skipped
-   The finalizeDatabasesScriptBlock is NOT executed
-   Premium plan is not assign to the user
-   The testtoolkit is not imported

Basically you can use the bakFolder parameter to avoid pre-creating backups or images to speed up things Just-In-Time.

_**Note:** Please do understand the functionality of -bakFolder before you put it to use as you might have other parameters or scripts doing other things to the container conflicting with the functionality._

## Restore a single database

If you have a database you want to restore into a running container, you can do this using Restore-DatabaseInBcContainer as well:

New-NavContainer -accept\_eula \`
                 -accept\_outdated \`
                 -containerName "to" \`
                 -imageName "mcr.microsoft.com/businesscentral/onprem:1904-cu1-w1" \`
                 -auth UserPassword \`
                 -Credential $credential \`
                 -updateHosts \`
                 -includeCSIDE \`
                 -doNotExportObjectsToText

Restore-DatabasesInBCContainer -containerName "to" \`
                               -bakFile "c:\\programdata\\navcontainerhelper\\converttest\\database.bak" \`
                               -databaseName "newdb" \`
                               -databaseFolder "c:\\databases\\newdb"

Connecting to the container with SSMS (SQL Server Management Studio), shows that we indeed have 2 databases:

![ssms](/assets/images/2019/containerhelper-0-6-4-1/ssms.png)

# Flush ContainerHelper Cache Folders

NavContainerHelper uses cache folders for a number of items. All cache folders are redundant information and is primarily used for performance reasons.

**C/AL Source Cache** is populated with C/AL Source code for a version/country of NAV/BC when you start a container with -includeCSIDE (without -doNotExportObjectsAsText)

**AL Source Cache** is populated with C/AL Source code for a version/country of Business Central when you start a container with -includeAL (without -doNotExportObjectsAsText)

**Files Cache** is used when you create a new container with the -useBestContainerOS flag. This will extract the necessary files from a container and place them in the cache, in order to spin up a generic image using these files.

**Application Cache** (new with 15.x) is used to store runtime packages of apps. When publishing test toolkit to 15.x containers it takes 10+ minutes (on my beefy laptop) the first time. Subsequent attempts to run the same container will grab the runtime packages from the cache and cut the time to less than half.

**BakFolder Cache** (new with 0.6.4.0) is used to store backup sets from containers as described in the previous section.

In 0.6.4.0, a new command called **Flush-ContainerHelperCache** was introduced. If you do not specify which cache to flush, it will flush all the cache folders described above.

# Run-Tests have become more resilient

If you have used Run-TestsInNavContainer, you might have experienced that sometimes you get an error stating: **ClientSession in error**, and the test execution is stopped. There can be many reasons for this error. Typically it is caused by an unhandled exception somewhere, which again can be due to lack of memory, communication errors or other things.

Beside some general improvements to run-tests, two new parameters was added to Run-TestsInNavContainer:

**\-restartContainerAndRetry** will, if an unhandled exception occurs, restart the container and re-run the exact same Run-TestsInNavContainer.

**\-debugMode** will print out some information about the pages currently opened in the client session opened by run-tests.

The following script shows one way of running all tests, including a rerun of failed tests:

$tests = Get-TestsFromBCContainer -containerName $containerName -credential $credential -ignoreGroups
$rerunTests = $()
$failedTests = $()
$first = $true
$tests | % {
    if (-not (Run-TestsInBcContainer -containerName $containerName \`
                                     -credential $credential \`
                                     -XUnitResultFileName 'c:\\programdata\\navcontainerhelper\\results.xml' \`
                                     -AppendToXUnitResultFile:(!$first) \`
                                     -testCodeunit $\_.Id \`
                                     -returnTrueIfAllPassed \`
                                     -restartContainerAndRetry)) { $rerunTests += $\_ }
    $first = $false
}
if ($rerunTests.Count -gt 0) {
    Restart-BCContainer -containerName $containername
    $rerunTests | % {
        if (-not (Run-TestsInBcContainer -containerName $containerName \`
                                         -credential $credential \`
                                         -XUnitResultFileName 'c:\\programdata\\navcontainerhelper\\results.xml' \`
                                         -AppendToXUnitResultFile:(!$first) \`
                                         -testCodeunit $\_.Id \`
                                         -returnTrueIfAllPassed \`
                                         -restartContainerAndRetry)) { $failedTests += $\_ }
        $first = $false
    }
}

Enjoy

_**Freddy Kristiansen**_  
_Technical Evangelist_
