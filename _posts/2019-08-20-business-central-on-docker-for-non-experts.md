---
layout: post
title: "Business Central on Docker for non-experts…"
date: 2019-08-20 06:50:49
categories: ["Docker", "NavContainerHelper"]
tags: ["ARM", "Docker", "NAV on Docker", "NavContainerHelper", "new-navcontainer", "PowerShell"]
permalink: /2019/08/20/business-central-on-docker-for-non-experts/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

With the release of NAV and Business Central images on Docker, a lot of people who are not familiar with Docker and Containers will be using Business Central on Docker.

Using Business Central on Docker doesn’t necessarily mean that you have to install Docker on your laptop, you can also use Virtual Machines on Azure for running your Containers.

Spinning up an Azure VM with Business Central can be done using [http://aka.ms/getbc](http://aka.ms/getbc) and you will find detailed description of all properties in the ARM template here: [https://freddysblog.com/2019/07/26/the-arm-templates-for-dynamics-365-business-central-and-microsoft-dynamics-nav/](/2019/07/26/the-arm-templates-for-dynamics-365-business-central-and-microsoft-dynamics-nav/).

This blog post will take you through how to spin up containers on your local machine.

# Why Docker?

We did not create Business Central containers to make life harder for anybody. We did create Business Central containers to make life easier for partners and based on the majority of the feedback from partners, I think it is safe to assume that we have done something right.

Yes, there are people who do not want to use Docker because they have had a bad experience, but there are also people who want to do their accounting using an Excel Spreadsheet, that is their choice and I have no issue with that.

# Is it hard?

There might be a lot of reasons why people think it is hard:

-   You need to know PowerShell
-   There is no GUI for Docker
-   Docker cannot run on your computer
-   You don’t know what image to use
-   There are a million parameters on New-BCContainer (or New-NavContainer)

# You need to know PowerShell

You do not need to be a PowerShell expert in order to use Docker, but you do need to know some basic PowerShell. I wrote a blog post a few weeks ago, which contains all the needed PowerShell knowledge in order to use Business Central Containers. You can read the blog post here: [https://freddysblog.com/2019/08/04/powershell-for-non-experts/](/2019/08/04/powershell-for-non-experts/)

# There is no GUI for Docker

Well there are GUI tools for Docker (look [here](https://www.upnxtblog.com/index.php/2018/01/17/top-6-gui-tools-for-managing-docker-environments/)), but the only thing that really gives you is, that you don’t need to know the basic PowerShell syntax. It doesn’t really help us determining the right parameters and options to start the container.

This blog post will come to the conclusion that what you really need for any project is a small PowerShell script, which recreates exactly what is needed for that project. Using a GUI will just require you to remember everything or the GUI would have to be very tailored towards Business Central containers.

# Docker cannot run on your computer

Typical reasons for Docker not to be able to run is:

-   Lack of memory, a NAV / Business Central container for extension development needs ~4GB. For AL Code Customizations this number is probably the double.
-   No HyperV present. With most Windows Versions you can run process isolation with Business Central containers, but if you are running pre-release versions of windows, you might have problems.
-   An Anti-Virus software package, which isn’t compatible with Docker.

If you cannot change these things, you might have to either run Docker centrally or run an Azure VM, which you connect your development environment to.

# You don’t know what image to use

All public Business Central containers are hosted at

mcr.microsoft.com/businesscentral/<sandbox\|onprem>:<tag>

and all public Dynamics NAV containers are hosted at

mcr.microsoft.com/dynamicsnav:<tag>

More information about the tag can be found by reading this blog post: [https://freddysblog.com/2019/07/14/nav-and-business-central-docker-images-moved-to-microsoft-container-registry/](/2019/07/14/nav-and-business-central-docker-images-moved-to-microsoft-container-registry/)

# There are a million parameters on New-NAVContainer/New-BCContainer

Maybe not one million, but yes, there are a lot. You shouldn’t see them as parameters though – you should see them as options and almost all options have very good defaults. In fact, if you want to spin up a Business Central container you can write:

```
New-BCContainer -accept_eula -containerName 'test'
```

in PowerShell and you will get a container called “test”, running the latest public version of Business Central On Premises.

![start](/assets/images/2019/business-central-on-docker-for-non-experts/start.png)

It will use Windows Authentication and ask for your Windows Credential when starting.

In order to run that statement, you of course need to have Docker installed and you also need to have the latest NavContainerHelper module installed in PowerShell. This is done by running:

```
install-module navcontainerhelper -force
```

The containerhelper provides a number of functions and features to help you run NAV or Business Central containers. The containerhelper will automatically determine which container OS to use based on the host OS. It will also determine whether or not it can run process isolation or it needs hyperv isolation. It will also automatically specify memorylimit if hyperv isolation is selected etc. etc.

**If the simplest New-BCContainer command doesn’t work, you might be in the situation that Docker doesn’t run on your machine and you should troubleshoot that or find other ways to run Business Central Containers.**

For every extra parameter you add, you should have a reason for doing so.

**Do NOT add parameters you don’t know why you add.**

It is kind of like doing a demo install of NAV or Business Central, after the installation, you might need to change some configurations, but it will normally work without.

The below list of parameters/options isn’t an exhaustive list, but it is the options that I most frequently see people changing and there are some parameters you likely will have to add in order to use the container you just created.

## \-updatehosts

When creating the container in the prior section, you might not be able to connect to the container using the name **test**. Name resolution doesn’t work, but if you add a line with the IP number and the container name to the file **c:\\windows\\system32\\drivers\\etc\\hosts**, then you will be able to connect to the container using the Web Client URL above.

For this, we have created the parameter **\-updatehosts**. If you add this switch, then the containerhelper will automatically update the hosts file when you create, restart or remove containers and name resolution should work. It actually injects a script into the container and let the container do the update.

## \-accept\_outdated

Another parameter you might need to add is **\-accept\_outdated**. When using a container, which is based on an old version of Windows Server Core, you will have to use this flag. If you are running the latest onprem version of Business Central or the latest CU of NAV 2018, you should never need this flag – but if you select NAV 2018 CU5 or other images that was created months ago – you need the flag.

## \-imageName

A third parameter, that you always will add is **\-imageName** – which version of NAV or Business Central do you need. More information about the images available can be found here: [https://freddysblog.com/2019/07/14/nav-and-business-central-docker-images-moved-to-microsoft-container-registry/](/2019/07/14/nav-and-business-central-docker-images-moved-to-microsoft-container-registry/). With that, we have

```
New-BCContainer -accept_eula -accept_outdated -updatehosts `
                -imageName 'mcr.microsoft.com/businesscentral/onprem:dk' `
                -containerName 'test'
```

## \-dns

When running the container, you might see a warning, that DNS resolution is not working from within the container:

![dns](/assets/images/2019/business-central-on-docker-for-non-experts/dns.png)

This means that the container (for some reason) cannot use the DNS settings provided by Docker (typically because of local enforced policies on your machine – Azure VMs doesn’t need this). For this, you can modify the Docker daemon properties and add DNS settings to that:

![daemon](/assets/images/2019/business-central-on-docker-for-non-experts/daemon.png)

or you can add a parameter called **\-dns** followed by the DNS server you want the container to use. In some cases you can grab the IP number of the DNS used by the host, in other cases you can use a well known public DNS: **8.8.8.8**

```
New-BCContainer -accept_eula -accept_outdated -updatehosts -dns '8.8.8.8' `
                -imageName 'mcr.microsoft.com/businesscentral/onprem:dk' `
                -containerName 'test'
```

## \-auth

If you do not specify authentication, Windows Authentication will automatically be selected for you and if you don’t specify **\-credential** you will be prompted to enter your Windows Credentials.

Windows Authentication does require you to be connected to your work network and be able to ping/contact your AD server. Without this, it won’t work.

You can specify **\-auth UserPassword** to change authentication to Username/Password authentication.

## \-credential

If you do not specify credentials, you will be prompted to enter the credentials for your container. If you are using Windows authentication, you should enter the Windows Credentials of the host computer, else you should specify a username and a password of your choice.

If you want to specify credentials programmatically, it is done by creating a PSCredential object, which consists of a username and a password in a securestring:

```
$password = ConvertTo-SecureString -String "P@ssword1" -AsPlainText -Force
$credential = New-Object PSCredential 'admin', $password
New-BCContainer -accept_eula -accept_outdated -updatehosts -dns '8.8.8.8' `
                -imageName 'mcr.microsoft.com/businesscentral/onprem:dk' `
                -containerName 'test' `
                -auth UserPassword -Credential $credential
```

Note that you should not have passwords in clear text in source code, but if you are creating containers, which are only accessible from your machine as your user, then it doesn’t really matter what the password is for the container.

## \-licenseFile

If you do not specify a license file for the container, you will be using the CRONUS Demo license, which traditionally ships on the DVD. License file is specified either as a filename on the host computer or as a secure URL to a license file. Read [this blog post](/2017/02/26/create-a-secure-url-to-a-file/) to learn how to create a secure URL.

```
$password = ConvertTo-SecureString -String "P@ssword1" -AsPlainText -Force
$credential = New-Object PSCredential 'admin', $password
New-BCContainer -accept_eula -accept_outdated -updatehosts -dns '8.8.8.8' `
                -imageName 'mcr.microsoft.com/businesscentral/onprem:dk' `
                -containerName 'test' `
                -auth UserPassword -Credential $credential `
                -licenseFile 'c:\temp\license.flf'
```

## \-includeCSIDE

If you need to do CSIDE development, you might want to specify **\-includeCSIDE**. This causes the containerhelper to do a few things for you:

1.  Share a folder with the container and ask the container to copy the RoleTailored Client binaries to this shared folder. This makes finsql.exe and the Windows Client available on the host.
2.  Create Shortcuts on the desktop for C/SIDE, Windows Client and the Windows Client based Debugger (unless you have specified that you do not want the shortcuts)
3.  Export a baseline of all objects to text (unless **\-doNotExportObjectsToText** is specified) to C:\\ProgramData\\NavContainerHelper\\Extensions\\Original-<version>-<country>. This is only done the first time you run a specific version of NAV/BC.

In order to export the baseline of all objects, you need to specify a license file.

The baseline of all objects are used for other functions in the ContainerHelper for calculating deltas of changed objects like **Export-ModifiedObjectsAsDeltas** or **Convert-ModifiedObjectsToAl**.

_\-IncludeCSIDE is only supported in containers with NAV or Business Central 13.x or 14.x._

## \-includeAL

If you need to make customizations to the BaseApp using code customizations, you might want to specify **\-includeAL**. This causes the containerhelper to do a few things for you:

1.  Export a baseline of all objects in .al format (unless **\-doNotExportObjectsToText** os specified) to C:\\ProgramData\\NavContainerHelper\\Extensions\\Original-<version>-<country>-al. This is only done the first time you run a specific version of Business Central.
2.  Share a folder with all .net DLLs used by the BaseApp to the host in order for VS Code to find and use the DLL’s for compilation.

IncludeAL is also a pre-requisite for functions like **Create-AlProjectFolderFromNavContainer** which will create a folder with all the files needed to build and publish the BaseApp to the container.

_\-includeAL is only supported in containers with Business Central 14.x or later._

## \-doNotExportObjectsToText

You can specify **\-doNotExportObjectsToText**, in order to suppress the generation of a baseline of the BaseApp to be generated as .al or .txt files as explained in **\-includeAL** and **\-includeCSIDE**.

The generation of the baseline is a prerequisite for some of the other functions in the containerhelper and is only done once for every local version of NAV/Business Central.

## \-enableSymbolLoading

Specifying the **enableSymbolLoading** switch will put the container in hybrid development mode, which allows you to make changes in C/AL, regenerate symbols and use this new C/AL functionality in AL.

If you are working in C/AL or AL only, there is no reason to enable symbol loading.

If you are using hybrid development, please read this blog post to understand symbol loading better: [https://freddysblog.com/2019/03/16/symbols-demystified/](/2019/03/16/symbols-demystified/) – biggest takeaway is probably that your app does NOT need a reference to test symbols if using hybrid development, whereas it does if not using hybrid.

15.x containers obviously doesn’t support symbol loading (as C/AL is no longer present) and the reference to test symbols will change in 15.x to be a dependency to the test apps.

## \-multitenant

If you need to use multitenancy, add this switch and your container will automagically turn into a multitenant container.

This will take a few extra minutes, as the containerhelper will have to split the database in an app and a tenant database.

## \-doNotCheckHealth

Specifying the **\-doNotCheckHealth** switch causes the container to switch off health checking. By default, a PowerShell script is run inside the container every 30 seconds, checking whether the container is healthy. If you are running a lot of containers on your machine, this can save some time.

## \-alwaysPull

Specifying **\-alwaysPull** causes the containerhelper to check whether a new image is available online for the imagename provided.

# Other functions that might come in handy

**Import-TestToolkitToBCContainer** will import the test toolkit to your container. This can also be achieved by specifying **\-includeTestToolkit** to **New-BCContainer**, which essentially just calls this function.

**Import-ObjectsToNavContainer** will import C/AL objects in either .txt or .fob format to a C/AL container. Might be needed when creating a development container for C/AL or hybrid projects.

**Publish-BCContainerApp** will publish an app to a container. Might be needed when creating a development container for AL projects, which has a dependency on other extensions.

**Compile-AppInBcContainer** will use the container to compile an app, for which the source code is in a directory, which is shared with the container. This is typically used in CI/CD scenarios, but can also be used when setting up your development container.

**Run-TestsInBcContainer** will use the container to run a set of tests which have been imported into the test toolkit. This is typically used in CI/CD scenarios, but can also run locally to test your stuff before checking in.

**Convert-ModifiedObjectsToAl** will convert the C/AL modifications you have in a container to an AL extension. There is no magic here, if the C/AL code customizations doesn’t follow the rules of an extension, things will fail. Read more here: [https://freddysblog.com/2019/04/15/c-al-to-al-extension/](/2019/04/15/c-al-to-al-extension/)

**Publish-NewApplicationToBCContainer** will publish a new BaseApp to a container. Might be needed when creating a development container for doing code customizations of the BaseApp in AL. Read more here: [https://freddysblog.com/2019/07/31/preview-of-dynamics-365-business-central-2019-release-wave-2/](/2019/07/31/preview-of-dynamics-365-business-central-2019-release-wave-2/) or here: [https://freddysblog.com/2019/04/15/c-al-to-al-code-customizations/](/2019/04/15/c-al-to-al-code-customizations/)

**Check-NavContainerHelperPermissions** can help you assign the right permissions to your user if you are running as non-administrator. If you are running PowerShell with elevated privileges, this function won’t do anything.

**Invoke-ScriptInBCContainer** will invoke a PowerShell script inside a container. When running the script, you will have all NAV/Business Central PowerShell modules pre-loaded and ready for use and the instance name of the service tier is in the $serverInstance variable (NAV for 14.x and earlier containers and BC for 15.x and later)

```
Invoke-ScriptInBCContainer -containerName $containerName -scriptblock {
    Get-NAVServerUser $serverInstance
}
```

**Remove-BCContainer** will remove a container. Please note that creating a new container with the same name as another container will implicitly remove the old container as well.

**and many many more…**

# Save the script – NOT the container

It is of course hard to remember all parameters you need, but if you bookmark this blog post, then you can use that as a reference.

In order to prepare a container for a project, you will likely need a small script. Creating a container with the right options, importing objects or apps, maybe applying a configuration package and other things.

It is my recommendation that you do all of this in a PowerShell script, a script, which creates the container you need for the work on a specific project, make sure that your script can reproduce the container from scratch just by running the script and then:

**Save the script!!!**

**Do not try to keep the container running!!!**

**Don’t ever store anything inside the container you cannot recreate!!!**

**Place the script together with the source code for your project in a git repository!!!**

All of this allows you to recreate your development environment on any machine really and removes your dependency on single machines or installations. Even if it is hard at first.

**This puts you in a much better place after you have completed the task!!!**

# What if things doesn’t work?

If you have an issue. Something doesn’t work like expected, then go to the issues list here: [https://github.com/microsoft/navcontainerhelper/issues](https://github.com/microsoft/navcontainerhelper/issues).

Search to see whether other people had the same issue and if you cannot find a solution, open a new issue. Please provide as much information as you can about the issue. An issue like:

**_Web Client doesn’t work in latest Business Central container!_**

with no further explanation doesn’t give us much to work with, there is a very small chance that everybody is running into this and you are the first to discover it.

Typically issues falls into one of these buckets:

1.  Local issues with your computer. Here it is necessary to know which container you are using and what OS you are running on the host, but also what Docker version. All this information is provided as output to New-BCContainer.
2.  Invalid PowerShell or wrong parameter values like an invalid license file, script override or like. Inspecting the script and the output might reveal issues like this very fast.
3.  Advanced scenarios you cannot make work, like you want to create a container which connects to a foreign SQL Server or you want to change ports and other things. Understanding the scenario is key to know whether you are heading down the wrong path and assisting you in doing things the right way.

In all cases, providing a detailed error description, the full script and the output from the script will enable us to reproduce the error locally and assist you without having to ask for additional information or spend a lot of time guessing what you are trying to achieve.

As an example, [this issue](https://github.com/microsoft/navcontainerhelper/issues/464) was resolved within minutes because it explains clearly what he is trying to achieve and the error he gets.

Another example, [this issue](https://github.com/microsoft/navcontainerhelper/issues/520) provides the full script and output and although I could actually have seen the problem without having to ask additional questions.

Enjoy

**Freddy Kristiansen**  
_Technical Evangelist_
