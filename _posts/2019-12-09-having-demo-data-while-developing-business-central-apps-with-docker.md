---
layout: post
title: "Having Demo Data while developing Business Central Apps with Docker"
date: 2019-12-09 07:28:40
categories: ["AL Development", "Docker", "NavContainerHelper", "PowerShell"]
tags: ["Compile", "DEMO", "Development Environment", "Docker", "NavContainerHelper", "Publish", "Upgrade"]
permalink: /2019/12/09/having-demo-data-while-developing-business-central-apps-with-docker/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

I have always preached that you shouldn’t try to keep your Docker containers running. Containers should be something, which easily can be dismissed and recreated for any developer. One of the frequent questions is then: But what about my demo/development data?

There are several options for handling demo data for your app and I won’t claim that I know them all, nor do I think I am in a position to teach anybody about which method is the right. I can however describe how to enable the methods I know about when using Docker, so that people can implement this in whatever DevOps setup they have.

A common mechanism for partners is to have a script, which sets up their development containers. Typically this script will even publish some apps and the apps the developer needs to be working on will be published using the –**useDevEndpoint** parameter to make if possible for VS Code to attach a debugger or re-publish using **F5**.

# Demo Data App

Like when you create a test app to test your app, you could also create a demo data app to pump demo data into our app. Your test app would likely have a dependency to your app AND your demo data app in order to make sure that you have the right demo data to perform your tests.

You can use **Configuration Packages**, **XML Ports** or **AL Code** to create the demo data in your app, but you could also include tables with the demo data in your demo data app and transfer the data to the actual app, options are many.

You can use the **OnInstall** trigger in the demo data app to load the demo data into your app and you can uninstall and unpublish (with **doNotSaveData**) the demo data app when it has done its task.

If your demo data include secrets (like a key), you might want to trigger the actual demo data load using APIs, where you can transfer parameters like shown in [this blog post](/2019/03/27/navcontainerhelper-0-5-0-11/).

A script for creating a container, adding your app + demo data, prepared to use VS Code to do subsequent compilation and debugging of your app could look like this:

$imageName = "mcr.microsoft.com/businesscentral/onprem:ltsc2019"
$auth = "UserPassword"
$credential = New-Object pscredential 'admin', (ConvertTo-SecureString -String 'P@ssword1' -AsPlainText -Force)
$licenseFile = "C:\\temp\\license.flf"
$containerName = "test"
$appProjectFolder = "C:\\ProgramData\\NavContainerHelper\\BingMaps\\app"
$demoDataAppProjectFolder = "C:\\ProgramData\\NavContainerHelper\\BingMaps\\demodata"

