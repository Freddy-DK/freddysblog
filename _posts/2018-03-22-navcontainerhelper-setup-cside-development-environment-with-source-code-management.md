---
layout: post
title: "NavContainerHelper – Setup CSIDE development environment with source code management"
date: 2018-03-22 14:17:50
categories: ["AL Development", "Docker"]
tags: ["AL", "CSIDE", "Development Environment", "NavContainerHelper", "VS Code"]
permalink: /2018/03/22/navcontainerhelper-setup-cside-development-environment-with-source-code-management/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

Most partners have different ways of setting up their CSIDE development environments and a number of partners are also using source code management to manage their source code. I have seen a few presentations on different ways of doing this and I will try to show how Docker and especially the NavContainerHelper can be used to setup a CSIDE development environment with source code management – very easily.

I will also cover how easy you can move your solution from one version of NAV to another and even how your C/AL solution can be moved to AL.

I know this blog post uses a very simple solution and my view on everything is fairly simplistic, but if your code is written using an event based architecture (called step 4 in [this blog post](https://freddysblog.com/2017/03/31/what-would-you-do/)), then it actually doesn’t have to be much harder than this…

# Install Docker and NavContainerHelper

In order for this to work, you need to setup a development machine with Docker and NavContainerHelper as described in [this blog post](https://freddysblog.com/2018/03/20/navcontainerhelper-1/).

**Note**, this uses [NavContainerHelper 0.2.7.3](https://www.powershellgallery.com/packages/navcontainerhelper/0.2.7.3).

# Fork my project

I have created a very simple project and placed it on GitHub. You will find it here: [https://github.com/NAVDEMO/MyFirstApp](https://github.com/NAVDEMO/MyFirstApp)

Go ahead and fork the project to your own GitHub account and clone your project to your development machine. I use the GitHub Desktop Client found here: [https://desktop.github.com/](https://desktop.github.com/) and after this, I have a folder with my project on my development machine like this:

[![](/assets/images/2018/navcontainerhelper-setup-cside-development-environment-with-source-code-management/bb5e8-myfirstapp2-1.png)](/assets/images/2018/navcontainerhelper-setup-cside-development-environment-with-source-code-management/bb5e8-myfirstapp2.png)

and if you look in the Source folder:

[![](/assets/images/2018/navcontainerhelper-setup-cside-development-environment-with-source-code-management/4f941-myfirstapp3-1.png)](/assets/images/2018/navcontainerhelper-setup-cside-development-environment-with-source-code-management/4f941-myfirstapp3.png)

Open PowerShell ISE as administrator and load the CreateDevEnv.ps1 file.

$mylicense = "c:\\temp\\mylicense.flf"
$imageName = "microsoft/dynamics-nav:2017-cu13"
$sourceFolder = Join-Path $PSScriptRoot "Source"
$containerName = Split-Path $PSScriptRoot -Leaf
New-NavContainer -accept\_eula \`
                 -containerName $containerName \`
                 -imageName $imageName \`
                 -auth Windows \`
                 -licensefile $mylicense \`
                 -updateHosts \`
                 -includeCSide \`
                 -additionalParameters @("--volume ${sourceFolder}:c:\\source") 
Import-DeltasToNavContainer -containerName $containerName -deltaFolder $sourceFolder -compile

This script assumes that you have a license file in c:\\temp – please modify the line if needed.

The script will create a NAV container called MyFirstApp, using Windows authentication, including CSIDE and sharing the source folder to the container. You should see an output like this:

...
Container IP Address: 172.19.157.232
Container Hostname : MyFirstApp
Container Dns Name : MyFirstApp
Web Client : http://MyFirstApp/NAV/WebClient/

Files:

Initialization took 38 seconds
Ready for connections!
Reading CustomSettings.config from MyFirstApp
Creating Desktop Shortcuts for MyFirstApp
Nav container MyFirstApp successfully created
Copy original objects to C:\\ProgramData\\NavContainerHelper\\Extensions\\MyFirstApp\\original for all objects that are modified (container path)
Merging Deltas from c:\\source (container path)
Importing Objects from C:\\ProgramData\\NavContainerHelper\\Extensions\\MyFirstApp\\mergedobjects.txt (container path)
Objects successfully imported
Compiling objects
Objects successfully compiled

# Start CSIDE and develop your solution

On your desktop you will find a shortcut to _MyFirstApp CSIDE_. Start this, and modify your solution. Try to add another field to the customer table: “My 2nd Field” and save the object. You can do multiple modifications to multiple objects and when you want to check in your modifications to GitHub, run the _GetChanges.ps1_ script, which looks like this:

$sourceFolder = Join-Path $PSScriptRoot "Source"
$containerName = Split-Path $PSScriptRoot -Leaf
Export-ModifiedObjectsAsDeltas -containerName $containerName -deltaFolder $sourceFolder

Now, switch to the GitHub Desktop app, which will show the modifications:

[![](/assets/images/2018/navcontainerhelper-setup-cside-development-environment-with-source-code-management/6bae9-github2-1.png)](/assets/images/2018/navcontainerhelper-setup-cside-development-environment-with-source-code-management/6bae9-github2.png)

and you can check these into the depot if needed.

After checkin, you might get changes from other developers. You might also have decided to discard some changes, meaning that your source folder is different from what you have in the development environment database.

Now simply re-run the _CreateDevEnv.ps1_ script to re-create your development environment based on the source folder. This only takes 1-2 minutes.

When you are done working on the project, simply remove the container, using the _RemoveDevEnv.ps1_ script, which looks like:

$containerName = Split-Path $PSScriptRoot -Leaf
Remove-NavContainer -containerName $containerName

**Note**, that you cannot re-create or remove the container if you have CSIDE or other files in the container open from the host.

# But…, I have .net add-ins!

If you have .net add-ins, that your solution depends on, you can place those in a folder and share this folder to the container as _c:\\run\\add-ins_, meaning that _CreateDevEnv.ps1_ now looks like:

$mylicense = "c:\\temp\\mylicense.flf"
$imageName = "microsoft/dynamics-nav:2017-cu13"
$sourceFolder = Join-Path $PSScriptRoot "Source"
$containerName = Split-Path $PSScriptRoot -Leaf
$addInsFolder = "C:\\temp\\addins"
New-NavContainer -accept\_eula \`
                 -containerName $containerName \`
                 -imageName $imageName \`
                 -auth Windows \`
                 -licensefile $mylicense \`
                 -updateHosts \`
                 -includeCSide \`
                 -additionalParameters @("--volume ${sourceFolder}:c:\\source",
                                         "--volume ${addInsFolder}:c:\\run\\Add-Ins")
Import-DeltasToNavContainer -containerName $containerName -deltaFolder $sourceFolder -compile

All files in the _c:\\run\\add-ins_ folder in the container will automatically be copied to the Add-ins folder in the Service folder and in the RoleTailored Client folder, for you to use when doing development.

# But…, I need to change some configuration settings!

If your solution depends on the Task Scheduler (which by default is not enabled in Docker images), then you normally would need to set the EnableTaskScheduler setting in CustomSettings.config and restart the service tier. This can also be done as part of running the container:

$mylicense = "c:\\temp\\mylicense.flf"
$imageName = "microsoft/dynamics-nav:2017-cu13"
$sourceFolder = Join-Path $PSScriptRoot "Source"
$containerName = Split-Path $PSScriptRoot -Leaf
$addInsFolder = "C:\\temp\\addins"
New-NavContainer -accept\_eula \`
                 -containerName $containerName \`
                 -imageName $imageName \`
                 -auth Windows \`
                 -licensefile $mylicense \`
                 -updateHosts \`
                 -includeCSide \`
                 -additionalParameters @("--volume ${sourceFolder}:c:\\source",
                                         "--volume ${addInsFolder}:c:\\run\\Add-Ins",
                                         "--env CustomNavSettings=EnableTaskScheduler=true")
Import-DeltasToNavContainer -containerName $containerName -deltaFolder $sourceFolder -compile

You will see during initialization of the container, that the settings are transferred:

Modifying NAV Service Tier Config File with Instance Specific Settings
Modifying NAV Service Tier Config File with settings from environment variable
Setting EnableTaskScheduler to true
Starting NAV Service Tier

and the Task Scheduler will be running.

# But…, I have other needs!

In general, the idea is that _CreateDevEnv.ps1_ should setup an environment that matches your solution again and again. The extensibility model of the NAV container allows you to dynamically override scripts, upload files, apply settings and much more.

If you are unable to setup a development environment like this for your solution, I would very much like to hear about it. Create an issue on the [issues list on navcontainerhelper](https://github.com/Microsoft/navcontainerhelper/issues) and I will see whether it is possible to fix this.

# What if I want to run my code in NAV 2018

I know this is a small solution and it is never as easy as it is here, but anyway.

Modify the imageName in CreateDevEnv.ps1 to

$imageName = "microsoft/dynamics-nav:2018"

and run the script.

It might take some time if you haven’t pulled the NAV 2018 image yet, but once the image is downloaded, the time should be the same as with NAV 2017.

Now run _GetChanges.ps1_ to see that a few other things was changed by moving the solution to NAV 2018.

# What if I want to move my solution to AL and VS Code

Now, you have got the hang of it, you are spinning up containers and living on the edge, but you want more… – you want to move your solution to AL.

In order to move the solution to AL, we need to import the changes to NAV 2018 or later and convert the modified objects to AL.

Modify the imageName in CreateDevEnv.ps1 to

$imageName = "microsoft/dynamics-nav:2018"

and run the script.

You should see some info about Dev. Server in the output, which you should note down.

Container Hostname : MyFirstApp
Container Dns Name : MyFirstApp
Web Client : http://MyFirstApp/NAV/
Dev. Server : http://MyFirstApp
Dev. ServerInstance : NAV

Files:
http://MyFirstApp:8080/al-0.12.17720.vsix

Initialization took 45 seconds

Also you should download the .vsix file to your host from the container and install this in Visual Studio Code.

After this, run this script

Convert-ModifiedObjectsToAl -containerName $containerName -startId 50100 -openFolder

In the same instance of ISE for the $containerName variable to be set.

and you should get a folder with your AL files.

[![](/assets/images/2018/navcontainerhelper-setup-cside-development-environment-with-source-code-management/a3403-toal-1.png)](/assets/images/2018/navcontainerhelper-setup-cside-development-environment-with-source-code-management/a3403-toal.png)

Now you can start VS Code, make sure the .vsix extension is the right version and press Ctrl+Shift+P and select AL Go!

Select local server and modify launch json to use the Dev. Server and Dev. Server Instance described in the container output.

Also set the authentication to Windows, copy the AL files to the folder and you are on your way to do Extensions v2 development…

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
