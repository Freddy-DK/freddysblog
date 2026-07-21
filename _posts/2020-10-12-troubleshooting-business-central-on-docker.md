---
layout: post
title: "Troubleshooting Business Central on Docker"
date: 2020-10-12 10:37:48
categories: ["BcContainerHelper", "Docker"]
tags: ["Azure VM", "BcContainerHelper", "Docker", "Troubleshooting", "TSG"]
permalink: /2020/10/12/troubleshooting-business-central-on-docker/
---

This blog post is not really a troubleshooting guide (although there is a small TSG at the end). It is more like a brain dump of what I have seen over time and how I would approach a trouble shooting session. I have divided it into 6 sections:

1.  Dockers worst enemy (just FYI)
2.  Installing Docker
3.  Installing BcContainerHelper
4.  Creating a Business Central container
5.  Keep the script, not the Container
6.  Frequently seen docker errors

Going forward, I will add a link to this blog post to all issues on [https://www.github.com/microsoft/navcontainerhelper/issues](https://www.github.com/microsoft/navcontainerhelper/issues), where the answer to the issue is in this blog post.

# Dockers worst enemy

I often gets asked which OS is best for running Docker.

I do think Windows Server is more stable but the real enemy of Docker is not Windows 10 or Docker Desktop, it is (IMO) 3rd party software, antivirus, VPN, network or security policies enforced on computers from IT departments, who doesn’t know anything about Docker. I cannot count the number of hours I (and partners) have spend fighting these things.

[http://aka.ms/getbc](http://aka.ms/getbc) will create an Azure VM with Windows Server 2019 – and it just always works. This is an up-to-date Windows Server with Docker and no extra software or policies enforced, only Windows Defender.

## Installing Docker

If you are running Windows 10, you should run [Docker Desktop](https://docs.docker.com/docker-for-windows/install/). If you are running Windows Server, you should run [Docker Enterprise](https://hub.docker.com/editions/enterprise/docker-ee-server-windows).

Docker Desktop needs Hyper-V and the Containers Windows feature enabled, if you are running Windows 10 in a Virtual Machine, you need to enable nested virtualization (VM in VM). Note that not all Azure VM sizes supports nested virtualization, while writing this, the D\_v3, Ds\_v3, E\_v3 and Es\_v3 supports nested virtualization.

Docker Desktop defaults to running Linux Containers. You cannot run Business Central containers while Docker Desktop is running Linux Containers. You need to right-click the whale symbol and select **Switch to Windows Containers**.

Docker Enterprise doesn’t require Hyper-V and can run containers in process isolation mode without. Docker Enterprise of course supports hyperv isolation if Hyper-V is present.

If you want to check that Docker is installed and accessible, open PowerShell as administrator and run the command **Docker Version** (you might need to restart PowerShell after the installation). This should display the Client and Server version of Docker, like this:

```
PS C:\WINDOWS\system32> docker version
Client: Docker Engine - Community
 Version: 19.03.12
 API version: 1.40
 Go version: go1.13.10
 Git commit: 48a66213fe
 Built: Mon Jun 22 15:43:18 2020
 OS/Arch: windows/amd64
 Experimental: false
Server: Docker Engine - Community
 Engine:
  Version: 19.03.12
  API version: 1.40 (minimum version 1.24)
  Go version: go1.13.10
  Git commit: 48a66213fe
  Built: Mon Jun 22 15:57:30 2020
  OS/Arch: windows/amd64
  Experimental: false
```

If the Server section is giving an error that it cannot connect, then you are either not running as administrator or your service is not running.

If the server section says Linux in OS/Arch then you didn’t switch to Windows Containers.

If you want to check that Docker can be used to run Windows containers, start a CMD prompt as administrator. In the very first line of the CMD prompt, your Windows version is displayed. Try to pull a Windows ServerCore image with the same version number:

![](/assets/images/2020/troubleshooting-business-central-on-docker/image-1.png)

If this fails, then your Windows Version is either too old or too new – meaning that there isn’t a Windows ServerCore image for your version of Windows. If your Windows Version is too old – update it – get on the latest version. If your Windows Version is too new (insider or prerelease like mine), then you need to find the latest released version number and you might be able to run with a specific version of ServerCore. If you search for **Windows 10 19041 september 2020** (where 19041 is the Windows Build number shown in your error and September 2020 is the month and year for which you want the update) using your favorite internet search engine, you should get something like this:

![](/assets/images/2020/troubleshooting-business-central-on-docker/508.png)

and then try using that. Running the docker run command should give you something like this:

![](/assets/images/2020/troubleshooting-business-central-on-docker/prompt.png)

If this works, then you might be able to get Business Central to work as well. In general, I do not recommend running Docker on a Windows 10 insider build or a host build that is not available as a docker image.

If this fails, you might be able to see the cause in the list of frequently seen Docker errors at the end of this blog post. There is no need in trying to run Business Central containers if Windows ServerCore doesn’t work.

# Installing BcContainerHelper

BcContainerHelper is a PowerShell module which can be found on the [PowerShell Gallery](https://www.powershellgallery.com/packages/BcContainerHelper) and can be installed using

```
Install-Module BcContainerHelper -Force
```

Between every release of BcContainerHelper, there might be a number of prereleases. In Package details under every release and/or prerelease, you will find information about what’s new in this release.

Installing a prerelease version requires you to add -allowprerelease:

```
Install-Module BcContainerHelper -Force -AllowPrerelease
```

A few errors have been seen when people try to install the BcContainerHelper.

**PackageManagement\\Install-Package : The following commands are already available on this system:’Add-FontsToBCContainer,Backup-BCContainerDatabases,Compile-AppInBCContainer,Copy-CompanyInBCContainer,Copy-FileFromBCContainer, …** This error appears if you have NavContainerHelper installed and try to install BcContainerHelper. Please uninstall all versions of NavContainerHelper before installing BcContainerHelper. This can be done using _Uninstall-Module NavContainerHelper -allversions_. Note also that you need to remove all containers before switching from NavContainerHelper to BcContainerHelper.

**Install-Module : The ‘Install-Module’ command was found in the module ‘PowerShellGet’, but the module could not be loaded. For more information, run ‘Import-Module PowerShellGet’.** When you then try to run Import-Module, you get another error saying that PackageManagement.psm1 cannot be loaded because running scripts is disabled on this system. As the error states, this is because script execution is disabled, you can run _Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser_ or

**Install-Module : The ‘Install-Module’ command was found in the module ‘PowerShellGet’, but the module could not be loaded. For more information, run ‘Import-Module PowerShellGet’.** When you then try to run Import-Module, you get another error saying that PackageManagement.psm1 cannot be loaded because running scripts is disabled on this system. As the error states, this is because script execution is disabled, you might need to run as Administrator or you can run _Set-ExecutionPolicy -ExecutionPolicy Unrestricted_ or _Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser_ to be able to install the module.

**PackageManagement\\Install-Package : Package ‘bccontainerhelper’ failed to download.** This is typically because PowerShell Gallery recently started to require the client to have TLS 1.2 enabled and on older Windows 10 versions, this is not enabled by default. You can try to run _\[Net.ServicePointManager\]::SecurityProtocol = \[Net.ServicePointManager\]::SecurityProtocol -bor \[Net.ServicePointManager\]::Tls12_ to enable TLS 1.2 and retry.

**Install-Module : A parameter cannot be found that matches parameter name ‘allowPrerelease’**. When you are trying to install the prerelease version of BcContainerHelper, you might run into this issue. The solution is to update PowerShellGet using _Install-Module -Name PowerShellGet -Repository PSGallery -Force_ and try again. Note that you need to restart PowerShell after this.

# Creating a container

When BcContainerHelper is successfully installed, the next thing is to create containers. Below you will find some of the most frequent reasons for issues when creating containers:

## Using “outdated” docker images

As described here: [https://freddysblog.com/2020/06/25/changing-the-way-you-run-business-central-in-docker/](/2020/06/25/changing-the-way-you-run-business-central-in-docker/) we no longer update the specific images on mcr.microsoft.com. When creating a container you will see a warning:

```
WARNING: You are running specific Docker images from mcr.microsoft.com. These images will no longer be updated, you should switch to user Docker artifacts. See https://freddysblog.com/2020/07/05/july-updates-are-out-they-are-the-last-on-premises-docker-images/
```

The solution is to change your scripts to use artifacts. Artifacts exists for ALL versions of NAV or Business Central images.

## Using Windows Authentication

Typically this doesn’t cause problems while creating the container, but can cause the Web Client to be inaccessible from a browser. The solution is to use Username/Password authentication.

## Process/Hyperv isolation

New-BcContainer will always try to find the best matching container OS for your host OS, but if you have signed up for insider builds of the host OS or you have downloaded a preview image, there might not be a matching windows servercore image (as explained in the beginning). Some times you can still run process isolation by specifying _\-isolation process_, but you might encounter issues. If the container is run as hyperv isolation, you will have a hard memory limit, which by default is set to 4G and that might not be enough.

## Unable to connect to Web Client

I have seen this one a few times lately. Most times, people are not using the URL displayed in the output stream from New-BcContainer. The Web Client Url might include ?tenant=default, in which case – this needs to be included.

Other reasons for this have been that people have used Windows Authentication and the username and password of the Windows user isn’t the same as the windows user on the host or the host is not connected to the AD server and running on cached credentials. Shift to username password authentication to make this work.

I have also seen people using proxies or VPNs, which causes connections to local containers to fail. Try to disable or configure these properly.

## “Too many” parameters

I get it, New-BcContainer has a LOT of parameters and a LOT of options and some times, people are trying to make the function do what they want by trying out different parameters.

It seldom works and is very hard for anybody to help troubleshoot. My recommendation is to use New-BcContainerWizard to create a script and run that un-modified. If this fails, then it is likely your setup.

```
New-BcContainer `    -accept_eula `    -artifactUrl (Get-BCArtifactUrl) `    -auth UserPassword `    -Credential (Get-Credential) `    -updateHosts
```

The output on this command on my machine is:

```
BcContainerHelper is version 1.0.8
BcContainerHelper is running as administrator
Host is Microsoft Windows 10 Enterprise - Unknown/Insider build
Docker Client Version is 19.03.13
Docker Server Version is 19.03.13
Removing container bcserver
Removing bcserver from host hosts file
Removing bcserver-* from host hosts file
Removing C:\ProgramData\BcContainerHelper\Extensions\bcserver
Fetching all docker images
Using image mcr.microsoft.com/dynamicsnav:10.0.19041.508-generic
Creating Container bcserver
Version: 17.0.17126.17549-W1
Style: sandbox
Multitenant: Yes
Platform: 17.0.17020.17501
Generic Tag: 0.1.0.23
Container OS Version: 10.0.19041.508 (2004)
Host OS Version: 10.0.19041.508 (2004)
Using Process isolation
Using locale en-US
Disabling the standard eventlog dump to container log every 2 seconds (use -dumpEventLog to enable)
Files in C:\ProgramData\BcContainerHelper\Extensions\bcserver\my:
AdditionalOutput.ps1
MainLoop.ps1
SetupVariables.ps1
updatehosts.ps1
Creating container bcserver from image mcr.microsoft.com/dynamicsnav:10.0.19041.508-generic
20f874a3ff015521a1cefe684f5b984a7bac1a3e64c43b9f7cdc3fe504869df7
Waiting for container bcserver to be ready
Using artifactUrl https://bcartifacts.azureedge.net/sandbox/17.0.17126.17549/w1
Using installer from C:\Run\150-new
Installing Business Central
Installing from artifacts
Starting Local SQL Server
Starting Internet Information Server
Copying Service Tier Files
Copying PowerShell Scripts
Copying dependencies
Copying ReportBuilder
Importing PowerShell Modules
Determining Database Collation from c:\dl\sandbox\17.0.17126.17549\w1\BusinessCentral-W1.bak
Restoring CRONUS Demo Database
Exporting Application to CRONUS
Removing Application from tenant
Modifying Business Central Service Tier Config File for Docker
Creating Business Central Service Tier
Installing SIP crypto provider: 'C:\Windows\System32\NavSip.dll'
Copying Web Client Files
Copying Client Files
Copying ModernDev Files
Copying additional files
Copying ConfigurationPackages
Copying Test Assemblies
Copying Extensions
Copying Applications
Starting Business Central Service Tier
Importing license file
Copying Database on localhost\SQLEXPRESS from tenant to default
Taking database tenant offline
Copying database files
Attaching files as new Database default
Putting database tenant back online
Mounting tenant database
Mounting Database for default on server localhost\SQLEXPRESS
Sync'ing Tenant
Tenant is Operational
Stopping Business Central Service Tier
Installation took 189 seconds
Installation complete
Initializing…
Setting host.docker.internal to 192.168.0.22 in container hosts file (copy from host hosts file)
Setting gateway.docker.internal to 192.168.0.22 in container hosts file (copy from host hosts file)
Setting kubernetes.docker.internal to 127.0.0.1 in container hosts file (copy from host hosts file)
Setting host.containerhelper.internal to 172.29.208.1 in container hosts file
Starting Container
Hostname is bcserver
PublicDnsName is bcserver
Using NavUserPassword Authentication
Creating Self Signed Certificate
Self Signed Certificate Thumbprint 8BCAE7BC880DE63E23401CBC23C6CAAE4C92C36B
Modifying Service Tier Config File with Instance Specific Settings
Starting Service Tier
Registering event sources
Creating DotNetCore Web Server Instance
Enabling Financials User Experience
Enabling rewrite rule: Don't rewrite system files
Enabling rewrite rule: Already have tenant specified
Enabling rewrite rule: Hostname (without port) to tenant
Dismounting Tenant
Mounting Tenant
Mounting Database for default on server localhost\SQLEXPRESS
Sync'ing Tenant
Tenant is Operational
Creating http download site
Setting SA Password and enabling SA
Creating admin as SQL User and add to sysadmin
Creating SUPER user
Container IP Address: 172.29.213.164
Container Hostname : bcserver
Container Dns Name : bcserver
Web Client : http://bcserver/BC/?tenant=default
Dev. Server : http://bcserver
Dev. ServerInstance : BC
Dev. Server Tenant : default
Setting bcserver to 172.29.213.164 in host hosts file
Setting bcserver-default to 172.29.213.164 in host hosts file

Files:
http://bcserver:8080/ALLanguage.vsix

Container Total Physical Memory is 63.8Gb
Container Free Physical Memory is 38.9Gb

Initialization took 41 seconds
Ready for connections!
Reading CustomSettings.config from bcserver
Creating Desktop Shortcuts for bcserver
Container bcserver successfully created

Use:
Get-BcContainerEventLog -containerName bcserver to retrieve a snapshot of the event log from the container
Get-BcContainerDebugInfo -containerName bcserver to get debug information about the container
Enter-BcContainer -containerName bcserver to open a PowerShell prompt inside the container
Remove-BcContainer -containerName bcserver to remove the container again
docker logs bcserver to retrieve information about URL's again
```

## Keep the script, Not the container

Windows updates are released every month for Windows computers and it is our recommendation that you apply these Windows updates to your host. This also means that you should update your containers with the monthly update. Unfortunately, there is no good way to apply Windows updates to containers, so the best option is just to re-create the container.

New-BcContainer is designed to give partners the ability to create and configure containers easily. It was never the intention that containers should be running for months, so instead of trying to keep a container alive, keep the script used to create and configure the container and re-run monthly when Windows updates have been applied.

If you need the database to live longer than the container you can use _Backup-BcContainerDatabases_ and _Restore-DatabasesInBcContainer_ – or you can use SQL Server on the host ([https://freddysblog.com/2019/11/04/using-sql-server-on-the-host/](/2019/11/04/using-sql-server-on-the-host/))

Also, if you open issues on [https://www.github.com/microsoft/navcontainerhelper/issues](https://www.github.com/microsoft/navcontainerhelper/issues) you will need to provide script and full output of the script and I will in general not help people try to troubleshoot a running container. Be prepared to re-run the script and re-create the container and share the script AND the full output.

# Frequently seen Docker errors

Here are some of the common errors given by Docker when trying to run a container and what you can do about them:

**docker.exe: Error response from daemon: CreateComputeSystem {ID}: No hypervisor is present on this system.** This issue is self-explanatory – you are trying to run a container in hyperv mode and the hypervisor is not present. Either install Hyper-V and the containers feature – or use process isolation.

**docker.exe: Error response from daemon: container {ID} encountered an error during hcsshim::System::Start: failure in a Windows system call: The virtual machine or container exited unexpectedly. (0xc0370106). ExitCode: 125** This error might be related to a Windows Update happening while containers are running. Best recommendation is to run a Reset to Factory default of Docker Desktop or follow [this blog post](/2018/12/11/clean-up-after-yourself-docker-your-mom-isnt-here/) when running Docker Enterprise.

**docker: Error response from daemon: hcsshim::CreateComputeSystem {ID}: The container operating system does not match the host operating system.** This error is given when you are trying to run an image, which was designed for another version of Windows in process isolation. You would need to use hyperv isolation or find an image, which matches the host OS either by locating another image or updating the host.

**docker.exe: Error response from daemon: hcsshim::PrepareLayer – failed failed in Win32: Incorrect function. (0x1). ExitCode: 125** This error has been seen on some machines and as mentioned in [this issue](https://github.com/microsoft/navcontainerhelper/issues/1152) it might be caused by a Windows Driver called cbfs6.sys.

**docker.exe: Error response from daemon: hcsshim::CreateComputeSystem {ID}: The requested resource is in use.** As discussed in [this issue](https://github.com/microsoft/navcontainerhelper/issues/1099), this error might be caused by lack of memory on the host. Add memory, remove containers, reboot computer seems to be possible mitigations.

**hcsshim::CreateComputeSystem {ID}: The request is not supported.** This error seems to appear when running different releases of the OS on the host and in the Container. Typically using hyperv isolation or installing latest Windows updates solves this issue. This might also be caused by trying to run in hyperv isolation when hardware or VM doesn’t support it.

**Failed to register layer: re-exec error: exit status 1: output: BackupWrite \\?{PATH}: Incorrect function. ExitCode: 1** I have only seen this error once and it was related to a data-root configuration setting in Docker. After removal of the setting, the problem went away.

**DockerDo : docker.exe: Error response from daemon: hcsshim::CreateComputeSystem {ID}: The virtual machine could not be started because a required feature is not installed.** This might be due to starting a container with hyperv isolation and the underlying hardware or VM doesn’t support virtualization or Hyper-V / Windows Containers are not installed.

**docker.exe: Error response from daemon: hcsshim::CreateComputeSystem {ID}: Wrong parameter**. This is a very generic error, but every time I have seen this, it was caused by Hyper-V not being properly installed.

**docker.exe: Error response from daemon: hcsshim::PrepareLayer – failed failed in Win32: The device is not ready. (0x15). ExitCode: 125** This error is seen a few times and have typically been due to exhausted resources. Adding more memory to the host and/or cleaning up docker using Reset to Factory default of Docker Desktop or follow [this blog post](/2018/12/11/clean-up-after-yourself-docker-your-mom-isnt-here/) when running Docker Enterprise.

**Failed to register layer: re-exec error: exit status 1: output: ProcessUtilityVMImage \\?{PATH}: The system cannot find the path specified. ExitCode: 1** Seen only a few times, where an attempt was done to run a docker image of a newer Windows Version on an older host. Update the host or locate a corresponding image.

# The TSG

if you are seeing other Docker errors, then it is typically very very hard to troubleshoot and I will always ask you to follow the following TSG:

1.  Inspect the New-BcContainer output – any warnings?
    1.  DNS not working, amount of memory, running old images, etc. – fix them
2.  Make sure your Windows host OS has the latest updates.
3.  Make sure you are running the latest Docker version
4.  Make sure you have the latest BcContainerHelper
5.  Remove special docker settings
    1.  Docker data drive, experimental features and more.
6.  Try to run the Windows Server Core image without NAV/BC (top part of this blog post)
    1.  If this doesn’t work, then New-BcContainer also won’t work.
    2.  If there isn’t a windows servercore image for your host OS, then you are likely running a preview and you might need to spin up an Azure VM using [http://aka.ms/getbc](http://aka.ms/getbc) until container OS’ get up to the same version.
7.  Try the most basic New-BcContainer (second part of this blog post)
    1.  If this doesn’t work, my best suggestion is that it can be antivirus, lack of memory, 3rd party software or other policies enforced on you machine blocking for Business Central to run. Your best option is probably to use [http://aka.ms/getbc](http://aka.ms/getbc) and use an Azure VM.
    2.  If this works, then try to add the parameters one by one to see which parameter causes the issues or use [http://aka.ms/getbc](http://aka.ms/getbc) to create a VM and try your script there.

Hope this is helpful

**_Freddy Kristiansen_**  
Technical Evangelist
