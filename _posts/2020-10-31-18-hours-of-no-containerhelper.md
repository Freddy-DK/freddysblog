---
layout: post
title: "18 hours of no containerhelper…"
date: 2020-10-31 19:24:26
categories: ["Azure", "BcContainerHelper"]
tags: ["BcContainerHelper"]
permalink: /2020/10/31/18-hours-of-no-containerhelper/
---

Friday morning around 6am. I had just kicked off a few validation builds when they started failing. Failing builds can happen and typically there is a valid reason for this, but in this case – ALL builds was failing and looking in the logs I quickly realized that this problem was something different.

PowerShell Gallery was down, PowerShell Gallery had an outage and it took a stunning 18 hours before it was back online…:-(

# Issues, Emails, Tweets etc…

I got a bunch of emails, a few replies on my tweet about the outage and a few issues on github. It seems like everybody knew that this was bigger than the containerhelper.

One of the github issue suggested to create an alternate package source for the containerhelper. This blog post will explain exactly this.

# Mitigation

Fortunately, I already had support for being able to download BcContainerHelper from a URL and using a direct github url like: [https://github.com/microsoft/navcontainerhelper/archive/master.zip](https://github.com/microsoft/navcontainerhelper/archive/master.zip) I would get the current content of the master repository.

Basically, you would have to download the .zip file, unpack, remove earlier loaded versions of bcContainerHelper using Remove-Module and import the new module by using Import-Module on the .psm1 file.

The problem with relying on master.zip is that you get the content of the repo before it has been tested and there could be bugs and un-tested issues. You really want to get the containerhelper AFTER it has passed all tests.

# Storage

So, as of today, BcContainerHelper will publish .zip files to a storage account together with publishing to PowerShell Gallery.

[https://bccontainerhelper.azureedge.net/public/latest.zip](https://bccontainerhelper.azureedge.net/public/latest.zip) will always be the latest non-prerelease version shipped on the PowerShell Gallery.

[https://bccontainerhelper.azureedge.net/public/preview.zip](https://bccontainerhelper.azureedge.net/public/preview.zip) will always be the latest prerelease version shipped on the PowerShell Gallery.

Using these urls for downloading the releases might even be faster than using the PowerShell Gallery, but it is a bit more cumbersome.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
