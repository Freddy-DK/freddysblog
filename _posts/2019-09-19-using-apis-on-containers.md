---
layout: post
title: "Using APIs on containers"
date: 2019-09-19 09:58:48
categories: ["Docker", "NavContainerHelper", "PowerShell"]
tags: ["APIs", "Configuration Packages", "RapidStart"]
permalink: /2019/09/19/using-apis-on-containers/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

A while ago, I added two functions to the ContainerHelper for using APIs on containers:

-   Get-BCContainerApiCompanyId
-   Invoke-BCContainerApi

In the end, these functions are just invoking a REST method on the container, so why create a function for this?

# SSL, HostName and Credentials

The primary reason for adding the functions is SSL, HostName and Credentials. When connecting to the API in a container, you need to connect to a URL. Typically, you can just connect to the right port and use the hostname as server, but in a lot of situations, this won’t work.

-   If the container isn’t started with -updateHosts, then you won’t (in a lot of cases) be able to use the hostname for connecting. In CI/CD scenarios, people frequently to not use -updateHosts and in those scenarios automation API’s might come in handy.
-   On Azure VMs created by aka.ms/getbc, the container is called navserver, but it is configured to use the DNS name of the Azure VM as Public OData URL.
-   If the container is setup for SSL with a self signed certificate, then I either need to disable SSL verification for this connection, or I need to install the self-signed certificate on the machine making the connection.

Also authentication to the container might be Windows or UserPassword and these functions will add authentication based on the settings in the container.

So basically – the functions calculate the right URL and disable SSL verification for this connection if necessary – that’s it – else they are “just” an invoke-RestMethod. In fact, the functions will display the URL it uses for the connection if you do not add -silent to the call.

# Configuration Packages

The reason why writing this post now is, that I needed to upload, import and apply a RapidStart package and I did discover a few issues of the two functions above.

So, in NavContainerHelper 0.6.4.2, these issues have been fixed and the script below shows how you can work with Configuration Packages on containers.

## Which Configuration Package to use?

In all containers, the standard configuration packages are included in C:\\ConfigurationPackages. For our script to work, we need the configuration package in a folder on the host. This script will locate and copy the file standard configuration package to the host:

$packageId = "W1.ENU.STANDARD"
$filename = Invoke-ScriptInBCContainer -containerName $containerName -scriptblock { Param($PackageId) 
    (Get-item "C:\\ConfigurationPackages\\\*$packageId\*").FullName
} -argumentList $packageId
$packageFilename = Join-Path "C:\\ProgramData\\NavContainerHelper\\Extensions\\$containerName" (\[System.IO.Path\]::GetFileName($filename))
Copy-FileFromBCContainer -containerName $containerName -containerPath $filename -localPath $packageFilename

## Which Company Id should be used?

When you connect to APIs, you typically need to specify a company Id, the first function above can be used to get the company id from the companies API. Internally the Get-BCContainerApiCompanyId calls Invoke-BCContainerAPI to get the CompanyId.

 $CompanyId = Get-NavContainerApiCompanyId -containerName $containerName -credential $credential -APIVersion "v1.0"

You can also specify a company name. If you do not specify a company name, the function will use the first company in the database.

## Deleting a configuration package (if it already exists)

I decided to NOT create separate functions for every API action. This would become a myriad of functions, and I would never be done.

