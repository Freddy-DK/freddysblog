---
layout: post
title: "New-BcContainerWizard aka Business Central on Docker script generator…"
date: 2020-07-03 13:43:39
categories: ["Docker", "NavContainerHelper", "PowerShell"]
tags: ["Docker", "New-BcContainer", "PowerShell"]
permalink: /2020/07/03/business-central-on-docker-script-generator/
---

**Update 2021/2/10:** BcContainerHelper has replaced NavContainerHelper. This blog post still reference NavContainerHelper, which is outdated.

I have been wanting to create a repository of scripts, where you could locate the right script for your usage. The problem I ran into was however that the number of scripts in a repository like that would very quickly explode. Yesterday evening while riding my bike, I got an idea…

Why not create a script generator script for Business Central on Docker. A script, which asks you a lot of questions and then generates a script, which you can save and run.

# New-BcContainerWizard

The script will be evolved over time, but it can already do cool stuff and feedback is very welcome…

Before running any scripts, you will need the latest NavContainerHelper module installed ([https://www.powershellgallery.com/packages/navcontainerhelper](https://www.powershellgallery.com/packages/navcontainerhelper))

With the latest ContainerHelper you can run

New-BcContainerWizard

Then you will run the version of the script, which is included in your version of NavContainerHelper.

There might however be a newer version of the script online. The way you can run this is, that you start Windows PowerShell (ISE and VSCode also works) and run this script:

Invoke-Expression (New-Object System.Net.WebClient).DownloadString("http://aka.ms/bcdockerscript")

You are of course welcome to download and inspect the script if you like, but the script will be updated in its location regularly. When launching the script you should see:

![](/assets/images/2020/business-central-on-docker-script-generator/accept-eula-3.png)

Type Y and continue. Now you will be lead through a number of questions and for each question, I will try to explain what this means. First question is if you want to run locally or inside an Azure VM.

![](/assets/images/2020/business-central-on-docker-script-generator/localorazure.png)

You will also need to specify which Business Central version you want and which country version you want and in the end, the script will display the script needed to create the container you want, giving you a chance to save it and execute it:

![](/assets/images/2020/business-central-on-docker-script-generator/execute.png)

Executing the script should give you a container, with the things you specified in the script:

![](/assets/images/2020/business-central-on-docker-script-generator/output.png)

If you instead select an Azure VM, then you “script” will be a URL, which launches one of the ARM templates with a number of parameters pre-filled with your selectioins.

![](/assets/images/2020/business-central-on-docker-script-generator/url.png)

# Try it out!

Try it out and let me know if there are bugs and what you think is missing?

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
