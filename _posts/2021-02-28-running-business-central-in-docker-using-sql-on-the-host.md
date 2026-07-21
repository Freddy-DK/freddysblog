---
layout: post
title: "Running Business Central in Docker using SQL on the host"
date: 2021-02-28 11:02:55
categories: ["BcContainerHelper", "Docker"]
tags: ["Database", "Remove-BcDatabase", "ReplaceExternalDatabases", "Restore-BcDatabaseFromArtifacts", "SQL on Host"]
permalink: /2021/02/28/running-business-central-in-docker-using-sql-on-the-host/
---

This is not my first blog post about how to use SQL Server on the host, but it is definitely the one describing the easiest way to do it. With the latest version of BcContainerHelper you can (with one Run-BcContainer command) create a container which uses SQL Server on the host as database engine for the container.

The functionality was actually enabled in version 1.0.14, but there are a few bug fixes, so I recommend that you use the latest BcContainerHelper

# Prerequisites

In order for the scripts in this blog post to work, you need to install SQL Server on the Docker Host with mixed mode authentication. The scripts will connect using Windows Authentication and the container will connect using Username/Password authentication.

# ReplaceExternalDatabases

New-BcContainer has a new parameter called –**replaceExternalDatabases**. This switch will only take effect if you also specify **databaseServer**, **databasePrefix**, **databaseName** and **databaseCredential**.

When ReplaceExternalDatabases is included, the New-BcContainer will do 4 things, which it typically doesn’t do:

1.  Remove all databases on **databaseServer** with **databaseInstance** (might be empty) where the name starts with **databasePrefix**. This is done using a new function called **_Remove-BcDatabase_**.
2.  Restore databases (single or multitenant) from the **selected artifacts** to **databaseServer** with **databaseInstance** (might be empty) starting with **databasePrefix**. This is done using a new function called **_Restore-BcDatabaseFromArtifacts_**.
3.  Start the container using **databaseServer** with **databaseInstance** (might be empty) as database server and **databasePrefix+databaseName** as database name, connecting using **databaseCredential** for authentication.
4.  Create the **default tenant** (if multitenant) and add the default super user specified in **$credential**.

The following script should setup a container using SQL Server on the host:

```
$password = $passwordSecret.SecretValue
$licenseFile = $licenseFileSecret.SecretValueText
$auth = "UserPassword"
$credential = New-Object pscredential -ArgumentList 'admin', $password
$artifactUrl = Get-BcArtifactUrl -country us

$containerName = "bcserver"

$databaseParams = @{
    "databaseServer" = 'host.containerhelper.internal'
    "databaseInstance" = ''
    "databasePrefix" = "$containerName-"
    "databaseName" = 'CRONUS'
    "databaseCredential" = New-Object pscredential 'sa', $password
    "multitenant" = $true
}

New-BcContainer @databaseParams -replaceExternalDatabases `
    -accept_eula `
    -containerName $containerName `
    -credential $credential `
    -auth $auth `
    -artifactUrl $artifactUrl `
    -licenseFile $licenseFile `
    -updateHosts
```

**Note:** It uses $containerName followed by a minus as database prefix, meaning that all databases on the SQL Server on the host with a name starting with **bcserver-** will be deleted according to the #1 in the description above.

After running this script, I will have 3 databases in my SQL Server:

![](/assets/images/2021/running-business-central-in-docker-using-sql-on-the-host/image.png)

No SQL processes running in my container:

![](/assets/images/2021/running-business-central-in-docker-using-sql-on-the-host/image-1.png)

and after some initialization, a running Business Central Container:

![](/assets/images/2021/running-business-central-in-docker-using-sql-on-the-host/image-2.png)

**Note:** If you are using an SQL Server (not the host), you need the SQL Server Client Tools installed on the host – the functions **_Remove-BcDatabase_** and **_Restore-BcDatabaseFromArtifacts_** (used by the replaceExternalDatabases functionality) both have a dependency on **SQLPS** and the **smoServer** DLL.

If you want to create a second container, using the same database, you can use this script:

```
New-BcContainer @databaseParams `
    -accept_eula `
    -containerName "$($containerName)2" `
    -credential $credential `
    -auth $auth `
    -artifactUrl $artifactUrl `
    -updateHosts
```

Only difference is really that the **licenseFile** and the **replaceExternalDatabases** parameters are NOT included.

**Note:** In the latest BcContainerHelper, the containerHelper will automatically share encryption keys between containers on the same host, which are connecting to the same external database. If you are creating a second container on a different host, you will need to copy the encryption key to the second host yourself.

# Remove-BcDatabase

The replaceExternalDatabases functionality uses the _**Remove-BcDatabase**_ function to remove existing databases. You can of course use this function manually as well, simply by specifying **databaseServer**, **databaseInstance** (if not blank) and **databaseName** (or pattern), like:

![](/assets/images/2021/running-business-central-in-docker-using-sql-on-the-host/image-3.png)

**Note:** that this is a dangerous command to run on a SQL Server.

# Restore-BcDatabasesFromArtifacts

The replaceExternalDatabases functionality uses the **_Restore-BcDatabasesFromArtifacts_** function to restore the database backup file from a set of artifacts to a database server specified by **databaseServer**, **databaseInstance** (or blank) and **databaseName**, like:

```
$artifactUrl = Get-BCArtifactUrl -country dk
Restore-BcDatabaseFromArtifacts `
    -artifactUrl $artifactUrl `
    -databaseServer localhost `
    -databasePrefix prefix `
    -databaseName CRONUS `
    -multitenant:$false `
    -async
```

Specifying -async means that the function won’t wait for the restore to complete. A file with the databasePrefix followed by databasescreated.txt will be created in the containerhelper folder when the restore is complete. Without async, the function will wait for the restore to complete.

**Note:** The New-BcContainer uses async and will add an override to the container SetupDatabase.ps1 script which waits for the databasescreated file to appear. If something fails in the database restore, this might lead to a endless loop.

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