Write-Host "Removing Configuration Package $packageId (if it exists)"
Invoke-BCContainerApi -containerName $containerName \`
                      -CompanyId $CompanyId \`
                      -credential $credential \`
                      -APIPublisher "microsoft" \`
                      -APIGroup "automation" \`
                      -APIVersion "v1.0" \`
                      -Method "DELETE" \`
                      -Query "configurationPackages('$packageId')" -ErrorAction SilentlyContinue | Out-Null

## Creating a configuration package

Before uploading the configuration package, you need to create the package.

Write-Host "Creating Configuration Package $packageId"
Invoke-BCContainerApi -containerName $containerName \`
                      -CompanyId $CompanyId \`
                      -credential $credential \`
                      -APIPublisher "microsoft" \`
                      -APIGroup "automation" \`
                      -APIVersion "v1.0" \`
                      -Method "POST" \`
                      -Query "configurationPackages" \`
                      -body @{ "code" = $PackageId; "packageName" = $PackageId; } | Out-Null

## Uploading a configuration package

Uploading a configuration package is done using the -infile parameter

Write-Host "Uploading Configuration Package $packageId"
Invoke-BCContainerApi -containerName $containerName \`
                      -CompanyId $CompanyId \`
                      -credential $credential \`
                      -APIPublisher "microsoft" \`
                      -APIGroup "automation" \`
                      -APIVersion "v1.0" \`
                      -Method "PATCH" \`
                      -Query "configurationPackages('$packageId')/file('$packageId')/content" \`
                      -headers @{ "If-Match" = "\*" } \`
                      -inFile $packageFilename | Out-Null

## Importing a configuration package

After uploading a configuration package, it needs to be imported. Importing does take some time, so after importing, I will wait for ImportStatus to be not Scheduled or InProgress.

**Note**, that in order for the import to start, you need to make sure the Task Scheduler is running in the container. In 0.6.4.2, there is a parameter on New-BCContainer called -EnableTaskScheduler, which will do just that.

Write-Host "Importing Configuration Package $packageId"
Invoke-BCContainerApi -containerName $containerName \`
                      -CompanyId $CompanyId \`
                      -credential $credential \`
                      -APIPublisher "microsoft" \`
                      -APIGroup "automation" \`
                      -APIVersion "v1.0" \`
                      -Method "POST" \`
                      -Query "configurationPackages('$packageId')/Microsoft.NAV.import" | Out-Null

$status = ""
do {
    Start-Sleep -Seconds 5
    $result = Invoke-BCContainerApi -silent -containerName $containerName \`
                                    -CompanyId $CompanyId \`
                                    -credential $credential \`
                                    -APIPublisher "microsoft" \`
                                    -APIGroup "automation" \`
                                    -APIVersion "v1.0" \`
                                    -Method "GET" \`
                                    -Query "configurationPackages('$packageId')"

    $newStatus = $result.importStatus
    $continue = ($newStatus -eq "Scheduled" -or $newStatus -eq "InProgress")
    if ($status -eq $newStatus) {
        Write-Host -NoNewline "."
    }
    else {
        $status = $newStatus
        Write-Host -NoNewline:$continue $status
    }
} while ($continue)

## Applying a configuration package

Much like importing, applying is all about invoking a bound action and waiting for a status.

Write-Host "Applying Configuration Package $packageId"
Invoke-BCContainerApi -containerName $containerName \`
                      -CompanyId $CompanyId \`
                      -credential $credential \`
                      -APIPublisher "microsoft" \`
                      -APIGroup "automation" \`
                      -APIVersion "v1.0" \`
                      -Method "POST" \`
                      -Query "configurationPackages('$packageId')/Microsoft.NAV.apply" | Out-Null

$status = ""
do {
    Start-Sleep -Seconds 5
    $result = Invoke-BCContainerApi -silent -containerName $containerName \`
                                    -CompanyId $CompanyId \`
                                    -credential $credential \`
                                    -APIPublisher "microsoft" \`
                                    -APIGroup "automation" \`
                                    -APIVersion "v1.0" \`
                                    -Method "GET" \`
                                    -Query "configurationPackages('$packageId')"

    $newStatus = $result.applyStatus
    $continue = ($newStatus -eq "Scheduled" -or $newStatus -eq "InProgress")
    if ($status -eq $newStatus) {
        Write-Host -NoNewline "."
    }
    else {
        $status = $newStatus
        Write-Host -NoNewline:$continue $status
    }
} while ($continue)

## The output

The output of running the above scripts should be something like and could very well be part of CI/CD pipelines to create a setup or a database with demo data

Invoke GET on http://172.21.118.48:7048/BC/api/v1.0/companies?$filter=name%20eq%20%27My%20Company%27&tenant=default
Removing Configuration Package W1.ENU.EXTENDED (if it exists)
Invoke DELETE on http://172.21.118.48:7048/BC/api/microsoft/automation/v1.0/companies(6ee50fc7-5ced-4dcc-899d-aaf41b287928)/configurationPackages('W1.ENU.EXTENDED')?tenant=default
Creating Configuration Package W1.ENU.EXTENDED
Invoke POST on http://172.21.118.48:7048/BC/api/microsoft/automation/v1.0/companies(6ee50fc7-5ced-4dcc-899d-aaf41b287928)/configurationPackages?tenant=default
Uploading Configuration Package W1.ENU.EXTENDED
Invoke PATCH on http://172.21.118.48:7048/BC/api/microsoft/automation/v1.0/companies(6ee50fc7-5ced-4dcc-899d-aaf41b287928)/configurationPackages('W1.ENU.EXTENDED')/file('W1.ENU.EXTENDED')/content?tenant=default
Importing Configuration Package W1.ENU.EXTENDED
Invoke POST on http://172.21.118.48:7048/BC/api/microsoft/automation/v1.0/companies(6ee50fc7-5ced-4dcc-899d-aaf41b287928)/configurationPackages('W1.ENU.EXTENDED')/Microsoft.NAV.import?tenant=default
InProgress...............Completed
Applying Configuration Package W1.ENU.EXTENDED
Invoke POST on http://172.21.118.48:7048/BC/api/microsoft/automation/v1.0/companies(6ee50fc7-5ced-4dcc-899d-aaf41b287928)/configurationPackages('W1.ENU.EXTENDED')/Microsoft.NAV.apply?tenant=default
InProgress.................................Completed

Of course you can remove the InProgress output or add -silent to other calls to avoid all the URLs if you like.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
