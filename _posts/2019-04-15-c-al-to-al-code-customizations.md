---
layout: post
title: "C/AL to AL code customizations"
date: 2019-04-15 11:46:07
categories: ["AL Development", "NavContainerHelper"]
tags: ["AL", "C/AL", "C/AL to AL"]
permalink: /2019/04/15/c-al-to-al-code-customizations/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

This blog post will take you through the steps needed to convert C/AL code customizations to AL code customizations.

I will show how to create a container and convert the C/AL baseapp to AL and publish the new AL baseapp to the container and thus running the baseapp as AL.

After this, the blog post will describe how you can add your own C/AL solution. When moving a solution from C/AL to AL, it is important that you first move the C/AL solution to the version of Business Central you want to utilize for the move.

**Note, this is ONLY intended for your preview to prepare your solution for the move to AL and to give Microsoft feedback on the conversion . Do NOT take customers live on code customized AL just yet.**

# Create a container with the solution you want to convert

I will use Business Central Spring release for the move and my solution was originally created in NAV 2017 CU3. The [first blog post in this series](/2019/04/15/c-al-to-al-preparations/) explains how you can move your solution from NAV 2017 CU3 to Business Central, but I am sure most partners have mechanisms and processes in place to perform this move. If you didn’t go through the process in the first blog post, you can create a container with the result of the first blog post by running this script:

\# Settings
$imageName = "mcr.microsoft.com/businesscentral/onprem:1904-rtm"
$auth = "NavUserPassword"
$credential = New-Object pscredential 'admin', (ConvertTo-SecureString -String 'P@ssword1' -AsPlainText -Force)
$licenseFile = "C:\\temp\\license.flf"
$demoSolution2Path = "C:\\ProgramData\\NavContainerHelper\\DemoSolution2.txt"

