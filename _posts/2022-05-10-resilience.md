---
layout: post
title: "Resilience"
date: 2022-05-10 11:58:57
categories: ["BcContainerHelper", "Docker"]
permalink: /2022/05/10/resilience/
---

Lately we have been seeing an increasing number of people having difficulties creating Docker containers on multiple host OS’. Since Thursday, I have been diving into error reports on GitHub, looking in Telemetry and had some partners helping out trying various tests to see what the result of various changes would be.

As an example, achim-t have run test scripts at least 20 times [here](https://github.com/microsoft/AL-Go/issues/103), since it wasn’t possible for me to repro the problems, but also [this](https://github.com/microsoft/navcontainerhelper/issues/2434), [this](https://github.com/microsoft/navcontainerhelper/issues/2374) and several other issues are from people having issues. Typically, the problem goes away when you use hyperv isolation, when you switch off Windows Defender or other anti-virus software – or when you are running on a Windows Server OS.

## New Generic Image

As a result, a new generic image (**currently version 1.0.2.4**), which should resolve most of these issues (for **NAV/BC versions 15 and up**) is available in preview at this time – add **\-dev** to the generic image name and you will get the preview.

```
-useGenericImage "$(Get-BestGenericImageName)-dev"
```

Also, the latest BcContainerHelper preview will automatically select the preview image when creating containers, there you don’t have to do anything. The next BcContainerHelper will even present you with a **warning if you are running an OS, isolation mode and/or NAV/BC version** which might require you to **use Hyper-V or switch off Windows Defender** while creating the container.

You can determine which generic image version you are using by looking for the **Generic Tag** in the container output (the -dev part is included in the next BcContainerHelper version):

![](/assets/images/2022/resilience/image-42.png)

The **new generic image** will likely be promoted to **prod** within the next week, so please let me know if you run into any issues when using this version, by creating an issue on [https://github.com/microsoft/navcontainerhelper/issues](https://github.com/microsoft/navcontainerhelper/issues).

**Note**: Do not respond to other people’s issues – create your own, thanks.

Thanks

**_Freddy Kristiansen_**  
Technical Evangelist
