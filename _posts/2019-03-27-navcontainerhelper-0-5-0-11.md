---
layout: post
title: "NavContainerHelper 0.5.0.11"
date: 2019-03-27 17:52:41
categories: ["AL Development", "Docker", "NavContainerHelper"]
tags: ["APIs", "Business Central", "Business Central Sandbox", "Docker", "NavContainerHelper", "PowerShell"]
permalink: /2019/03/27/navcontainerhelper-0-5-0-11/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

I don’t write blog posts about every single version of the NavContainerHelper, but I wanted to call out a few things in this version.

# .app instead of .fob

Both NavContainerHelper and the ARM templates ([http://aka.ms/getbc](http://aka.ms/getbc)) has been using .fob files for setting up Business Central. Importing a .fob file, which contains code to setup test users and invoking this through Invoke-NavContainerCodeunit.

As many of you have discovered, Business Central CU4 and CU5 (and spring release) all have a bug, where Invoke-NavCodeunit does not work if you are using NavUserPassword authentication. This basically had the effect that all partners setting up test or demo environments through the ARM templates, started to run into problems.

This basically left me with 3 options:

1.  Stating that these versions could not be used with Docker and/or ARM templates in conjunction with AAD authentication or Test Users.
2.  Creating an ugly workaround for these versions.
3.  Accelerate the transformation of these .fob files to extensions and create a solution which will work in the future when we go to AL only.

I decided to go for #3 – and that is included in this version of NavContainerHelper.

The function **Setup-NavContainerTestUsers** is now downloading, publishing and installing an app. This app will expose an Api, which is invoked from PowerShell and after doings its job, the app is uninstalled and unpublished, leaving no trace.

The source for this app is here: [https://dev.azure.com/businesscentralapps/CreateTestUsers](https://dev.azure.com/businesscentralapps/CreateTestUsers)

The other app is used for setting up the Azure Ad. This app is used in the ARM templates and the source can be found here: [https://dev.azure.com/businesscentralapps/AzureAdAppSetup](https://dev.azure.com/businesscentralapps/AzureAdAppSetup)

# Functions for invoking APIs

The new version has two new functions for communicating with APIs:

-   **Get-NavContainerApiCompanyId** will return the Company Id to be used when invoking APIs
-   **Invoke-NavContainerApi** will invoke an API.

The best way to describe the functions is to display how Setup-NavContainerTestUsers works in the new version:

$appfile = Join-Path $env:TEMP "CreateTestUsers.app"
Download-File -sourceUrl "http://aka.ms/Microsoft\_createtestusers\_13.0.0.0.app" -destinationFile $appfile
Publish-NavContainerApp -containerName $containerName -appFile $appFile -skipVerification -install -sync

$companyId = Get-NavContainerApiCompanyId -containerName $containerName -tenant $tenant -credential $credential

$parameters = @{ 
  "name" = "CreateTestUsers"
  "value" = (\[System.Runtime.InteropServices.Marshal\]::PtrToStringAuto(\[System.Runtime.InteropServices.Marshal\]::SecureStringToBSTR($Password)))
}
Invoke-NavContainerApi -containerName $containerName -tenant $tenant -credential $credential -APIPublisher "Microsoft" -APIGroup "Setup" -APIVersion "beta" -CompanyId $companyId -Method "POST" -Query "testUsers" -body $parameters | Out-Null

UnPublish-NavContainerApp -containerName $containerName -appName CreateTestUsers -unInstall

Yes, I could have used **Invoke-RestMethod** directly, but then I would have to handle authentication and other things in all locations.

**Invoke-NavContainerApi** actually uses the IP address of the container to do the communication, and thus does not rely on updatehosts or other things being in place. It also disables SSL verification temporarily if the container uses SSL.

# AL files

Another “new” feature is, that if you spin up your Business Central spring release container using -includeCSide and do not use -doNotExportObjectsToText, then you will now get three folders with source code:

-   ****Original-******<version>-<country>** which is the classic CSide representation of objects
-   ****Original-******<version>-<country>****\-newsyntax** which is the CSide representation of objects in new format.
-   **Original-<version>-<country>-al** which is all objects in the newsyntax folder converted to AL and basically your baseline for AL objects.

If you copy the content of the AL folder to another folder and initialize a git repository on those files, then you can import your objects, reexport everything as AL and copy them on top of the baseline.

This will make git disover the changes in your objects.

Try to run this:

$imageName = "bcinsider.azurecr.io/bconprem:w1-ltsc2019"
$myalfolder = "c:\\temp\\myal"
$containerName = "temp"
$credential = New-Object pscredential 'admin', (ConvertTo-SecureString -String 'P@ssword1' -AsPlainText -Force)

New-NavContainer -accept\_eula -containerName $containerName -imageName $imageName -includeCSide -licensefile C:\\temp\\license.flf -auth NavUserPassword -Credential $credential -updateHosts
$version = Get-NavContainerNavVersion -containerOrImageName $containerName
$baseAppFolder = "C:\\ProgramData\\NavContainerHelper\\Extensions\\Original-$version-al"
if (Test-Path $myalfolder) {
  remove-item -Path (Join-Path $myalfolder "\*") -Recurse -Force
}
else {
  New-Item $myalfolder -ItemType Directory
}
Set-Location $myalfolder
& git init
Copy-Item -Path (Join-Path $baseAppFolder "\*") -Destination $myalfolder
& git add .
& git commit --message "baseapp"

Now open $myalfolder in VSCode and you will see all your files:

![vscode2](/assets/images/2019/navcontainerhelper-0-5-0-11/vscode2.png)

You should also see that the source control branch indicates no changes.

Now you import your C/AL object modifications (or just modify a few objects) and then run this script (Remember that everything must be compiled in C/Side):

$ModifiedFolder = Convert-ModifiedObjectsToAl -containerName $containerName -sqlCredential $credential -doNotUseDeltas
Copy-Item -Path (Join-Path $ModifiedFolder "\*") -Destination $myalfolder

As you can see this script just re-exports everything as AL and copies it on top of your baseline. Git will compare and discover what you actually changed:

![vscode3](/assets/images/2019/navcontainerhelper-0-5-0-11/vscode3.png)

This is by all means not perfect, but it is a starting point in finding your AL changes and starting to look at them and seeing whether it is right…

Enjoy

_Freddy Kristiansen_  
Technical Evangelist
