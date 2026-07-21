---
layout: post
title: "NavContainerHelper – License"
date: 2018-03-20 03:37:36
categories: ["Docker", "NavContainerHelper"]
tags: ["Docker", "License", "NAV on Docker", "NavContainerHelper"]
permalink: /2018/03/20/navcontainerhelper-license/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

If you haven’t read the [first post](/2018/03/20/navcontainerhelper-1/) about the NavContainerHelper, you should do so.

By default the NAV containers are using the CRONUS demo database and the CRONUS Demo license file is already imported in that. If you want to use you own licensefile, you have a few options on how to do this.

## Specify a database which already contains a license file

If you are connecting to a foreign database server or you have specified a database backup which already contains a license file, then there is no need to specify a lciens

## Specify a secure Url to a license file

If you have a secure Url from which you can download your license file, you can specify this Url as a parameter

```
-licenseFile ""
```

The secure license file Url needs to start with http or https, in which case, the script will proceed to download the license file and import it into the NAV Database. Information on how to create a secure url can be found [here](/2017/02/26/create-a-secure-url-to-a-file/).

Example:

```
New-NavContainer -accept_eula `
                 -containerName "test" `
                 -auth NavUserPassword `
                 -imageName "microsoft/dynamics-nav" `
                 -Credential $credential `
                 -licensefile "https://www.dropbox.com/s/abcdefghijkl/mylicense.flf?dl=1"
```

**Note**, when you specify a licensefile to New-NavContainer, it will always import that license file, also if you are specifying a foreign database server or your own bak file.

## Specify the path of a license file

If the license file is accessible from the container through a path, you can specify this path as a parameter

```
-licenseFile ""
```

Example:

```
$additionalParameters = @("-v c:\temp:c:\temp")
New-NavContainer -accept_eula `
                 -containerName "test" `
                 -auth NavUserPassword `
                 -imageName "microsoft/dynamics-nav" `
                 -Credential $credential `
                 -additionalParameters $additionalParameters `
                 -licensefile "c:\temp\mylicense.flf"
```

in this sample, the host c:\\temp folder is shared to the container as c:\\temp and the mylicense file from this folder is used.

**Note**, when you specify a licensefile to New-NavContainer, it will always import that license file, also if you are specifying a foreign database server or your own bak file.

## Override the SetupLicense script

You can also choose to override the script, which imports the license file and take all in your own hands.

The SetupLicense script is executed as the first thing after the Service Tier has started and it is invoked always. This means that you need to determine whether or not you need to install a license file. If you add a file called SetupLicense.ps1 to -myScripts, then this script will be executed instead of the default script.

Please inspect the file c:\\run\\SetupLicense.ps1 in the container for the default behavior and see Appendix 1 – Scripts for information about how to override scripts.

Example:

```
$setupLicenseScript = @'
if ($restartingInstance) {
    # Nothing to do
} else {
    Import-NAVServerLicense -ServerInstance NAV -LicenseFile "c:\run\my\my.flf" -Database NavDatabase -WarningAction SilentlyContinue
}
'@

New-NavContainer -accept_eula `
                 -containerName "test" `
                 -auth NavUserPassword `
                 -imageName "microsoft/dynamics-nav" `
                 -myScripts @("https://www.dropbox.com/s/abcdefghijkl/my.flf?dl=1", @{"SetupLicense.ps1" = $setupLicenseScript})
```

This script will download the my.flf file and place it in the c:\\run\\my folder and place the script above in the SetupLicense.ps1 file. The same syntax can be used with a file on the host computer.

**Note**, if you specify a .zip file in myscripts, it will be extracted into the c:\\run\\my folder in the container.

## Use Import-NavContainerLicense on a running container

If you want to import a license to an already running container, you can use the Import-NavContainerLicense function from the navcontainerhelper.

Example: Import-NavContainerLicense -containerName test -licenseFile “[https://www.dropbox.com/s/abcdefghijkl/mylicense.flf?dl=1](https://www.dropbox.com/s/abcdefghijkl/mylicense.flf?dl=1)”

The license file parameter can be a file on the host or a secure url.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
