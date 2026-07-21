---
layout: post
title: "Mounting a database backup from my online environment inside a container"
date: 2019-11-12 07:35:13
categories: ["Demo Environments", "Docker", "NavContainerHelper", "PowerShell"]
tags: ["Business Central Sandbox", "Docker", "Multitenancy", "NavContainerHelper"]
permalink: /2019/11/12/mounting-a-database-backup-from-my-online-environment-inside-a-container/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

Just recently, a new functionality was enable in the Dynamics 365 Business Central admin center. The ability to request a backup. It didn’t take long before I got the first question from a partner, who asked whether they could run this locally using Docker. This blog post describes how to do just that. In this blog post, **I will mount the database inside Docker**, which won’t work if your online database exceeds 10Gb (due to SQL EXPRESS). The next blog post will describe how to do the same using a SQL Server on the host.

# **Please Note:** Not for production

_**Please note** that following this blog post will not give you an environment you can use for production. First of all, there might be extensions installed in your online environment, which you cannot install locally and furthermore Docker is not supported for production even if Docker would have been supported for production, we would definitely not support sandbox containers in production (as they simulate online and there are a number of things that doesn’t work in sandbox containers)._

# Requesting the backup

You will find the database export functionality in the admin center. Select the environment and select Database -> Create Database Export and you will see something like this:

![Screenshot 2019-11-10 09.26.17](/assets/images/2019/mounting-a-database-backup-from-my-online-environment-inside-a-container/screenshot-2019-11-10-09.26.17.png)

