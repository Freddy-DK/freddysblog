---
layout: post
title: "Clean up after yourself Docker, your mom isn’t here!"
date: 2018-12-11 15:04:26
categories: ["Docker"]
tags: ["Cleanup", "Docker", "Go", "Layers", "NAV on Docker"]
permalink: /2018/12/11/clean-up-after-yourself-docker-your-mom-isnt-here/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

_**Update**: After writing this blog post, Docker Desktop has a feature called **Reset To Factory defaults…**  I use this function now and then (instead of the DockerCiZap) to cleanup everything in my Docker environment. Right Click the Whale icon, select Settings -> Reset._

![reset](/assets/images/2018/clean-up-after-yourself-docker-your-mom-isnt-here/reset.png)

Docker is an amazing tool.

As Dynamics 365 Business Central or Microsoft Dynamics NAV developers we can spin up any version of the platform and app in minutes and use our favourite tools to work with it.

Using the layering techniques from Docker, these images will be stored on disc as efficient as possible and every time I start up a new container, it reuses the disc space and your running container just becomes another layer on top of your image.

Really really smart, but it does take up some disc space and it isn’t very good at cleaning up.

# Folders

Docker will store all images and containers in a folder called **C:\\ProgramData\\Docker**. You can modify the location of this folder when configuring Docker by changing a setting called **data-root**. Read more here: [https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-docker/configure-docker-daemon](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-docker/configure-docker-daemon)

NavContainerHelper will store extracted image files and object sources under **C:\\ProgramData\\NavContainerHelper**.

