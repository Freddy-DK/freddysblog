---
layout: post
title: "Placing the database on the host…"
date: 2019-04-12 09:45:02
categories: ["Docker", "NavContainerHelper"]
tags: ["Database", "Docker", "NavContainerHelper"]
permalink: /2019/04/12/placing-the-database-on-the-host/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

I have had quite a few partners asking me how to connect a NAV / Business Central container to a SQL Server placed on the host. Even though this sounds like a simple question, it isn’t easy to answer.

# First, why would you do this?

When you start a standard NAV or Business Central Container, you will have everything installed in the container, Business Central, Web Server, SQL Server and the database folder containing the database files.

![bccontainerandhost](/assets/images/2019/placing-the-database-on-the-host/bccontainerandhost.png)

If you just want to run a container with the standard CRONUS database, there is IMO no reason at all to not just use the built-in database. For most AL Extension Development that is the case. But, what if you have your own database backup with a lot of demo data, which is necessary for your development. In that case, the database backup file might take forever to restore into a container and the container might not even have space for the database.

For this reason, a lot of people turn to the solution of trying to host the database server on the host and connecting the container to that.

![bccontainerandhost2](/assets/images/2019/placing-the-database-on-the-host/bccontainerandhost2.png)

It just isn’t that easy…

# 2 different machines

The reason why it isn’t that easy is, that the host and the container are 2 different machines, even if you are running Process Isolation.

Normally when running SQL Server on the same machine as the Service Tier, you will used the Shared Memory protocol in SQL Server and as such you don’t have to enable TCP/IP, open firewalls, handle host name resolution,  etc. – it just works.

When running SQL Server on a different machine you need to handle all these things and to add to the complexity, the computer might have a dynamic IP address and if you are running containers with NAT networking, you container cannot directly see the host.

So what can we do?

# Place the database on the host, not the database server!

The solution to this problem is IMO to restore your database backup to a container and then extract the database files (.mdf + .ldf) and place them on the host in a folder, which is shared with the Container.

Then override the SetupDatabase script to attach these files to the SQL Server inside the container.

![bccontainerandhost3](/assets/images/2019/placing-the-database-on-the-host/bccontainerandhost3.png)

This drawing is slightly misleading as the database server in the container actually doesn’t know that the file is on the host – for the database server, the file is local.

What are the benefits of this:

-   Attaching a database is very very fast compared to restoring a backup
-   The database files will survive renewal of the container
-   The database file will not be limited by the disk size limit of the container (default 20gb)
-   Accessing the host file system is faster than the docker file system (although with the CRONUS database it doesn’t make a big difference)
-   It is very very easy to setup

and the flipside:

-   Only one container can attach the same set of database files. If you used a SQL Server on the host you could connect
-   The .mdf and .ldf files are SQL Server version specific. If the container changes SQL Server version, you might need to convert the files through a standard backup file to the new version. This happens very very infrequently though.

# Enough with the theory, lets see the code!

First of all, you need the .mdf + .ldf files. There is a function in NavContainerHelper called **Extract-FilesFromNavContainerImage** which can extract selected files from a container image. Running this code:

$imageName = "mcr.microsoft.com/businesscentral/onprem:w1-ltsc2019"
$path = "c:\\temp\\hostdbfolder"
Extract-FilesFromNavContainerImage -imageName $imageName -path $path -extract database -force

will place the database files from the latest BusinessCentral on premises container in c:\\temp\\hostdbfolder\\databases.

There is another function called **Extract-FilesFromStoppedNavContainer**, which can extract the files from a stopped container. This function will only work with a single tenant container, where the database is in its original location. If you need this to work with multi-tenancy, you can probably easily decipher how this is done here: [https://github.com/Microsoft/navcontainerhelper/blob/dev/ContainerHandling/Extract-FilesFromStoppedNavContainer.ps1](https://github.com/Microsoft/navcontainerhelper/blob/dev/ContainerHandling/Extract-FilesFromStoppedNavContainer.ps1)

Creating a container, using a new database created by attaching the .mdf + .ldf files on the host, can be done using this script:

$credential = New-Object pscredential 'admin', (ConvertTo-SecureString -String 'P@ssword1' -AsPlainText -Force)
$containerName = "mytemp"
$attachdbSetupDatabaseScript = "https://raw.githubusercontent.com/Microsoft/nav-docker/master/override/attachdb/SetupDatabase.ps1"
New-NavContainer -accept\_eula \`
                 -imageName $imageName \`
                 -containerName $containerName \`
                 -additionalParameters @("--volume $path\\databases:c:\\mydb") \`
                 -myScripts @($attachdbSetupDatabaseScript) \`
                 -auth "NavUserPassword" \`
                 -Credential $credential \`
                 -updateHosts

Basically, you share the folder with the .mdf + .ldf files to c:\\mydb inside the container and then you override the SetupDatabase script with the script located here:  [https://raw.githubusercontent.com/Microsoft/nav-docker/master/override/attachdb/SetupDatabase.ps1](https://raw.githubusercontent.com/Microsoft/nav-docker/master/override/attachdb/SetupDatabase.ps1)

That’s it.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
