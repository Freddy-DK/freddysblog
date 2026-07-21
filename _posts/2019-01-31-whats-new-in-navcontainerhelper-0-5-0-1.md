---
layout: post
title: "What’s new in NavContainerHelper 0.5.0.1"
date: 2019-01-31 14:05:54
categories: ["Docker", "Not Archived"]
tags: ["Docker", "ltsc2016", "ltsc2019", "NAV on Docker", "NavContainerHelper"]
permalink: /2019/01/31/whats-new-in-navcontainerhelper-0-5-0-1/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

NavContainerHelper 0.5.0.1 was released recently. Read this blog post to learn what’s new.

# Run As Administrator – or not!

NavContainerHelper now supports running as a standard user, but it does require you to add some permissions to this user:

-   Full Control to the folder C:\\ProgramData\\NavContainerHelper and all subfolders and files.
-   Modify permissions to the file C:\\Windows\\System32\\drivers\\etc (if you want to use the -updateHosts switch on New-NavContainer).
-   Full Control to the docker engine pipe as described in Tobias Fensters blog post [here](https://www.axians-infoma.com/techblog/allow-access-to-the-docker-engine-without-admin-rights-on-windows/).

After installing or updating the NavContainerHelper, you can run the command Check-NavContainerHelperPermissions

# Check-NavContainerHelperPermissions

During initialization of NavContainerHelper, it automatically runs Check-NavContainerHelperPermissions with the -silent parameter. This will check the permissions for the user and display any missing permissions. Note that if you are running as administrator, you do have the right permissions.

You can also run Check-NavContainerHelperPermissions yourself and the output could look like this:

[![](/assets/images/2019/whats-new-in-navcontainerhelper-0-5-0-1/check-permissions.png)](/assets/images/2019/whats-new-in-navcontainerhelper-0-5-0-1/check-permissions.png)

You can ignore the hosts file permission check by adding –**IgnoreHosts**.

Which also indicates that I can run

```
Check-NavContainerHelperPermissions -Fix
```

or

```
Check-NavContainerHelperPermissions -Fix -IgnoreHosts
```

to fix the permissions for me:

[![](/assets/images/2019/whats-new-in-navcontainerhelper-0-5-0-1/74110-fix-permissions-1.png)](/assets/images/2019/whats-new-in-navcontainerhelper-0-5-0-1/74110-fix-permissions.png)

Note that PowerShell might ask for permissions to run some scripts elevated in order to assign these permissions.

After that, permissions are OK:

[![](/assets/images/2019/whats-new-in-navcontainerhelper-0-5-0-1/a1aa0-permissions-ok-1.png)](/assets/images/2019/whats-new-in-navcontainerhelper-0-5-0-1/a1aa0-permissions-ok.png)

The script for checking and fixing permissions can be found here: [https://github.com/Microsoft/navcontainerhelper/blob/master/Check-NavContainerHelperPermissions.ps1](https://github.com/Microsoft/navcontainerhelper/blob/master/Check-NavContainerHelperPermissions.ps1)

# Manually assigning permissions

If for some reason the script doesn’t work on your machine, you can also assign the permissions manually.

## Full Control to the folder C:\\ProgramData\\NavContainerHelper and all subfolders and files.

Navigate to C:\\ProgramData and open properties for the NavContainerHelper folder and add permissions for the user, it should look like this:

[![](/assets/images/2019/whats-new-in-navcontainerhelper-0-5-0-1/c05ff-folderpermissions-1.png)](/assets/images/2019/whats-new-in-navcontainerhelper-0-5-0-1/c05ff-folderpermissions.png)

## Modify permissions to the file C:\\Windows\\System32\\drivers\\etc

Only if you are planning to use the -updateHosts flag on New-NavContainer (where the hosts file is automatically updated with the IP number of the container), you need this permission. It should look like this:

[![](/assets/images/2019/whats-new-in-navcontainerhelper-0-5-0-1/4072d-hostspermissions-1.png)](/assets/images/2019/whats-new-in-navcontainerhelper-0-5-0-1/4072d-hostspermissions.png)

## Full Control to the docker engine pipe

Tobias Fenster has written a blog post and created a small PowerShell module, which fixes this issue. Please read Tobias Fensters blog post [here](https://www.axians-infoma.com/techblog/allow-access-to-the-docker-engine-without-admin-rights-on-windows/).

# Invoke-ScriptInNavContainer

In the new NavContainerHelper you will find a function called Invoke-ScriptInNavContainer and it does exactly like the name says – it invokes a script in a NavContainer.

```
Invoke-ScriptInNavContainer -containerName test -scriptblock { Get-NavServerUser NAV }
```

If you are running as an administrator, NavContainerHelper will create a PsSession to the container and invoke the script in a session.

If you are running as an end user, NavContainerHelper will save the script in a shared folder and use docker exec to run the script in the container. The return value from the script will also be serialized into a file in the shared folder and deserialized in the script.

**Note:** You can transfer Secure strings to the script (they will be transferred using a random encryption key), but currently Secure strings returned from the script is not supported/cannot be unpacked. Let me know if this is needed and I will consider.

**The biggest difference is really that outputting this to Host** in your script will go to the Host if you are running as administrator and it will be returned as a return value if running as an end user.

Example:

```
$a = Invoke-ScriptInNavContainer -containerName test -scriptblock { Write-Host "This is a test" }
```

When run as administrator, **this will output the string “This is a test” to the Host and $a will be empty**. When you run this as an end user, **nothing will output to the Host and $a will contain “This is a test”**.

My recommendation is that you **avoid outputting to Host in the Scriptblock**.

_**All places within NavContainerHelper, where a script is invoked in a NavContainer has been replaced by a call to Invoke-ScriptInNavContainer.**_

# Using the best container image

After releasing ltsc2019 images and NavContainerHelper 0.4.3.2, I could see that a number of people still would spin up 2016 images, simply because they were already downloaded. The policy was, that if you reference an image (eg. microsoft/dynamics-nav) without specifying a tag and without specifying -alwayspull – and you had that image downloaded, then NavContainerHelper would reuse that.

This is still going to happen – but now you will see a note when starting the container, which indicates that you aren’t using the optimal image:

[![](/assets/images/2019/whats-new-in-navcontainerhelper-0-5-0-1/fe323-usebestsuggestion-1.png)](/assets/images/2019/whats-new-in-navcontainerhelper-0-5-0-1/fe323-usebestsuggestion.png)

adding -useBestContainerOS will force NavContainerHelper to use the optimal image and even re-platform the image if needed.

# New function Get-NavContainerImageTags

Try to run this script:

```
$imageName = "mcr.microsoft.com/businesscentral/onprem"
$tags = (Get-NavContainerImageTags -imageName $imageName).Tags | Where-Object { $_.contains("w1-ltsc2019") }
```

$tags will now contain all tags of ltsc2019 images with w1. While writing this blog post, that is:

```
13.0.24630.0-w1-ltsc2019
13.1.25940.0-w1-ltsc2019
13.2.26556.0-w1-ltsc2019
13.3.27233.0-w1-ltsc2019
cu1-w1-ltsc2019
cu2-w1-ltsc2019
cu3-w1-ltsc2019
latest-w1-ltsc2019
rtm-w1-ltsc2019
w1-ltsc2019
```

Grabbing all tags will right now return 572 image tags, which is 3 platforms (default (which is the same as ltsc2016), ltsc2016 and ltsc2019), 4 CU’s, 2 CU tags (version number and CU) – and all of that times 20 localizations. On top of that there are a number of latest tags.

# New function Get-NavContainerImageLabels

I have often found that I would like to know whether there is a new version of a specific image without pulling it.

Get-NavContainerImageLabels will download the labels section of the image manifests without pulling the image.

```
Get-NavContainerImageLabels -imageName "mcr.microsoft.com/businesscentral/onprem:cu4"
```

will return nothing until cu4 ships.

```
Get-NavContainerImageLabels -imageName "mcr.microsoft.com/businesscentral/onprem:cu3
```

will (today) return

```
country : W1
created : 201901171250
cu : 
eula : https://go.microsoft.com/fwlink/?linkid=861843
legal : http://go.microsoft.com/fwlink/?LinkId=837447
maintainer : Dynamics SMB
nav : 
osversion : 10.0.14393.2665
platform : 13.0.27183.0
tag : 0.0.9.0
version : 13.3.27233.0
```

Indicating that the cu field in the labels isn’t set, which is an error that will be fixed with the next rebuild round. When the images are being rebuild, the created date will also change.

Also if you want to know the labels of the latest Business Central On Prem image:

```
Get-NavContainerImageLabels -imageName "mcr.microsoft.com/businesscentral/onprem:latest-ltsc2019"
```

Finally, if you want to have all Business Central tags, sorted by image creation date:

```
$imageName = "mcr.microsoft.com/businesscentral/onprem"
$tags = (Get-NavContainerImageTags -imageName $imageName).Tags | Where-Object { $_.contains("w1-ltsc2019") }
($tags | % { Get-NavContainerImageLabels -imageName "${imageName}:$_" } | Sort-Object -Descending -Property created).version | Select -Unique
```

Which won’t return the images ordered by version number…

If you want to see all versions of NAV 2018, ordered by version number, you need to

```
$imageName = "microsoft/dynamics-nav"
$tags = (Get-NavContainerImageTags -imageName $imageName).Tags | Where-Object { $_.startswith("11.0") } | Where-Object { $_.contains("w1-ltsc2019") }
$labels = $tags | % { Get-NavContainerImageLabels -imageName "${imageName}:$_" }
($labels | Sort-Object -Descending -Property version).version
```

You should see:

```
11.0.26893.0
11.0.26401.0
11.0.25466.0
11.0.24742.0
11.0.24232.0
11.0.23572.0
11.0.23019.0
11.0.22292.0
11.0.21836.0
11.0.21441.0
11.0.20783.0
11.0.20348.0
11.0.19846.0
11.0.19394.0
```

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