# Create Business Central container
New-NavContainer -accept\_eula \`
                 -imageName $imageName \`
                 -containerName "bc" \`
                 -licenseFile $licenseFile \`
                 -auth $auth \`
                 -Credential $Credential \`
                 -updateHosts \`
                 -includeCSide
#
# Import and compile objects
#
if (!(Test-Path $demoSolution2Path)) {
    Download-File -sourceUrl "https://bcdocker.blob.core.windows.net/public/DemoSolution2.txt" -destinationFile $demoSolution2Path
}
Import-ObjectsToNavContainer -containerName "bc" -objectsFile $demoSolution2Path
Compile-ObjectsInNavContainer -containerName "bc" -filter "Modified=Yes"

# Create a development container

Next up, I need to create a development container for my AL code customized solution. I will not be doing any changes in C/AL, meaning that I do not need to use **\-includeCSIDE** and I don’t need **\-enableSymbolLoading**. I will however use **\-includeAL** which will create a baseline of AL objects and also create a shared folder which I can use as assembly reference path from VS Code.

\# Settings
$imageName = "mcr.microsoft.com/businesscentral/onprem:1904-rtm"
$auth = "NavUserPassword"
$credential = New-Object pscredential 'admin', (ConvertTo-SecureString -String 'P@ssword1' -AsPlainText -Force)
$licenseFile = "C:\\temp\\license.flf"

# Create Business Central container
New-NavContainer -accept\_eula \`
                 -imageName $imageName \`
                 -containerName "myal" \`
                 -licenseFile $licenseFile \`
                 -auth $auth \`
                 -Credential $Credential \`
                 -updateHosts \`
                 -includeAL

Looking at the shared containerfolder for the myal container, you will find a folder called .netpackages:![dotnetpackages](/assets/images/2019/c-al-to-al-code-customizations/dotnetpackages.png)

The content of this folder is a copy of all the dotnet DLLs needed from VS Code in order to compile the base app. Furthermore, you will find a folder with the baseline of AL objects in a folder called Original-<version>-<country>-al (example: C:\\ProgramData\\NavContainerHelper\\Extensions\\Original-14.0.29537.0-W1-al)

# Create the AL project

First thing I want to do is to create an AL project containing all the baseapp objects, without my own customizations. NavContainerHelper contains a function called **Create-AlProjectFolderFromNavContainer**, which will create a project folder with all base application objects, setup app.json with reference to platform only, launch.json with reference to your development container and a settings.json with assemblyProbingPaths to the shared folder from this container.

Create-AlProjectFolderFromNavContainer -containerName "myal" -alProjectFolder "C:\\ProgramData\\NavContainerHelper\\AL\\DemoSolution" -useBaseLine -addGIT

The parameter **\-useBaseline** means that the function will copy the base app objects from the pre-exported baseline folder and **\-addGIT** means that the function will create a offline git repository on the folder and commit all objects.

Note that in order to use -addGIT, you need to have GIT installed (see [https://www.git-scm.com](https://www.git-scm.com)).

The reason for utilizing GIT here will be obvious a bit later.

**Note that the alProjectFolder should be in a location, which is shared with the container, a folder underneath C:\\ProgramData\\NavContainerHelper will work.**

Open the folder in VS Code, download symbols and build the project (**do NOT publish**):![demosolutionvscode3](/assets/images/2019/c-al-to-al-code-customizations/demosolutionvscode3.png)

Unfortunately, there is no way to exclude the deprecation warnings.

Note that VS Code might still be very slow when working with projects of this size, we are working on making this better.

You can also compile the project using the **Compile-AppInNavContainer** function:

Compile-AppInNavContainer -containerName "myal" -credential $credential -appProjectFolder "C:\\ProgramData\\NavContainerHelper\\AL\\DemoSolution"

It should give an output ending with:

C:\\ProgramData\\NavContainerHelper\\AL\\DemoSolution\\output\\Default Publisher\_myal\_1.0.0.0.app successfully created in 131 seconds
C:\\ProgramData\\NavContainerHelper\\AL\\DemoSolution\\output\\Default Publisher\_myal\_1.0.0.0.app

# Commit the changes done by the compiler

After compiling the project, you will see that GIT reveals 500+ modifications.

The reason for this is, that when you compile the project the first time after the conversion, the compiler will do some final conversion work.

Add the modifications and commit them in order to finalize the conversion.

Unfortunately this also means that any reruns of the conversion will show these changes again.

# Replace the C/AL objects in the development container with the new AL app

The **\-includeAL** flag used when creating this container doesn’t cause the container to be running the AL app – it just means that the container is prepared for AL code customizations. It is not needed for extension development. For extension development we actually cannot really see the difference in whether the container is running C/AL internally or AL.

But, since we want to do code customizations, we need to replace the C/AL objects in the database with the newly compiled AL app.

NavContainerHelper contains a function called **Publish-NewApplicationToNavContainer**, which uninstalls all apps, removes all C/AL objects and use the development endpoint of the container to publish the new app.

PS C:\\WINDOWS\\system32> Publish-NewApplicationToNavContainer -containerName "myal" -appDotNetPackagesFolder "C:\\ProgramData\\NavContainerHelper\\AL\\DemoSolution\\.netpackages" -appFile "C:\\ProgramData\\NavContainerHelper\\AL\\DemoSolution\\output\\Default Publisher\_myal\_1.0.0.0.app" -credential $credential -useCleanDatabase
Uninstalling apps
Removing C/AL Application Objects
Publishing Default Publisher\_myal\_1.0.0.0.app to http://172.19.186.163:7049/NAV/dev/apps?SchemaUpdateMode=synchronize
New Application successfully published to myal

Note that **\-useCleandatabase** is that flag signalling to remove C/AL objects and uninstall apps. You can use the same function (without the -useCleandatabase) or VS Code to do subsequent deployments of the app.

# Adding the Demo Solution

It’s all fine that I can create a Container where the base app if running AL code, but it doesn’t really do anything for me unless I can add my own solution.

Like with AL extension development, you can call Convert-ModifiedObjectsToAl in order to convert your objects to AL. In this scenario, you want to convert the full database. The reason for this is, that translation files (.xlf) will be re-written by the conversion tool and as such be overridden with partly translation files if you only convert part of the solution.

For converting and copying to my AL folder, I run this command:

Convert-ModifiedObjectsToAl -containerName "bc" -sqlCredential $credential -startId 50100 -doNotUseDeltas -alProjectFolder "C:\\ProgramData\\NavContainerHelper\\AL\\DemoSolution" -alFilePattern "\*.al,\*.xlf"

Specifying \*.al,\*.xlf in the file pattern means that we will not copy the reconverted reports and get all the 500+ changes when compiling the reports again. It is my recommendation to convert reports and the remaining app in  two rounds to avoid this.

And now, it is obvious why I have added GIT to the folder. I will have a very nice view of my solutions, with the modified files marked with M (modified) and new files marked with U (untracked). I can even see exactly what I have changed in my solution compared to the base app:![demosolutionvscode4](/assets/images/2019/c-al-to-al-code-customizations/demosolutionvscode4.png)

# Post Conversion modifications

As we saw during preparations, the demo solution included a menusuite item, which isn’t automatically converted. I will have to open the Directions Sessions List and add **ApplicationArea = All** and **UsageCategory = Lists.**

# Compile and publish the AL app

A very very nice feature in the spring release of the AL extension is Alt+F5 and Ctrl+Alt+F5:![rapid](/assets/images/2019/c-al-to-al-code-customizations/rapid.png)

These will compile and deploy only the changes made to the extension and it is really fast. Pressing Ctrl+Alt+F5 and it takes around 5-10 seconds on my laptop until it opens the Web Client and reveals my code modifications as part of the app:![directionslist](/assets/images/2019/c-al-to-al-code-customizations/directionslist.png)

# Conclusion

Code customized AL should really only be used if you cannot create an AL extension. I would prefer extensions over code customized AL any time. [This blog post](/2019/04/15/c-al-to-al-extension/) describes how to convert your C/AL solution to an AL extension.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