Only thing you need to specify is a **SAS URI** for a blob storage, where you want to receive the backup. Clicking the small (i) after SAS URI will take you to [this document](https://docs.microsoft.com/en-us/dynamics365/business-central/dev-itpro/administration/tenant-admin-center-database-export), which describes how to create a SAS URI. After requesting the backup you will have to wait for some time before the backup arrives in your storage account.

# Downloading the backup

When the backup arrives in the storage account, you will see a blob.

![Screenshot 2019-11-10 11.04.02](/assets/images/2019/mounting-a-database-backup-from-my-online-environment-inside-a-container/screenshot-2019-11-10-11.04.02.png)

Clicking the blob, you can download the .bacpac file locally and use that, but you can also generate a SAS URL, which also be be used when creating a container. The latter will do an on-demand download of the database and a sample SAS URL will look something like:

[https://nav2016wswe0.blob.core.windows.net/productionexports/Production\_20191110\_02.bacpac?sp=r&st=2019-11-10T10:05:22Z&se=2019-11-10T18:05:22Z&spr=https&sv=2019-02-02&sr=b&sig=H8FbmVqy%2B1Tns%2FhiCcP7dWUEnRY4IAn%2FXsLSsKuLQsk%3D](https://nav2016wswe0.blob.core.windows.net/productionexports/Production_20191110_02.bacpac?sp=r&st=2019-11-10T10:05:22Z&se=2019-11-10T18:05:22Z&spr=https&sv=2019-02-02&sr=b&sig=H8FbmVqy%2B1Tns%2FhiCcP7dWUEnRY4IAn%2FXsLSsKuLQsk%3D)

# What docker image to use

Having downloaded the .bacpac file (or created a direct download URL to the file), next issue is to determine what docker image to use. **The .bacpac is a tenant database** and **doesn’t contain the app database**. Furthermore it is stamped with a database version number, meaning that you need to use a platform and an app, which is capable of mounting the database.

Onprem images are only shipped monthly and they don’t contain the same apps as the sandbox images. The likelihood that you would be able to use an onprem image (and thus also an onprem version installed from a downloaded DVD) is slim.

Now, you could try to determine the exact version of your app online and see if you can find a matching image. In your online environment, you can navigate to the Help and Support page to get the platform and app version:

![Screenshot 2019-11-11 11.29.19](/assets/images/2019/mounting-a-database-backup-from-my-online-environment-inside-a-container/screenshot-2019-11-11-11.29.19.png)

Now, docker images are tagged after application version (as multiple app versions might share the same platform). So you could take the docker version with the version tag 15.0.36560.0, but if we inspect the labels of that image, we will see:

![labels](/assets/images/2019/mounting-a-database-backup-from-my-online-environment-inside-a-container/labels.png)

Platform version 15.0.36510.0, which means that my online tenant apparently have some platform updates.

We can investigate all existing docker images using this small script:

$repo = "mcr.microsoft.com/businesscentral/sandbox"
(Get-NavContainerImageTags -imageName $repo).Tags | Where-Object { $\_ -like "15.\*-us-ltsc2019" } | % {
    $labels = Get-NavContainerImageLabels -imageName "$($repo):$\_"
    Write-Host "Platform: $($labels.Platform)   App: $($labels.version)"
}
Platform: 15.0.36510.0   App: 15.0.36560.0
Platform: 15.0.36510.0   App: 15.0.36560.36626
Platform: 15.0.36510.0   App: 15.0.36590.0
Platform: 15.0.36510.0   App: 15.0.36626.36675
Platform: 15.0.36895.0   App: 15.0.36626.36918
Platform: 15.0.36924.0   App: 15.0.36626.36938
Platform: 15.0.36924.0   App: 15.0.36626.36966
Platform: 15.0.36985.0   App: 15.0.36626.37046
Platform: 15.0.36985.0   App: 15.0.36626.37056
Platform: 15.0.36985.0   App: 15.0.36626.37063
Platform: 15.0.37259.0   App: 15.0.36626.37299
Platform: 15.0.37259.0   App: 15.0.36626.37307
Platform: 15.0.37259.0   App: 15.0.36626.37394
Platform: 15.0.37395.0   App: 15.0.36626.37481
Platform: 15.0.37492.0   App: 15.0.36626.37536
Platform: 15.0.37492.0   App: 15.0.36626.37543
Platform: 15.0.37582.0   App: 15.0.36626.37679
Platform: 15.0.37582.0   App: 15.0.36626.37711
Platform: 15.0.37867.0   App: 15.0.36626.37934

and find that no image fits directly. My recommandation is to just use the latest image, you should be able to mount and sync the database on that.

# Create a multitenant container

As always, I assume that you have the latest NavContainerHelper from the PowerShell gallery [here](https://www.powershellgallery.com/packages/navcontainerhelper), the following will probably work with older versions as well, but my first response to an issue will always be to **update NavContainerHelper and try again**.

First thing we need to do is to create a multitenant container using the latest sandbox image with us localization.

$auth = "UserPassword"
$credential = New-Object PSCredential 'admin', (ConvertTo-SecureString -String 'P@ssword1' -AsPlainText -Force)
$imageName = "mcr.microsoft.com/businesscentral/sandbox:us-ltsc2019"
$containerName = "test"

New-BCContainer \`
    -accept\_eula \`
    -accept\_outdated \`
    -containerName $containerName \`
    -imageName $imageName \`
    -auth $auth \`
    -Credential $credential \`
    -multitenant \`
    -alwaysPull \`
    -updateHosts \`
    -licenseFile 'c:\\temp\\license.flf'

Next thing is to restore the .bacpac in the container, mount the database as a tenant, synchronize and create a user in the tenant. Note that I have placed the .bacpac file in a location, which is shared with the container.

$tenantBacpac = "C:\\ProgramData\\NavContainerHelper\\Production\_20191110\_02.bacpac"
$tenantId = "mydata"
Invoke-ScriptInBCContainer -containerName $containerName -scriptblock { Param($tenantId, $tenantBacpac)
    Restore-BacpacWithRetry \`
        -bacpac $tenantBacpac \`
        -databasename $tenantId \`
        -maxattempts 1
    Mount-NavTenant \`
        -ServerInstance $ServerInstance \`
        -id $tenantId \`
        -databasename $tenantId \`
        -databaseserver localhost \`
        -databaseinstance SQLEXPRESS \`
        -EnvironmentType Sandbox \`
        -OverwriteTenantIdInDatabase \`
        -Force
    Sync-NavTenant \`
        -ServerInstance $ServerInstance \`
        -Tenant $tenantId \`
        -Force
} -argumentList $tenantId, (Get-BCContainerPath -containerName $containerName -path $tenantBacpac)
New-NavContainerNavUser \`
    -containerName $containerName \`
    -Credential $credential \`
    -ChangePasswordAtNextLogOn:$false \`
    -PermissionSetId SUPER \`
    -tenant $tenantId

and basically that’s it.

# Unless…

Unless you have installed any applications which are not standard MS apps.

The sandbox containers comes with all standard MS apps pre-installed, but if you have installed apps from appsource or installed per-tenant applications, the script above will fail, saying something like this:

![missing](/assets/images/2019/mounting-a-database-backup-from-my-online-environment-inside-a-container/missing.png)

In my case, I have the source for both extensions, so I could just install them before mounting, but what if I didn’t?

Fortunately, we can manage both cases. Publishing apps are done using the **Publish-BCContainerApp** function and removing extensions are done by deleting the corresponding record in the tables **NAV App Published App** and **NAV App Installed App**. Note that by removing the apps from these tables we kind of invalidate the schema. There might be tables in the tenant database now which aren’t used, but there is no code referencing the tables, so everything should be ok. Of course, you will not be able to see or touch any data in tables or fields belonging to not-installed extensions.

Instead of the above script, you would have to run a script like the script below to **publish the BingMaps app** and **delete the MyApp app** reference:

$tenantBacpac = "C:\\ProgramData\\NavContainerHelper\\Production\_20191110\_02.bacpac"
$tenantId = "mydata"
$removeApps = @("MyApp")
Publish-BCContainerApp \`
    -containerName $containerName \`
    -appFile "C:\\Users\\freddyk\\Downloads\\Freddy Kristiansen\_BingMaps\_15.0.56.0.app" \`
    -skipVerification \`
    -sync \`
    -install
Invoke-ScriptInBCContainer -containerName $containerName -scriptblock { Param($tenantId, $tenantBacpac, $removeApps)
    Restore-BacpacWithRetry \`
        -bacpac $tenantBacpac \`
        -databasename $tenantId \`
        -maxattempts 1
    $removeApps | ForEach-Object {
        Invoke-Sqlcmd -Query "USE \[$tenantId\]
GO
DELETE FROM \[dbo\].\[NAV App Published App\]
WHERE Name = '$\_'
DELETE FROM \[dbo\].\[NAV App Installed App\]
WHERE Name = '$\_'
GO"
    }
    Mount-NavTenant \`
        -ServerInstance $ServerInstance \`
        -id $tenantId \`
        -databasename $tenantId \`
        -databaseserver localhost \`
        -databaseinstance SQLEXPRESS \`
        -EnvironmentType Sandbox \`
        -OverwriteTenantIdInDatabase \`
        -Force
    Sync-NavTenant \`
        -ServerInstance $ServerInstance \`
        -Tenant $tenantId \`
        -Force 
} -argumentList $tenantId, (Get-BCContainerPath -containerName $containerName -path $tenantBacpac), $removeApps

New-NavContainerNavUser \`
    -containerName $containerName \`
    -Credential $credential \`
    -ChangePasswordAtNextLogOn:$false \`
    -PermissionSetId SUPER \`
    -tenant $tenantId

Now, I can logon to mydata tenant in my container using [http://test/BC?tenant=mydata](http://test/BC?tenant=mydata) and get:

![Screenshot 2019-11-12 07.20.34](/assets/images/2019/mounting-a-database-backup-from-my-online-environment-inside-a-container/screenshot-2019-11-12-07.20.34.png)

and I can see that my tenant data is preserved, my BingMaps extension is still installed, configured and data (geocoding) is preserved.

As stated in the beginning, this only works for databases which are less than 10Gb in size due to SQL EXPRESS. In the next blog post, I will describe how to do this with SQL Server Developer edition installed on the host.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