I typically run Docker on Azure VMs, which doesn’t live long. When I need a new version, I use [http://aka.ms/getbc](http://aka.ms/getbc) to spin up a new VM. I never really had any issues with these folders taking up space. I have however gotten the question from a number of people running Docker locally: **How do I clean up after Docker?** and after Windows 10 1809 came out and after Windows 10 supports process isolation (if you use Docker insider builds) I have started using my laptops more and more and I decided that I would investigate how this is done.

# The process is simple

What we need to do is simple:

1.  Remove all running Containers
2.  Remove all images, build caches and other things Docker related.
3.  Empty the NavContainerHelper folder
4.  Pull the images you need

It is the how-to, which becomes complicated – especially #2. You might have tried to delete the folder manually or tried to delete individual layers, just to find that you get a ton of errors and if you finally succeed deleting things, you might have ruined your host operating system and only a reinstall will get you back up running.

The reason for this is that the folders contains a lot of symbolic links (when they say that they are sharing the OS with the host, they mean it) and you need to be a backup/restore service in order to touch/delete these files. No user can do so, not even the administrator.

So who can?

# Docker-Ci-Zap can!

[Docker-Ci-Zap](https://github.com/jhowardmsft/docker-ci-zap) is an open source project, written in a programming language called go. Read more about go [here](https://golang.org/)!

It uses the [Host Compute Service Shim](https://github.com/microsoft/hcsshim), another go project, used of the Docker Host foundation.

It basically enumerates the layes and destroys them.

**Disclaimer: There is absolutely NO guarantee that this tool works and it might cause your computer to need a reinstall.**

But it worked for me.

So, I have built a script around this tool. A script, which performs the other 3 steps and as step 2, automatically downloads Docker-Ci-Zap and calls it with your Docker Data Dir. The script will detect that Docker is installed and determine that you are running Windows Containers and detect your Docker Root Folder.

If you successfully run the script, you should see something like:

[![](/assets/images/2018/clean-up-after-yourself-docker-your-mom-isnt-here/43046-remove2-1.png)](/assets/images/2018/clean-up-after-yourself-docker-your-mom-isnt-here/43046-remove2.png)

And both your Docker folder and your NavContainerHelper folder should now be empty.

If you encounter an error like:

[![](/assets/images/2018/clean-up-after-yourself-docker-your-mom-isnt-here/5c286-remove1-1.png)](/assets/images/2018/clean-up-after-yourself-docker-your-mom-isnt-here/5c286-remove1.png)

It is likely because you need a restart of your computer and a re-run of the script.

# NOTE: This script will delete all Containers and Images

The Script can be copied from below or downloaded from this URL: [https://raw.githubusercontent.com/Microsoft/nav-arm-templates/master/CleanupAfterDocker.ps1](https://raw.githubusercontent.com/Microsoft/nav-arm-templates/master/CleanupAfterDocker.ps1)

```
$ErrorActionPreference = "Stop"
$WarningActionPreference = "Stop"

# Specify which images to download
$ImagesToDownload = @()

$bcContainerHelperFolder = "C:\ProgramData\BcContainerHelper"

$currentPrincipal = New-Object Security.Principal.WindowsPrincipal([Security.Principal.WindowsIdentity]::GetCurrent())
if (!($currentPrincipal.IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator))) {
    throw "This script must run with administrator privileges"
}

Write-Host "Checking Docker Service Settings..."
$dockerService = (Get-Service docker -ErrorAction Ignore)
if (!($dockerService)) {
    throw "Docker Service not found / Docker is not installed"
}

if ($dockerService.Status -ne "Running") {
    throw "Docker Service is $($dockerService.Status) (Needs to be running)"
}

$dockerInfo = (docker info)
$dockerOsMode = ($dockerInfo | Where-Object { $_.Trim().StartsWith('OSType: ') }).Trim().SubString(8)
if ($dockerOsMode -ne "Windows") {
    throw "Docker is not running Windows Containers"
}

$dockerRootDir = ($dockerInfo | Where-Object { $_.Trim().StartsWith('Docker Root Dir: ') }).Trim().SubString(17)
if (!(Test-Path $dockerRootDir -PathType Container)) {
    throw "Folder $dockerRootDir does not exist"
}

Write-Host -Foregroundcolor Red "This function will remove all containers, remove all images and clear the folder $dockerRootDir"
Write-Host -Foregroundcolor Red "The function will also clear the contents of $bcContainerHelperFolder."
Write-Host -Foregroundcolor Red "Are you absolutely sure you want to do this? (This cannot be undone)"
Write-Host -ForegroundColor Red "Type Yes to continue:" -NoNewline
if ((Read-Host) -ne "Yes") {
    throw "Mission aborted"
}

Write-Host "Running Docker System Prune"
docker system prune -f

Write-Host "Removing all containers (forced)"
docker ps -a -q | % { docker rm $_ -f 2> NULL }

Write-Host "Stopping Docker Service"
stop-service docker

Write-Host "Downloading Docker-Ci-Zap"
$dockerCiZapExe = Join-Path $Env:TEMP "docker-ci-zap.exe"
Remove-Item $dockerCiZapExe -Force -ErrorAction Ignore
(New-Object System.Net.WebClient).DownloadFile("https://github.com/moby/docker-ci-zap/raw/master/docker-ci-zap.exe", $dockerCiZapExe)
Unblock-File -Path $dockerCiZapExe

Write-Host "Running Docker-Ci-Zap on $dockerRootDir"
Write-Host -ForegroundColor Yellow "Note: If this fails, please restart your computer and run this script again"
& $dockerCiZapExe -folder $dockerRootDir

Write-Host "Removing Docker-Ci-Zap"
Remove-Item $dockerCiZapExe

Write-Host "Starting Docker Service"
Start-Service docker

if (Test-Path $bcContainerHelperFolder -PathType Container) {
    Write-Host "Cleaning up $bcContainerHelperFolder"
    Get-ChildItem $bcContainerHelperFolder -Force | ForEach-Object { 
        Remove-Item $_.FullName -Recurse -force
    }
}

if ($ImagesToDownload) {
    Write-Host -ForegroundColor Green "Done cleaning up, pulling images for $os"

    $os = "ltsc2016"
    if ((Get-CimInstance win32_operatingsystem).BuildNumber -ge 17763) { $os = "ltsc2019" }
    
    # Download images needed
    $imagesToDownload | ForEach-Object {
        if ($_.EndsWith('-ltsc2016') -or $_.EndsWith('-1709') -or $_.EndsWith('-1803') -or $_.EndsWith('-ltsc2019') -or
            $_.EndsWith(':ltsc2016') -or $_.EndsWith(':1709') -or $_.EndsWith(':1803') -or $_.EndsWith(':ltsc2019')) {
            $imageName = $_
        } elseif ($_.Contains(':')) {
            $imageName = "$($_)-$os"
        } else {
            $imageName = "$($_):$os"
        }
        Write-Host "Pulling $imageName"
        docker pull $imageName
    }
}
```

I hope you find the explanation and the script helpful.

If the tool and the script doesn’t work for you, there isn’t much I can do about it.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
