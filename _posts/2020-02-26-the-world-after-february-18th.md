---
layout: post
title: "The world after the February update"
date: 2020-02-26 17:27:04
categories: ["Docker", "NavContainerHelper"]
tags: ["Docker", "Hyperv isolation", "Process Isolation"]
permalink: /2020/02/26/the-world-after-february-18th/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

As you probably discovered, the February 2020 security update had a significant impact on NAV/Business Central Docker Containers. Especially if you were running Process isolation. Most visible problem was the fact that you couldn’t launch 32bit executables in containers after applying the security update. My blog post from February 14th ([https://freddysblog.com/2020/02/14/hyperv-isolation-to-the-rescue/](/2020/02/14/hyperv-isolation-to-the-rescue/)) would explain two ways to mitigate the problem:

1.  Run hyperv isolation
2.  Uninstall February update

None of these are perfect solutions. If you are using non-unicode apps (like finsql and C/AL code) then hyperv isolation really doesn’t work very well. It causes destructive character set conversions and you really don’t want to not apply security updates for a longer period in time.

I was looking forward to an official resolution of the problem and see where that leaves us in the NAV/BC Docker world.

# SAC and LTSC

Before continuing, I think it is important to understand the Windows Release channels.

Every 2-3 years, a new LTSC (Long-Term Servicing Channel) release is shipped. LTSC is a new major version of Windows Server, complete with GUI and all. Windows 10 get the same features as Windows Server. LTSC versions have 5 years of mainstream support.

Every half year, a new SAC (Semi-Annual Channel) release is shipped. Windows Server only ships a new ServerCore version here and NOT a full Windows Server with GUI. Windows 10 will get the SAC updates as well. SAC has 18 months of mainstream support.

Read more here: [https://docs.microsoft.com/en-us/windows-server/get-started-19/servicing-channels-19](https://docs.microsoft.com/en-us/windows-server/get-started-19/servicing-channels-19)

We decided early on to build NAV/BC containers for supported LTSC versions and all images are as such available for ltsc2016 (Windows Server 2016) and ltsc2019 (Windows Server 2019). Based on which host you are running you would fall back to the best LTSC version of the container image and download that.

# On February 18th…

February 18th, the official resolution for the issue was released and described here: [https://support.microsoft.com/en-ie/help/4542617/you-might-encounter-issues-when-using-windows-server-containers-with-t](https://support.microsoft.com/en-ie/help/4542617/you-might-encounter-issues-when-using-windows-server-containers-with-t). After reading this, I must admit that I was a bit disappointed.

The essence of this is, that Windows 10 with February update can only run Windows ServerCore images from February 18th or later in process isolation. It also states, that Windows ServerCore images from February 18th or later cannot run with Windows versions before February update.

This places us in a small problem. If I rebuild all ltsc2019 images with the ServerCore from February 18th, then no Windows without February security update will work. Should I have a ltsc2019a and ltsc2019b – but that would add a huge number of images to the already very high number of images out there.

The last thing, which we can deduct from the description is, that we always should have to run images with the same revision of Servercore as the host… – ouch – this poses a big problem, as this effectively means that I would have to rebuild all images 10000+ every month – that is definitely NOT the solution. Let’s look at some version numbers.

# The problem

The essence of the problem really is the following. Every month new Cumulative Updates are applied to Windows (the Host), but not to the 10000+ docker images

The matrix below shows some version numbers of Windows 10 and what month they are released in:  
![versions](/assets/images/2020/the-world-after-february-18th/versions.png)

Looking at a few images, then:

-   Dynamics NAV 2018 CU1 was built using 10.0.17763.316
-   Latest Business Central images was build using 10.0.17763.973

So we want to be able to run both these images on the same host using process isolation, knowing that container and host OS must match…!

# Are we stuck with Hyperv?

Hyperv is fine and it actually works pretty well when you are writing AL code and it has a lot of advantages over process Isolation but… it does also come with some challenges and I will describe those in a separate post. So, we definitely want to support Process isolation as well, so NO – we are NOT stuck with Hyperv.

# \-useBestContainerOS

The UseBestContainerOS flag in New-BCContainer was created to help people run containers in process isolation, when the Host OS didn’t match the Container OS, but originally it only looked at the build number (ltsc2019 vs. 1903 vs. 1909).

With the new information, I have extended this functionality to also look at the revision and allow to re-platform any image to a generic image, which matches the host.

NavContainerHelper 0.6.5.0 supports this and in order for this to work, we of course have to have generic images for all Windows 10 revisions.

# Generic images

Since yesterday, we now have generic images for most supported versions of the host. Try to run this command:

```
(get-navcontainerimagetags -imageName "mcr.microsoft.com/dynamicsnav").tags | Where-Object { $_.startswith('10.0.') -and $_.endswith('-generic') }
```

and you should see:

```
10.0.14393.3025-generic
10.0.14393.3085-generic
10.0.14393.3144-generic
10.0.14393.3204-generic
10.0.14393.3326-generic
10.0.14393.3384-generic
10.0.14393.3443-generic
10.0.14393.3506-generic
10.0.17763.504-generic
10.0.17763.557-generic
10.0.17763.615-generic
10.0.17763.678-generic
10.0.17763.737-generic
10.0.17763.864-generic
10.0.17763.914-generic
10.0.17763.973-generic
10.0.17763.1040-generic
10.0.18362.175-generic
10.0.18362.239-generic
10.0.18362.295-generic
10.0.18362.356-generic
10.0.18362.476-generic
10.0.18362.535-generic
10.0.18362.592-generic
10.0.18362.658-generic
10.0.18363.476-generic
10.0.18363.535-generic
10.0.18363.592-generic
10.0.18363.658-generic
```

# Get-BestGenericImageName

NavContainerHelper 0.6.5.0 also contains a function called Get-BestGenericImageName. This function will return the image name of the generic image, which best matches the Windows version you are running.

This function is used by New-NavContainer with -useBestContainerOS to find and use the best generic image to re-platform and run your container.

# Let’s try

On my work PC, I have added the February update and I was unable to run NAV/BC with process isolation. Now, with NavContainerHelper 0.6.5.0, I can run this script:

```
$credential = New-Object pscredential 'admin', (ConvertTo-SecureString -String 'P@ssword1' -AsPlainText -Force)
New-NavContainer `
    -accept_eula `
    -imageName mcr.microsoft.com/businesscentral/onprem:dk `
    -containerName test `
    -auth UserPassword `
    -Credential $credential `
    -updateHosts `
    -useBestContainerOS
```

and the output I get is this (I have split it up to explain what is happening).

**First part.** Some version numbers, pulling the best suitable image, removing the “old” container and displaying some more version numbers. Downloading the image of course only happens once.

```
NavContainerHelper is version 0.6.5.0
NavContainerHelper is running as administrator
Host is Microsoft Windows 10 Enterprise - 1909
Docker Client Version is 19.03.5
Docker Server Version is 19.03.5
Pulling image mcr.microsoft.com/businesscentral/onprem:dk-ltsc2019
dk-ltsc2019: Pulling from businesscentral/onprem
0837e82ff12e: Pull complete
250b6915cf74: Pull complete
f60208ba3443: Pull complete
f7350fa3fc3e: Pull complete
23d976f6e1a4: Pull complete
f6f250e28234: Pull complete
aef5c4ad7e51: Pull complete
ed3f76f16bab: Pull complete
f6ad0024ec3d: Pull complete
Digest: sha256:f9a8cde419180f2d1f8eb509b026cca43036eb809022ee9ee003ec3710a8a787
Status: Downloaded newer image for mcr.microsoft.com/businesscentral/onprem:dk-ltsc2019
Using image mcr.microsoft.com/businesscentral/onprem:dk-ltsc2019
Removing container test
Removing test from host hosts file
Removing C:\ProgramData\NavContainerHelper\Extensions\test
Creating Container test
Version: 15.3.40074.40822-dk
Style: onprem
Platform: 15.0.40073.40791
Generic Tag: 0.0.9.99
Container OS Version: 10.0.17763.973 (ltsc2019)
Host OS Version: 10.0.18363.657 (1909)
```

**Second part**, the ContainerHelper discovers that you can run the image with a better generic container image. In order to do this, the ContainerHelper will create a container with the requested image (not run, just create) and extract all the files needed to run the generic image:

```
A better Generic Container OS exists for your host (mcr.microsoft.com/dynamicsnav:10.0.18363.658-generic)
Creating temp container from mcr.microsoft.com/businesscentral/onprem:dk-ltsc2019 and extract necessary files
Extracting Service Tier and WebClient Files
Extracting Windows Client Files
Extracting Configuration packages
Extracting Test Assemblies
Extracting Test Toolkit
Extracting Upgrade Toolkit
Extracting Extensions
Extracting Applications
Extracting Applications.dk
Extracting Files from Run folder
Extracting Database Files
Downloading prerequisites
Downloading C:\ProgramData\NavContainerHelper\15.3.40074.40822-dk-Files\Prerequisite Components\IIS URL Rewrite Module\rewrite_2.0_rtw_x64.msi
Downloading C:\ProgramData\NavContainerHelper\15.3.40074.40822-dk-Files\Prerequisite Components\Open XML SDK 2.5 for Microsoft Office\OpenXMLSDKv25.msi
Downloading C:\ProgramData\NavContainerHelper\15.3.40074.40822-dk-Files\Prerequisite Components\DotNetCore\DotNetCore.1.0.4_1.1.1-WindowsHosting.exe
Performing cleanup
Removing temp container
```

**Third part**, launch the generic image with information about the files just extracted:

```
Using generic image mcr.microsoft.com/dynamicsnav:10.0.18363.658-generic
Generic Container OS Version: 10.0.18363.658 (1909)
Generic Tag of better generic: 0.0.9.99
WARNING: The container operating system matches the host operating system, but the revision is different.
If you encounter issues, you might want to specify -isolation hyperv
Using locale da-DK
Using process isolation
Disabling the standard eventlog dump to container log every 2 seconds (use -dumpEventLog to enable)
Files in C:\ProgramData\NavContainerHelper\Extensions\test\my:
- AdditionalOutput.ps1
- MainLoop.ps1
- SetupVariables.ps1
- updatehosts.ps1
Creating container test from image mcr.microsoft.com/dynamicsnav:10.0.18363.658-generic
3561f9614c4f77f520c2889f2e730f446eeb8dd4d31b98d0985c1508d819ec25
Waiting for container test to be ready
```

**Fourth part**, when running the generic image, it has to install all pre-requisite components and Business Central, using the extracted files.

```
Installing OpenXML
Installing DotNetCore
Starting Local SQL Server
Starting Internet Information Server
Copying Service Tier Files
Copying Web Client Files
Copying Client Files
Copying ModernDev Files
Copying PowerShell Scripts
Copying ConfigurationPackages
Copying Test Assemblies
Copying Applications
Copying ReportBuilder
Changing Database Server Collation to Danish_Greenlandic_100_CI_AS
SQL Server 2017 transmits information about your installation experience, as well as other usage and performance data, to Microsoft to help improve the product. To learn more about SQL Server 2017 data processing and privacy
controls, please see the Privacy Statement.
Copying Cronus database
Modifying Business Central Service Tier Config File for Docker
Creating Business Central Service Tier
Installing SIP crypto provider: 'C:\Windows\System32\NavSip.dll'
Starting Business Central Service Tier
Installation took 191 seconds
Installation complete
```

**Fifth part**, with the installation done, the remaining part is exactly the same as when running a normal Business Central container. The extra time spend for running the generic image is roughly 200 seconds + image download and file extraction, but those happens only once.

```
Initializing...
Setting host.docker.internal to 192.168.0.16 in container hosts file (copy from host hosts file)
Setting gateway.docker.internal to 192.168.0.16 in container hosts file (copy from host hosts file)
Setting kubernetes.docker.internal to 127.0.0.1 in container hosts file (copy from host hosts file)
Setting host.containerhelper.internal to 172.19.240.1 in container hosts file
Starting Container
Hostname is test
PublicDnsName is test
Using NavUserPassword Authentication
Creating Self Signed Certificate
Self Signed Certificate Thumbprint 93FA09DA838B2010B10D27992832B5D0C9F81027
Modifying Service Tier Config File with Instance Specific Settings
Restarting Service Tier
Registering event sources
Creating DotNetCore Web Server Instance
Creating http download site
Setting SA Password and enabling SA
Creating admin as SQL User and add to sysadmin
Creating SUPER user
Container IP Address: 172.19.243.112
Container Hostname : test
Container Dns Name : test
Web Client : http://test/BC/
Dev. Server : http://test
Dev. ServerInstance : BC
Setting test to 172.19.243.112 in host hosts file

Files:
http://test:8080/al-4.0.231354.vsix

Initialization took 53 seconds
Ready for connections!
Reading CustomSettings.config from test
Creating Desktop Shortcuts for test
Container test successfully created
```

And there you have it, your container is ready to be used using process isolation.

# Timing

On my machine, container generation using Hyperv isolation takes a total of 3 minutes and 30 seconds. Using useBestContainerOS takes 5 minutes and 11 seconds.

# NavContainerHelper 0.6.5.0

**NavContainerHelper 0.6.5.0 is available now!**

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
