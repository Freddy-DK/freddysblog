---
layout: post
title: "BcContainerHelper configuration"
date: 2020-10-10 12:50:39
categories: ["BcContainerHelper", "Docker"]
tags: ["BcContainerHelper", "Docker", "PowerShell"]
permalink: /2020/10/10/bccontainerhelper-configuration/
---

The one thing most people have complained about in NavContainerHelper is that you cannot decide where it places it’s temporary files. With the shift to artifacts that got even worse because also artifacts was now placed on the c: drive.

With the shift to BcContainerHelper – this is now possible.

# Remove All Containers and shift

Before you rush out and uninstall NavContainerHelper and install BcContainerHelper – please remove all containers first:

Get-NavContainers | Remove-NavContainer

After this, you can run

Uninstall-Module NavContainerHelper -Force -AllVersions

and then remove the C:\\ProgramData\\NavContainerHelper folder.

Now you can install the BcContainerHelper using

Install-Module BcContainerHelper -Force

like also explained here: [https://freddysblog.com/2020/08/11/bccontainerhelper/](https://freddysblog.com/2020/08/11/bccontainerhelper/)

# The $bcContainerHelperConfig variable

First thing you can try after starting BcContainerHelper is to simply type:

$bcContainerHelperConfig

and it should output something like:

![](/assets/images/2020/bccontainerhelper-configuration/config.png)

These are all the settings, which you can use when configuring BcContainerHelper at this time and their default values. All values except for _ContainerHelperFolder_ and _HostHelperFolder_ er settable at runtime and will work in the PowerShell session in which they are set. At startup, the settings are read from a file:

C:\\ProgramData\\BcContainerHelper\\BcContainerHelper.config.json

You cannot change the location of this one file, but if you want to write your changed settings to this file, you can use this line:

$bcContainerHelperConfig | ConvertTo-Json | Set-Content "C:\\ProgramData\\BcContainerHelper\\BcContainerHelper.config.json"

meaning that this script:

$bcContainerHelperConfig.hostHelperFolder = "D:\\BcContainerHelper"
$bcContainerHelperConfig.bcartifactsCacheFolder = "D:\\Artifacts.cache"
$bcContainerHelperConfig | ConvertTo-Json | Set-Content "C:\\ProgramData\\BcContainerHelper\\BcContainerHelper.config.json"

would mean that next time you restart BcContainerHelper, you would place all temp. files on D:. You can of course also modify the .json file manually, but be aware. If the config file isn’t correctly formatted, BcContainerHelper won’t start.

# All the config settings

Below are a list of all the config settings and their meaning.

## bcartifactsCacheFolder

This is the location in which BcContainerHelper will place the artifacts cache. Download-Artifacts, New-BcContainer and New-BcImage will download artifacts to this location. New-BcImage will also use a temp folder in this location for building the image.

Default: _c:\\bcartifacts.cache_

## defaultContainerName

The default containerName is used for all functions in BcContainerHelper if the containerName parameter is not specified. This means that you can avoid using the ContainerName altogether if you always just reuse the same containerName as containers are removed when creating a new container.

Default: _bcserver_

## defaultNewContainerParameters

This is a hashtable of default parameters for New-BcContainer. If you f.ex. know that on this machine, you always want to use process isolation, you could specify:

$bcContainerHelperConfig.defaultNewContainerParameters = @{ "Isolation" = "Process" }

and write that to the .json file, meaning that all calls on this machine would get -isolation process added (unless otherwise specified).

Stuff like isolation, authentication and updateHosts is often machine specific and not necessarily related to the individual container. Memory is more related to what you use the container for and what version of the container you use. On my machine I have run this:

$bcContainerHelperConfig.defaultNewContainerParameters = @{
    "Accept\_Eula" = $true
    "Auth" = "UserPassword"
    "Credential" = @{
        "Username" = $credential.UserName
        "Password" = $credential.Password | ConvertFrom-SecureString
    }
    "UpdateHosts" = $true
    "Isolation" = "Process"
}
$bcContainerHelperConfig | ConvertTo-Json | Set-Content "C:\\ProgramData\\BcContainerHelper\\BcContainerHelper.config.json"

will cause this part to be added to my config file:

"defaultNewContainerParameters":  {
    "Credential":  {
        "Password":  "01000000d08c9ddf0115d1118c7a00c04fc297eb010000009ef6655e3dafde43a2d80063bf4c457e00000000020000000000106600000001000020000000455606faf894727669f14bdfb841b51e3391901a373b0f24261b8cd83ff6d1b9000000000e80000000020000200000009994142904a3b286453a55caa4404303ad3469394b8de9c996bce3fa10aa33ef20000000c17ec4ac1648a40dde1efe932dd1bf36fff3a79e15a04a468438ef57e76e5eb040000000b3c183d5845b36c456f710af0e2d15abdc443ac35d8c31b1b0debf5b31012a3d4cc7155efec90c357bb17bd031ec0c8a3cf6807b3ff393d19b10d0eed5a2ce36",
        "Username":  "admin"
    },
    "Accept\_Eula":  true,
    "Isolation":  "Process",
    "UpdateHosts":  true,
    "Auth":  "UserPassword"
}

As you can see the Password is encrypted and can only be un-encrypted on my machine and you would even be able to use Windows Authentication and store your Windows Password relatively secure here.

Note that there is a bug in BcContainerHelper 1.0.8 if you set defaultNewContainerParameters credential in the same session as you run New-BcContainer, that will fail until you get 1.0.9-preview.

## containerHelperFolder

This folder is the folder _INSIDE THE CONTAINER_, which is mapped to the hostHelperFolder on the host. This value does not have to be the same as hostHelperFolder.

Default is _C:\\ProgramData\\BcContainerHelper_

## hostHelperFolder

This folder is the folder _ON THE HOST_, which is mapped to the containerHelperFolder inside the container. This value does not have to be the same as containerHelperFolder.

Default is _C:\\ProgramData\\BcContainerHelper_

## genericImageName

When you run a container based on artifacts, the ContainerHelper will try to locate the generic image, which matches the current host OS best. The genericImageName setting identifies where to look for this image and the {0} part of this value is replaced with the host operating system number (ex. 10.0.19041.508)

Default value is _mcr.microsoft.com/businesscentral:{0}_

If you set the genericImageName to _mcr.microsoft.com/businesscentral:{0}-dev_ you will get preview releases of the next version of the generic image. Worst case, this will be the same as the released.

You can also override the _useGenericImage_ in the _defaultNewContainerParameters_ or manually set _useGenericImage_ when calling _New-BcContainer_ or _New-BcImage_ to set the generic image to a specific image.

## sandboxContainersAreMultitenantByDefault

This setting determines whether sandbox containers are multitenant by default.

Default is _true_.

## use7zipIfAvailable

This setting determines whether or not the ContainerHelper will look for 7zip to be installed and use that for unpacking the downloaded artifacts. 7zip is much faster than the built-in _Expand-Archive_ function.

Default is _true_.

## usePsSession

The normal behavior of Invoke-ScriptInBcContainer is to use PsSessions when running as administrator and docker exec when running as non-administrator. Setting usePsSession to false will force ContainerHelper to use docker exec always.

Default is _true_.

More setting will be added when needed and I will update this blog post with any changes.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
