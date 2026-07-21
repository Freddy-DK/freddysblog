---
layout: post
title: "Preview of Dynamics 365 Business Central 2019 release wave 2"
date: 2019-07-31 10:54:43
categories: ["AL Development", "Docker", "NavContainerHelper"]
tags: ["Docker", "insider builds", "NavContainerHelper"]
permalink: /2019/07/31/preview-of-dynamics-365-business-central-2019-release-wave-2/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

As of today, we have started to create Docker images of Business Central 2019 Wave 2 preview on bcinsider. This blog post will describe how to get the image, what you can use it for and what you should be aware of.

# Get the image

The image tag for the preview images are:

**bcinsider.azurecr.io/bcsandbox-master:<country>-<platform>**

where <country> is the country code for the localization you need and <platform> is either **ltsc2016** (for Windows Server 2016 or Windows 10 pre-1809) or **ltsc2019** (for Windows Server 2019 or Windows 10 1809 or later).

The bcinsider repository requires authentication. The credentials are available through Microsoft Collaborate as part of the “Ready to Go” program. Read more about the “Ready to Go” program here: [https://docs.microsoft.com/en-us/dynamics365/business-central/dev-itpro/developer/readiness/readiness-ready-to-go?tabs=learning](https://docs.microsoft.com/en-us/dynamics365/business-central/dev-itpro/developer/readiness/readiness-ready-to-go?tabs=learning)

# Get the latest ContainerHelper

Please make sure you are using the latest version of the NavContainerHelper PowerShell module, here: [https://www.powershellgallery.com/packages/navcontainerhelper](https://www.powershellgallery.com/packages/navcontainerhelper) – you need at least version 0.6.2.9.

# Use the image for extension development

When using the image for extension development you basically start the container exactly like we have done since we shipped the first preview containers. Note that includeCSIDE and all functions working with C/AL objects are no longer supported and will return an error. Example:

$imageName = "bcinsider.azurecr.io/bcsandbox-master:w1"
$containerName = "test"
$auth = "UserPassword"
$credential = New-Object pscredential 'admin', (ConvertTo-SecureString -String 'P@ssword1' -AsPlainText -Force)
$licenseFile = "<licensefile>"

New-BCContainer -accept\_eula \`
                -imageName $imageName \`
                -containerName $containerName \`
                -auth $auth \`
                -credential $credential \`
                -licenseFile $licenseFile \`
                -updateHosts

The output after starting the container should be something like:

...
Creating SUPER user
Container IP Address: 172.25.15.205
Container Hostname : test
Container Dns Name : test
Web Client : http://test/BC/
Dev. Server : http://test
Dev. ServerInstance : BC

Files:
http://test:8080/AL-15.0.34329.0.vsix

Initialization took 121 seconds
Ready for connections!
Reading CustomSettings.config from test
Creating Desktop Shortcuts for test
**Container test successfully created**

Download the .vsix file (in this case from **[http://test:8080/AL-15.0.34329.0.vsix](http://test:8080/AL-15.0.34329.0.vsix)**) and install it in VS Code using **Install from VSIX** in the … menu.

**AL: Go!** will after selecting the directory for your project, ask you for a target platform:

![target platform](/assets/images/2019/preview-of-dynamics-365-business-central-2019-release-wave-2/target-platform.png)

4.0 is Business Central 2019 release wave 2 (sorry for the wrong caption).

Modify the server and serverinstance in launch.json:

"server": "http://<containername>",
"serverInstance": "BC",

Use the container name you used.

**Note:** The serverInstance in 15.x containers and later is BC.

**Download Symbols and press F5** and you will be running your first Wave 2 extension.

Please note the changes in app.json if you want to compile and publish your own extension to a preview container.

# Use the image for Code Customizations

The process for creating a container, which should be used for code customizations is very much like described here: [https://freddysblog.com/2019/04/15/c-al-to-al-code-customizations/](https://freddysblog.com/2019/04/15/c-al-to-al-code-customizations/)

$imageName = "bcinsider.azurecr.io/bcsandbox-master:base-ltsc2019"
$containerName = "test"
$auth = "UserPassword"
$credential = New-Object pscredential 'admin', (ConvertTo-SecureString -String 'P@ssword1' -AsPlainText -Force)
$licenseFile = "<licensefile>"

New-BCContainer -accept\_eula \`
                -imageName $imageName \`
                -containerName $containerName \`
                -auth $auth \`
                -credential $credential \`
                -licenseFile $licenseFile \`
                -updateHosts \`
                -includeAL \`
                -memoryLimit 10G

Note that the only extra parameters are **\-includeAL** and an increased memory limit (**\-memoryLimit 10G**). If you are using process isolation, you don’t need the memoryLimit flag.

After this you will need to create a folder with the source from the container, which you can work with in VS Code. This also follows the process from the former blog post. First extract the source:

$alProjectFolder = "C:\\ProgramData\\NavContainerHelper\\AL\\BaseApp"
Create-AlProjectFolderFromNavContainer -containerName $containerName \`
                                       -alProjectFolder $alProjectFolder \`
                                       -useBaseLine \`
                                       -addGIT \`
                                       -useBaseAppProperties

Then compile the app:

$app = Compile-AppInNavContainer -containerName $containerName \`
                                 -credential $credential \`
                                 -appProjectFolder $alProjectFolder \`
                                 -appOutputFolder $alProjectFolder

Then publish the new BaseApp to the container:

Publish-NewApplicationToNavContainer -containerName $containerName \`
                                     -appDotNetPackagesFolder (Join-Path $alProjectFolder ".netpackages") \`
                                     -appFile $app \`
                                     -credential $credential \`
                                     -useCleanDatabase

You only have to extract the source once of course – containers created on othrer computers do still need **\-includeAL**, but then you can add **\-doNotExportObjectsToText**  and **\-useCleanDatabase** to start a container, which is ready for publishing a new BaseApp.

# Organize your files

You might quickly discover that the source files after creating an al project with 15.x are significantly differently named and organized, compared to creating the same with 14.x.

This might of course cause issues when comparing sources etc. (as you probably can imagine). A seperate blog post will describe how you can specify a function to **Create-AlProjectFolderFromNavContainer** in which you can define how you want to structure your files, stay tuned.

# Please be aware…

Things are not cast in stone, things will change…

Below is a list of things to be aware of when using the image:

-   The Test Toolkit, the Test Libraries and the Tests are not included yet. We are currently refactoring tests into individual apps and will include those on the image as soon as we are ready.
-   The filename of the .vsix in the image is AL-<platformversionnumber>.vsix, uses the version number of the platform, which is NOT the version of the .vsix. In a future image, this will follow the version number of the .vsix.
-    finsql.exe can still be found on the image. This doesn’t mean that C/AL still exists. finsql.exe is only used for database creation and will be removed when this functionality is moved to a different tool.
-   You will still find a folder called RoleTailored Client in the image. This doesn’t mean that the Windows Client still is available. The folder contains some tools, that needs to be moved before removing the folder.

# What’s next

We will start shipping daily builds from master as of today. There might be days, where no builds are available and there can be multiple reasons for this. The image name above is always the latest available and use -alwaysPull if you need the latest at any time.

We will also ship preview DVD images (approx. biweekly) on collaborate, probably starting within the next 2 weeks.

If you need a specific image, you can add the version to the image tag, example:

bcinsider.azurecr.io/bcsandbox-master:15.0.34648.0-dk-ltsc2019

You will see blog posts describing what’s new and how to do things both on Freddys blog [https://freddysblog.com/category/al-development/](https://freddysblog.com/category/al-development/) and on the team blog [https://cloudblogs.microsoft.com/dynamics365/it/product/business-central/](https://cloudblogs.microsoft.com/dynamics365/it/product/business-central/).

You will also see various Business Central PMs to start sending updates and requests for feedback on Yammer ([https://www.yammer.com/dynamicsnavdev](https://www.yammer.com/dynamicsnavdev)). We love your feedback about all the new features and will provide details and platform to talk about those. We are also planning to use Yammer to share more info when some of the features are lighting up in the consecutive builds as not everything may be fully enabled yet.

For reference please also look at our **2019 Release wave 2 Plan**: [https://docs.microsoft.com/en-us/dynamics365-release-plan/2019wave2/dynamics365-business-central/](https://docs.microsoft.com/en-us/dynamics365-release-plan/2019wave2/dynamics365-business-central/)

# And…

Please remember to thank all the people in the development team working hard every day to make this possible – I only created the Docker images and some helper scripts on top of everything they did…

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
