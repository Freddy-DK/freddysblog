---
layout: post
title: "BcContainerHelper"
date: 2020-08-11 06:38:37
categories: ["BcContainerHelper", "Docker", "NavContainerHelper", "PowerShell"]
tags: ["BcContainerHelper", "Docker", "NavContainerHelper", "PowerShell"]
permalink: /2020/08/11/bccontainerhelper/
---

Before you read anything, please understand that **NavContainerHelper is still available in the PowerShell gallery and it will still be available for the foreseeable future**. Existing pipelines using NavContainerHelper can continue running – no problem just yet.

Future innovations and changes to support future versions of Business Central might require you to switch to BcContainerHelper. BcContainerHelper is another PowerShell module available in the PowerShell Gallery here: [https://www.powershellgallery.com/packages/BcContainerHelper](https://www.powershellgallery.com/packages/BcContainerHelper).

BcContainerHelper is a replacement for NavContainerHelper. Although you can install both modules side by side, the function names will clash and you will only get yourself into problems. Both modules contain the same functions and BcContainerHelper can do the same things as NavContainerHelper.

# Why the change then?

Beside the obvious reason, that the product is called Business Central and not NAV, there are a few other reasons why I wanted to start a new module in the PowerShell Gallery.

# Stability

NavContainerHelper is widely used and pipelines all over the world rely on NavContainerHelper not to fail. I do have a lot of tests running with every release of the module, but things can go wrong and 3 weeks ago I shipped a version called 0.7.0.12 which would have broken every single Business Central pipeline based on NavContainerHelper if I hadn’t released a fix within 45 minutes. 80 pipelines/people managed to download the faulty version before a new version was available.

At that time I realized that I had to find a way to have a stable and an insider version of the containerhelper.

The insider (preview) should be used by people who work closely with docker and pipelines, people who want to have the latest and greatest and people who want to contribute to the functionality, performance and stability of the module.

The stable version should be used by everyone else. Ideally the insider could break occasionally and the stable would never.

In order to start submitting prereleases I needed a different versioning schema. As explained in [this blog post](https://devblogs.microsoft.com/powershell/prerelease-versioning-added-to-powershellget-and-powershell-gallery/), I needed to shift to [SemVer 1.0](https://semver.org/spec/v1.0.0.html).

# Directory change

A frequent feature request I have received is the ability to control where the module will place its files. NavContainerHelper is hardcoded to **c:\\programdata\\navcontainerhelper** and as you can imagine, changing this could cause some disruption. With BcContainerHelper the default location of the work directory becomes **c:\\programdata\\bccontainerhelper** but I have also added a way to modify the location of the caches and shared folders/files. Adding this to NavContainerHelper was a bit scary and I decided to do this with the move to BcContainerHelper.

# Artifacts

It was important to have the change to artifacts done before the rename. Primary reason for this is, that people can stay on the latest NavContainerHelper until they have to move due to functionality they need. Development and PRs to NavContainerHelper has stopped and only bug fixes to keep pipelines running will be approved.

NavContainerHelper will stay for people finding things on blogs and trying things out. In a future version I will add an information stating that people should move to BcContainerHelper in order to

[BcContainerHelper 1.0.0](https://www.powershellgallery.com/packages/BcContainerHelper/1.0.0) has been shipped with exactly the same functionality as [NavContainerHelper 0.7.0.21](https://www.powershellgallery.com/packages/navcontainerhelper/0.7.0.21). New features will ONLY be added to BcContainerHelper and 1.0.2 is shipped now with terminology changes to use Bc instead of Nav where it makes sense.

NavContainerHelper will from now only get bug fixes and not all bug fixes. Only bug fixes which are blocking for people who for some reason cannot move to BcContainerHelper yet.

While writing this blog post version 1.0.2 is the latest build and insider builds of BcContainerHelper 1.0.3 are being deployed with every successful build (CD) with a prerelease tag/build number and any bug fix will be available as soon as it has been checked in the containerhelper pipeline.

# Getting the latest BcContainerHelper

Like with NavContainerHelper, installing the latest version is easy

```
Install-Module BcContainerHelper -force
```

should give you the latest stable version of BcContainerHelper (1.0.0 while writing this blog post).

```
Install-Module BcContainerHelper -force -allowPrerelease
```

should give you the latest prerelease version (1.0.1-preview143 while starting to write this blog post)

If PowerShell tells you that -allowPrerelease isn’t a known parameter, you will need to update your version of PowerShellGet. You should be able to do that using:

```
Install-Module -Name PowerShellGet -Repository PSGallery -Force
```

# Differences

A few things have changed between NavContainerHelper and BcContainerHelper. The obvious ones are the folders and the configuration variable. As already stated, BcContainerHelper uses c:\\programdata\\bccontainerhelper and NavContainerHelper uses c:\\programdata\\navcontainerhelper.

**NOTE: I think it is possible to have both modules loaded at the same time using the allowclobber switch, but I strongly recommend that you remove all containers and uninstall navcontainerhelper before installing bccontainerhelper** (remember to restart PowerShell after uninstalling NavContainerHelper)

**BcContainerHelper** does however support that you place a file called **BcContainerHelper.config.json** in the above folder, where you can point out which folder to use for cache and shared folders.

The two lines in the config file looks like:

```
{
    "hostHelperFolder": "d:\\containerhelper",
    "containerHelperFolder": "c:\\programdata\\bccontainerhelper"
}
```

If you set both values to **c:\\\\programdata\\\\navcontainerhelper** the module will use the data from the NavContainerHelper, I don’t recomment this.

**NOTE: You should not make the switch from NavContainerHelper to BcContainerHelper while you have containers running.**

**NOTE: This does require the prerelease builds as 1.0.0 didn’t fully support this yet, and there might be issues with this (I found a few)**

The configuration variable is called **$bcContainerHelperConfig** instead of **$navContainerHelperConfig**. Through this variable you can get access to configuration and also set configurations for this session.

**NOTE: Changing the hostHelperFolder and containerHelperFolder at runtime through this variable is not supported.**

# Sandbox containers are multitenant by default

A bigger change with BcContainerHelper is that sandbox containers becomes multitenant by default. You can still specify -multitenant:$false if you want them to run single tenancy but since Business Central online is multitenant and sandbox containers main purpose is to emulate Business Central online, I decided that this change was necessary.

If you do not want this behavior, you can set $bcContainerHelperConfig.sandboxContainersAreMultitenantByDefault = $false.

# The navcontainerhelper github repository

The github repository for the ContainerHelper has not changed and is likely not to. The master branch is the branch for BcContainerHelper and a release branch for NavContainerHelper has been created.

## The master branch

[https://github.com/microsoft/navcontainerhelper](https://github.com/microsoft/navcontainerhelper) is the source for BcContainerHelper and all successful builds from this branch will result in a prerelease on PowerShell Gallery.

**The dev branch**

Is a branch I use for development purposes, for experimental features etc. Beside this there will be feature branches for various development areas.

## The NavContainerHelper branch

[https://github.com/microsoft/navcontainerhelper/tree/NavContainerHelper](https://github.com/microsoft/navcontainerhelper/tree/NavContainerHelper) is now the source branch for NavContainerHelper.

# More tests

I do have a backlog of tests to write to ensure stability of the BcContainerHelper module, those will be written over the next weeks/months.

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
