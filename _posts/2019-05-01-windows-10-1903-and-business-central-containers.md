---
layout: post
title: "Windows 10 1903 (and Business Central Containers)"
date: 2019-05-01 09:29:26
categories: ["Docker", "NavContainerHelper", "PowerShell"]
tags: ["1903", "Docker", "Generic", "Image", "ltsc", "NavContainerHelper", "new-navcontainer", "sac"]
permalink: /2019/05/01/windows-10-1903-and-business-central-containers/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

Some weeks ago, I read about Windows 10 1903 in a blog post from Mike Fortin here: [https://blogs.windows.com/windowsexperience/2019/04/04/improving-the-windows-10-update-experience-with-control-quality-and-transparency](https://blogs.windows.com/windowsexperience/2019/04/04/improving-the-windows-10-update-experience-with-control-quality-and-transparency). Since general rolllout wasn’t until May, I decided to wait until after Directions NA to do my update.

1903 was made available on MSDN and internally for people who wanted to update early and I didn’t really pay too much attention to this. Two evenings ago, a colleague ping’ed m e with an error when trying to create a container. Looking at the output, the containerhelper identified his Windows version as unknown/insider.

I promised him to look into the issue, but I needed to borrow his machine, I didn’t really want to upgrade my own machine to 1903 less than one week before Directions NA.

# Upgrade started

Not sure what I did, but after a reboot of my laptop, it started downloading and applying 1903. As I read in the blog post, I am sure that I could have stopped it but then again – I might have more colleagues going to Directions with 1903 experiencing issues, who am I to postpone my own update and not face the issues (if there are any), so…![Configuringupdates](/assets/images/2019/windows-10-1903-and-business-central-containers/configuringupdates.jpg)

So I decided to take the update and find out what works and what doesn’t.

# Reset to factory defaults

After the update, I had to reset my docker installation to factory defaults, meaning that all containers where deleted and I am basically starting from scratch. This might sound problematic, but IMO you should never have containers, which you cannot recreate or replace at any time, so not really a big deal.

# No-break space

After the update, I ran New-NavContainer and I immediately saw the same error as my colleague:

_The term ‘ï¿½’ is not recognized as the name of a cmdlet, function, script file, or operable program. Check the spelling of the name, or if a path was included, verify that the path is correct and try again._

Investigating the script where the error occured revealed a unicode character C2A0 ([http://www.fileformat.info/info/unicode/char/00a0/index.htm](http://www.fileformat.info/info/unicode/char/00a0/index.htm)). Have no idea why there is a difference between 1809 and 1903 in PowerShells interpretation of a no-break space, but it seems there is.

I removed the character and my container was created.

# Tests

In order to make sure that I wouldn’t get other surprises, I ran the tests from NavContainerHelper and everything passed. I know that I do not have full test coverage of everything in NavContainerHelper (need to work on that), but I have some good amount of tests that can validate the functionality.

Beside that, I also went through my demo scripts and everything seems to work fine, but…

# HyperV isolation

I wasn’t surprised to find that Windows 10 1903 was unable to run ltsc2019 (1809) containers in process isolation. After all – if a container is sharing the OS with the host, you kind of assume that the host and the container needs to be on the same build.

I did however find, that hyperv isolation has become lightning fast. In the “old” days I always found that spinning up containers in hyperv isolation took a looooong time and with the container pre-allocating the memory assigned, I really disliked hyperv isolation.

With Windows 10 1903 I almost don’t feel the difference. It doesn’t preallocate all the memory. In fact, when I start a hyperv container with 20G, it only uses the memory it needs and it even seems to release some of the memory to the host when cleaning up. (Now I am in doubt whether this was always the behavior)

# 1903 containers?

But, there are more reasons why one would run process isolation. I know some people who do not have the hypervisor supported and as such can only use process isolation and on the memory issue, I still find that process isolation is better at memory management.

So, how do we run Business Central containers using process isolation?

The easy approach would be to wait until 1903 containers are available – but will they be?

The strategy has been to create images for all ltsc (Long-Term Servicing Channel) builds of Windows Server and not on the sac (Semi-Annual Channel). This did cause challenges with 1709 and 1803, mostly because the difference between 1803 and the foundation of the container (lstc2016) was very big. To accommodate for this, we created the generic image for 1709 and 1803 and added the -useBestContainerOS flag to allow new-navcontainer to re-platform a container on-the-fly.

So, **as soon as 1903 is publicly available** we will have a generic image for 1903 and new-navcontainer will be able to re-platform ltsc2019 containers to 1903 on-the-fly and start them with process isolation if you use -useBestContainerOS.

# But, what if you cannot wait?

Note that you do NOT need to do this if a generic image of 1903 has shipped. As soon as 1903 ships, we will create a generic image, publish it and blog about it.

If your machine is on 1903 – and you need process isolation, then you still have options, but it means that you have to build a generic image yourself. I don’t think a lot of people will be doing this, but hey – it might be interesting for people to know the details, so – let’s do this…

**Step 1 – what build are you running?**

Run winver to identify the build of Windows you are running:

![Screenshot 2019-05-01 07.25.20](/assets/images/2019/windows-10-1903-and-business-central-containers/screenshot-2019-05-01-07.25.20.png)

**Step 2 – locate the best insider version of Windows**

Navigate to [https://mcr.microsoft.com/v2/windows/servercore/insider/tags/list](https://mcr.microsoft.com/v2/windows/servercore/insider/tags/list) in a brower and identify the best tag. The best tag is the newest build with the same build number as your host OS. If no build exists – you are out of luck.

Best tag for my host os is **10.0.18362.53**

**Step 3 – clone the nav-docker repository**

Clone this repository to your local machine: [https://github.com/Microsoft/nav-docker](https://github.com/Microsoft/nav-docker)

**Step 4 – edit and run mybuild.ps1**

In the generic folder you will find a script called mybuild.ps1 – edit that and set the baseimage tag to the one identified earlier as the best.

Also modify the image name and genericversion if necessary.

Run the script and after a while, you should have an image locally on your machine called mygeneric:latest – your own private generic image.

That was easy – right:-)

# Use your private generic image

The next step is to use your private generic image when spinning up containers. The -useBestContainerOS flag doesn’t know about your image, so it cannot automatically detect your image as a valid generic image. You will have to do things manually…

In the script below, I will create a parameter set in a variable called $imageParam, which is the replacement of the -imageName parameter to New-NavContainer. This parameter set determines whether to start the container as hyperv container or to replatform and use the private generic image.

$hyperv = $false

# Settings
$imageName = "mcr.microsoft.com/businesscentral/onprem:1904-rtm"
$auth = "NavUserPassword"
$credential = New-Object pscredential 'admin', (ConvertTo-SecureString -String 'P@ssword1' -AsPlainText -Force)
$imageName = Get-BestNavContainerImageName -imageName $imageName

if ($hyperv) {
    $imageParam = @{ 
        "imageName" = $imageName
        "memoryLimit" = "8G"
    }
}
else {
    docker pull $imageName
    $navVersion = Get-NavContainerNavVersion -containerOrImageName $imageName
    $navDvdPath = "c:\\ProgramData\\NavContainerHelper\\$($NavVersion)-Files"
    if (!(Test-Path $navDvdPath)) {
        Extract-FilesFromNavContainerImage -imageName $imageName -path $navDvdPath
    }
    $imageParam = @{
        "imageName" = "mygeneric"
        "navdvdPath" = $navDvdPath
        "navDvdCountry" = ($navVersion.Split('-')\[1\])
    }
}

New-NavContainer -accept\_eula @imageParam \`
                 -containerName "bc" \`
                 -auth $auth \`
                 -Credential $Credential \`
                 -updateHosts

On my machine, the difference on New-NavContainer when running process isolation and hyperv isolation is only 23 seconds. Process isolation is 23 seconds slower to start, but might run faster than hyperv isolation.

Note that the extracted files are stored in a shared location and will be reused if you create another instance of the same image.

# NavContainerHelper 0.6.0.14

In order to be able to run the above scripts, you will need NavContainerHelper 0.6.0.14 or later.

Stay tuned

_**Freddy Kristiansen**_  
Technical Evangelist
