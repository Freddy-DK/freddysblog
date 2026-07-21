---
layout: post
title: "Restoring your online Business Central database locally"
date: 2021-03-02 18:47:21
categories: ["BcContainerHelper", "BcSaaS", "Docker", "PowerShell"]
tags: ["Business Central", "Multitenancy", "restore", "Saas", "SQL on Host"]
permalink: /2021/03/02/restoring-your-online-business-central-database-locally/
---

1½ years ago I wrote a blog post called [Mounting a database from my online environment using SQL Server on the host](/2019/11/12/mounting-a-database-from-my-online-environment-using-sql-server-on-the-host/). This blog post explains exactly the same thing, just end 2 end and much easier to understand (I hope), using artifacts and some new functions in BcContainerHelper.

The steps involved in restoring your online Business Central database locally are:

1.  Installing/configuring prerequisites
2.  Determine artifacts based on the version of your online Business Central environment.
3.  Backup your online Business Central environment
4.  Create a multitenant docker container based on these artifacts
5.  Restore your online Business Central backup
6.  Make sure necessary apps are installed or removed
7.  Mount and sync the database as a new tenant
8.  Sync all apps
9.  Upgrade all apps
10.  Upgrade tenant
11.  Create the user
12.  Start the Web Client

## Issues?

If you encounter issues, it might be a good idea to consult issues on [https://github.com/microsoft/navcontainerhelper/issues](https://github.com/microsoft/navcontainerhelper/issues) to save time. Example: [https://github.com/microsoft/navcontainerhelper/issues/2537](https://github.com/microsoft/navcontainerhelper/issues/2537)

## Prerequisites

You need **Docker** installed and you need the **BcContainerHelper PowerShell module** – latest version. That goes for most my blog posts.

Like [this blog post](/2021/02/28/running-business-central-in-docker-using-sql-on-the-host/), in order for the scripts in this blog post to work, you need to install **SQL Server on the Docker Host with mixed mode authentication**. The scripts will connect using Windows Authentication and the container will connect using Username/Password authentication.

You need an **Azure Subscription** with credits or pay-as-you-go. We are only going to use this for a backup storage account, it doesn’t cost much, but it needs to be there.

You need to install the **Az PowerShell module** and you need to run **Connect-AzAccount** to connect your Windows Account with your Azure Subscription.

The DAC Framework needs to be installed in order to restore bacpacs. The DAC Framework can be downloaded [here](https://aka.ms/dacfx-msi).

You need to setup a storage account for the online backup either using **New-AzStorageAccount** or directly in the **Azure Portal**. This storage account will be the destination for the online backup.

You should setup a **Key Vault** for your secrets (like described [here](/2021/03/02/managing-secrets-in-your-scripts/)) and you should store the following secrets in the Key Vault:

-   **BcSaasRefreshToken** holds a refresh token for a delegated admin user, created using New-BcAuthContext as described [here](/2021/01/25/bcauthcontext/).
-   **storageAccountName** holds the name of the storage account in your subscription, which holds (or will hold) the online backup.
-   **password** holds the password you want to use for the container.
-   **licenseFile** holds a URL to the developer license file you will be using.

# Determine the artifacts to use

The first code section will read the Key Vault and set the 4 secrets as described above, then retrieve information about the environment (specified in the $environment variable) and retrieve the base app version number. Based on this, we find the right artifact url,

if (!($licenseFileSecret)) {
    Write-Host -ForegroundColor Yellow "Reading Key Vault"
    Get-AzKeyVaultSecret $vaultName  | % {
        Write-Host $\_.Name
        Set-Variable \`
            -Name "$($\_.Name)Secret" \`
            -Value (Get-AzKeyVaultSecret $vaultName -Name $\_.Name)
    }
}

Write-Host -ForegroundColor Yellow "SaaS Settings"
$environment = "Production"
$refreshToken = $BcSaasRefreshTokenSecret.SecretValue | Get-PlainText
$authContext = New-BcAuthContext -refreshToken $refreshToken
$bcEnvironment = Get-BcEnvironments \`
    -bcAuthContext $authContext | Where-Object {
        $\_.Name -eq $environment }
$baseApp = Get-BcPublishedApps \`
    -bcAuthContext $authContext \`
    -environment $environment | Where-Object { 
        $\_.Name -eq "Base Application" }
$artifactUrl = Get-BCArtifactUrl \`
    -country $bcEnvironment.countryCode \`
    -version $baseApp.Version \`
    -select Closest

After running this on my setup, $baseApp is

id        : 437dbf0e-84ff-417a-965d-ed2bb9650972
name      : Base Application
publisher : Microsoft
version   : 17.4.21491.22501
state     : installed

and $artifactUrl is

https://bcartifacts.azureedge.net/sandbox/17.4.21491.22501/us

We need the artifacts as the online tenant is just a tenant database. I will need to create a multitenant sandbox container with the right version to be able to mount the tenant database.

# Backup your online Business Central environment

Next code section will either reuse an existing database backup, or create a new backup. When the backup is complete, it will download the backup to the local temp folder as a .bacpac file. This code snippet uses the storageAccountName secret to locate the storage account in which the backup is or should be placed and create a shared access service token for this. The $bcAuthContext from the previous code snippet will be used for authentication to the Business Central environment.

$UseLatestBackup = $true
$stoAcc = Get-AzStorageAccount | Where-Object { 
    $\_.StorageAccountName -eq ($storageAccountNameSecret.SecretValue | Get-PlainText) }
$stoToken = $stoAcc | New-AzStorageAccountSASToken \`
    -Service Blob \`
    -ResourceType Container,Object \`
    -Permission "cdrw" \`
    -ExpiryTime (Get-Date).AddHours(25) \`
    -Protocol HttpsOnly
if ($UseLatestBackup) {
    $export = Get-BcDatabaseExportHistory \`
        -bcAuthContext $authContext \`
        -environment $environment | Sort-Object \`
            -Property blob | Select-Object -Last 1
    if ($export) {
        Write-Host -ForegroundColor Yellow "Use Latest Database Export"
        $blobCtName = $export.container
        $blobName = $export.blob
    }
    else {
        $UseLatestBackup = $false
    }
}
if (!$UseLatestBackup) {
    Write-Host -ForegroundColor Yellow "Create New Database Export"
    $uri = "$($stoAcc.PrimaryEndpoints.Blob)$stoToken"
    $blobCtName = "$($environment)backup".ToLowerInvariant()
    $blobName = "$(\[datetime\]::Now.ToString('yyyyMMddHHmm')).bacpac"
    New-BcDatabaseExport \`
        -bcAuthContext $authContext \`
        -environment $environment \`
        -storageAccountSasUri $uri \`
        -blobContainerName $blobCtName \`
        -blobName $blobName
}
Write-Host -ForegroundColor Yellow "Download Database Export"
$uri = "$($stoAcc.PrimaryEndpoints.Blob)$blobCtName/$blobName$stoToken"
$backupFile = Join-Path $env:TEMP $blobName
Download-File \`
    -sourceUrl $uri \`
    -destinationFile $backupFile

# Create a multitenant container from the artifacts

Much like described [here](/2021/02/28/running-business-central-in-docker-using-sql-on-the-host/), create a multitenant container called saas using the artifacts with the same version of your online tenant.

$containerName = "saas"
$auth = "UserPassword"
$credential = New-Object pscredential \`
    -ArgumentList 'admin', $passwordSecret.SecretValue
$licenseFile = $licenseFileSecret.SecretValue | Get-PlainText
$databaseParams = @{
    "databaseServer" = "host.containerhelper.internal"
    "databaseInstance" = ""
    "databasePrefix" = "$($containerName)-"
    "databaseName" = "CRONUS"
    "databaseCredential" = New-Object pscredential \`
        -ArgumentList 'sa', $passwordSecret.SecretValue
    "multitenant" = $true
}
New-BcContainer @databaseParams -replaceExternalDatabases \`
    -accept\_eula \`
    -containerName "$containerName" \`
    -credential $credential \`
    -auth $auth \`
    -artifactUrl $artifactUrl \`
    -enableTaskScheduler \`
    -licenseFile $licenseFile

# Restore the backup from your online environment

Using the DAC framework, we can restore the downloaded .bacpac file to a name which consists of the database prefix (from above) and the tenant name (environment name).

$tenantId = $environment
$dacdll = Get-Item "C:\\Program Files\\Microsoft SQL Server\\\*\\DAC\\bin\\Microsoft.SqlServer.Dac.dll"
if (!($dacdll)) {
    throw "DAC Framework is not installed"
}
Add-Type -path $dacdll.FullName
$databaseName = "$($databaseParams.databasePrefix)$tenantId"
$conn = @(
    "Data Source=localhost"
    "Initial Catalog=master"
    "Connection Timeout=0"
    "User Id=$($dbCredential.UserName)"
    "Password=$($dbCredential.Password|Get-PlainText)"
) -join ";"
Write-Host "Restoring Database from $backupFile as $databaseName"
$AppimportBac = New-Object Microsoft.SqlServer.Dac.DacServices $conn
$ApploadBac = \[Microsoft.SqlServer.Dac.BacPackage\]::Load($backupFile)
$AppimportBac.ImportBacpac($ApploadBac, $databaseName)

# Install 3rd Party Apps

Before mounting the tenant, we need to install the apps, which the tenant are depending on. This is done using an array of the apps we want to install:

$baseUrl = "https://businesscentralapps.blob.core.windows.net"
$installApps = @(
    "$baseUrl/bingmaps-appsource/latest/apps.zip"
    "$baseUrl/helloworld-preview/latest/apps.zip"
)
Publish-BCContainerApp \`
    -containerName $containerName \`
    -appFile $installApps \`
    -skipVerification \`
    -sync \`
    -install

# Remove Unknown Apps

Having installed the known apps, we need to analyze if all necessary apps are present and detect which apps we have to remove from the Installed Apps table.

$dockerApps = Get-BcContainerAppInfo \`
    -containerName $containerName
$tenantApps = Get-BcInstalledExtensions \`
    -bcAuthContext $authContext \`
    -environment $environment 
$allApps = $true
$removeApps = $tenantApps | ForEach-Object {
    $appId = $\_.id
    $appName = $\_.DisplayName
    $appPub = $\_.Publisher
    $appVer = \[Version\]::new(
        $\_.versionMajor,
        $\_.versionMinor,
        $\_.versionBuild,
        $\_.versionRevision)
    $app = $dockerApps | Where-Object { $\_.appid -eq $appid }
    if (!($app)) {
        Write-Host "- $appName from $appPub version $appVer is missing"
        $appName
        $allApps = $false
    }
    elseif ($app.Version -lt $appVer) {
        Write-Host "- $appName from $appPub is version $($app.Version)."
        Write-Host "  Version expected was $appVersion"
        $appName
        $allApps = $false
    }
}

and now remove the apps from the NAV App Published App and the NAV App Installed App tables:

if ($allApps) {
    Write-Host -ForegroundColor Green "All apps are present in container"
}
else {
    Invoke-ScriptInBcContainer \`
        -containerName $containerName \`
        -scriptBlock {
            Param(
                $databaseServer,
                $databaseInstance,
                $databaseCredential,
                $databaseName,
                $removeApps)
            $removeApps | ForEach-Object {
                Write-Host "Remove $\_"
                Invoke-SqlCmdWithRetry \`
                    -DatabaseServer $databaseServer \`
                    -DatabaseInstance $databaseInstance \`
                    -DatabaseName $databaseName \`
                    -databaseCredentials $databaseCredential \`
                    -maxattempts 1 -Query "
                DELETE FROM \[dbo\].\[NAV App Published App\]
                WHERE Name = '$\_'
                DELETE FROM \[dbo\].\[NAV App Installed App\]
                WHERE Name = '$\_'
                GO"
        }
    } -argumentList \`
        $databaseParams.databaseServer,
        $databaseParams.databaseInstance, 
        $databaseParams.databaseCredential,
        $databaseName,
        $removeApps
}

# Mount and sync the tenant database

The tenant database is now ready and we can mount and sync the database

Write-Host "Mount and sync tenant $tenantId"
Invoke-ScriptInBcContainer \`
    -containerName $containerName \`
    -scriptBlock { 
    Param(
        $databaseServer,
        $databaseInstance,
        $databaseCredential,
        $tenantId,
        $databaseName)
    Mount-NavTenant \`
        -ServerInstance $ServerInstance \`
        -id $tenantId \`
        -databaseserver $databaseServer \`
        -databaseinstance $databaseInstance \`
        -databasename $databaseName \`
        -databaseCredentials $databaseCredential \`
        -EnvironmentType Sandbox \`
        -OverwriteTenantIdInDatabase \`
        -Force
    Sync-NavTenant \`
        -ServerInstance $ServerInstance \`
        -Tenant $tenantId \`
        -Mode ForceSync \`
        -Force
} -argumentList \`
    $databaseServer,
    $databaseInstance,
    $databaseCredential,
    $tenantId,
    $databaseName

# Sync all apps

After mounting and sync’ing the tenant, the tenant will not be ready, you will need sync and upgrade apps, and also upgrade the tenant. First, sync all apps:

Get-NavContainerAppInfo \`
    -containername $containerName \`
    -tenant $tenantId \`
    -tenantSpecificProperties \`
    -sort DependenciesFirst | ForEach-Object {
        $syncState = $\_.SyncState.ToString()
        $appName = $\_.Name
        if ($SyncState -eq "NotSynced") {
            Sync-NavContainerApp \`
                -containerName $containerName \`
                -tenant $tenantId \`
                -appName $appName \`
                -Force
        }
}

# Upgrade all apps

Next up, run data upgrade for apps that needs it:

Get-NavContainerAppInfo \`
    -containername $containerName \`
    -tenant $tenantId \`
    -tenantSpecificProperties \`
    -sort DependenciesFirst | ForEach-Object {
        if ($\_.NeedsUpgrade) {
            Start-BcContainerAppDataUpgrade \`
                -containerName $containerName \`
                -tenant $tenantId \`
                -appName $\_.Name
        }
}

# Upgrade the tenant

Next up, upgrade the tenant

Invoke-ScriptInBcContainer \`
    -containerName $containerName \`
    -scriptblock { Param( $tenantId )
        Start-NAVDataUpgrade \`
            -ServerInstance $serverInstance \`
            -Tenant $tenantId \`
            -Force \`
            -FunctionExecutionMode Serial \`
            -SkipIfAlreadyUpgraded
        Get-NAVDataUpgrade \`
            -ServerInstance $serverInstance \`
            -Tenant $tenantId \`
            -Progress
} -argumentList $tenantId

# Create the user

Last, but still necessary, create a user for login

New-BcContainerBcUser \`
    -containerName $containerName \`
    -tenant $tenantId \`
    -Credential $credential \`
    -PermissionSetId 'SUPER' \`
    -ChangePasswordAtNextLogOn:$false

# Start the Web Client

Starting the Web Client can be done using:

Start-Process "http://$containerName/BC?tenant=$tenantId"

and we should have the Web Client with our production environment

![](/assets/images/2021/restoring-your-online-business-central-database-locally/image.png)

# Script

You can download the full script [here](https://bcdocker.blob.core.windows.net/public/saas2docker.ps1) and modify various variables.

# Issues

If you experience any issues, feel free to create an issue [here](https://github.com/microsoft/navcontainerhelper/issues), but please remember to include the full (and very lengthy) output from the script running.

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