# Create initial container
New-BCContainer \`
    -accept\_eula \`
    -imageName $imageName \`
    -containerName $containerName \`
    -licenseFile $licenseFile \`
    -auth $auth \`
    -Credential $credential \`
    -updateHosts

# Compile app
$appFile = Compile-AppInBCContainer \`
    -containerName $containerName \`
    -appProjectFolder $appProjectFolder \`
    -credential $credential \`
    -appOutputFolder $appProjectFolder

# Publish via dev endpoint - ready for development
Publish-BCContainerApp \`
    -containerName $containerName \`
    -appFile $appFile \`
    -useDevEndpoint \`
    -credential $credential

# Compile demo data app
$demoDataAppFile = Compile-AppInBCContainer \`
    -containerName $containerName \`
    -appProjectFolder $demoDataAppProjectFolder \`
    -credential $credential \`
    -appOutputFolder $demoDataAppProjectFolder

# Publish and install demo data app
Publish-BCContainerApp \`
    -containerName $containerName \`
    -appFile $appFile \`
    -skipVerification \`
    -sync
# Uninstall and unpublish demo data app
UnPublish-BCContainerApp \`
    -containerName $containerName \`
    -appName "BingMapsDemoData" \`
    -unInstall \`
    -doNotSaveData

# Configuration Packages

Configuration Packages (formerly known as RapidStart) is another good way of adding demo data and these can be applied using AL Code (in a demo data app) or using PowerShell as explained in [this blog post](/2019/09/19/using-apis-on-containers/).

# Database

If you have worked with NAV for a long time, this might be your default option. Lets have a database backup with our demo data and use that, and depending on what you are developing, this might still be a good option.

In fact, especially if you are working on a customer project, with multiple apps and you want to make sure the upgrade process always works, then your can use a database backup file with demo/development data, which has the same application versions as the current production database.

When creating the development environment, we will publish apps and perform data upgrade and ensure that the process always works. After deploying a new version of all apps to the customer, you grab a new database backup as your development backup file.

The following script assumes that you have a database backup in **C:\\ProgramData\\NavContainerHelper\\mybackup\\database.bak** and want to do development of the **BingMaps app** placed in this folder: **C:\\ProgramData\\NavContainerHelper\\BingMaps\\app**

$imageName = "mcr.microsoft.com/businesscentral/onprem:ltsc2019"
$auth = "UserPassword"
$credential = New-Object pscredential 'admin', (ConvertTo-SecureString -String 'P@ssword1' -AsPlainText -Force)
$licenseFile = "C:\\temp\\license.flf"
$containerName = "test"

# Create development container
New-BCContainer \`
    -accept\_eula \`
    -imageName $imageName \`
    -containerName $containerName \`
    -licenseFile $licenseFile \`
    -auth $auth \`
    -Credential $credential \`
    -updateHosts \`
    -bakFile "C:\\ProgramData\\NavContainerHelper\\mybackup\\database.bak"

# Unpublish old version of the app
UnPublish-BCContainerApp \`
    -containerName $containerName \`
    -appName "BingMaps" \`
    -unInstall

# Update version number for my development environment
$projectFolder = "C:\\ProgramData\\NavContainerHelper\\BingMaps\\app"
$appJsonFile = Join-Path $projectFolder "app.json"
$appJson = Get-Content -path $appJsonFile | ConvertFrom-Json
$appVersion = \[System.Version\]$appJson.version
$appJson.version = (\[System.Version\]::new($appVersion.Major, $appVersion.Minor, $appVersion.Build, $appVersion.Revision+1)).ToString()
$appJson | ConvertTo-Json -Depth 99 | Set-Content -Path $appJsonFile

# Compile
$appFile = Compile-AppInBCContainer \`
    -containerName $containerName \`
    -appProjectFolder $projectFolder \`
    -credential $credential \`
    -appOutputFolder $projectFolder

# Publish new version of the app via PowerShell
Publish-NavContainerApp \`
    -containerName $containerName \`
    -appFile $appFile \`
    -skipVerification \`
    -sync

# Perform data upgrade
Start-NavContainerAppDataUpgrade \`
    -containerName $containerName \`
    -appName "BingMaps"

# Uninstall and unpublish app
UnPublish-BCContainerApp \`
    -containerName $containerName \`
    -appName "BingMaps" \`
    -unInstall

# Compile / Create a new app package - cannot republish the same package
$appFile = Compile-AppInBCContainer \`
    -containerName $containerName \`
    -appProjectFolder $projectFolder \`
    -credential $credential \`
    -appOutputFolder $projectFolder

# Publish via dev endpoint - ready for development
Publish-BCContainerApp \`
    -containerName $containerName \`
    -appFile $appFile \`
    -useDevEndpoint \`
    -credential $credential

A few things to notice in this script is the **double publish** and the **double compile**…

## Double Publish

As you might have noticed, the script starts out by publishing the app to the container without using the **\-devEndPoint** parameter. Then it unpublishes and publishes again using the **\-devEndPoint** parameter.

As already described, we want to publish the app using -devEndPoint to enable VS Code to build, publish and debug subsequently. The problem however is, that data upgrade cannot be done when publishing through the dev endpoint. For this reason, I start out by publishing the new version using PowerShell, performing data upgrade to ensure data consistency and then unpublish and publish again.

## Double Compile

As you also might have noticed, the script will compile the app before publishing via PowerShell and again compile before publishing via devEndPoint. Why re-compile when nothing has changed you might think?

The simple explanation is, that you cannot publish the same app twice. If the package you try to publish have the same identity (the package contains a package id, which is a unique identifier being set with every compile), then Business Central will fail with an unprocessible entity error.

# Conclusion

Depending on your needs one solution might be better than the other. There might be other ways to solve the problem, which I haven’t thought about and I would be happy to receive ideas as comments to this blog post (might even extend the blog post with ideas I like:-)).

Personally, I like the Demo Data App idea because it separates the demo data from the actual app and allows for a simpler setup.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
