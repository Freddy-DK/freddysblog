---
layout: post
title: "One of these days…"
date: 2020-08-18 08:08:46
categories: ["BcContainerHelper", "CI/CD", "Docker", "NavContainerHelper"]
tags: ["BcContainerHelper", "Docker", "Generic Image", "NavContainerHelper"]
permalink: /2020/08/18/one-of-these-days/
---

Many years ago, I heard a joke:

3 people (a biologist, a mathematician and a developer) were in Africa on a Safari. They drive by a blue elephant. The biologist shouts out: “Look, there is a **BLUE** elephant.”. The mathematician states: “Right you are, there is **ONE** blue elephant”. The developer slaps his palm against his forehead and says: “**Damn, there are blue elephants…**“

As developers we often make assumptions and when we see these assumptions no longer being true, we immediately start thinking: What does that mean to my code and what does that mean to the people using my code.

# This morning (August 18th 2020)

This morning, I woke up at 5:30 and as usual, the first thing I do is to scan email subjects. One email from Ryan caught my attention and in the email I could see a short output of the problem he was encountering:

```
Downloading Prerequisite Components
Downloading F:\bcart.cache\sandbox\16.4.14693.15627\platform\Prerequisite Components\Open XML SDK 2.5 for Microsoft Office\OpenXMLSDKv25.msi
Downloading F:\bcart.cache\sandbox\16.4.14693.15627\platform\Prerequisite Components\IIS URL Rewrite Module\rewrite_2.0_rtw_x64.msi
[2020-08-17 22:42:36] Exception calling "DownloadFile" with "2" argument(s): "The remote server returned an error: (404) Not Found."
```

What…

Did URL rewrite just disappear from Microsoft Download?

Can that happen?

and yes, the URL used in all docker images and artifacts to download URL rewrite is dead as of this morning… – there are blue elephants!

# Impact

The impact of this is pretty severe.

The dead URL is stamped into all artifacts and all docker images ever created.

During artifact download, the pre-requisites are also downloaded and that will fail. If you already have the artifacts downloaded and cached, you are good to go.

If you use Microsoft hosted agents for CI/CD pipelines, artifacts cannot download and your build will fail.

Next major / Next minor pipelines will likely fail as well.

# The fix?

I will try to get Microsoft Download to restore the file. That can however take some time and we cannot have pipelines failing all over the world for days…

In the meantime, I have created an Azure CDN with the pre-requisites used and in the Download-File function in containerhelper, I have added a replacement url array, meaning that if any attempt is made to download the dead url, it will be downloaded from my own cdn instead.

This fix is available in BcContainerHelper 1.0.3, which already has shipped (9am), and in NavContainerHelper 0.7.0.25, which ships in a few hours.

The same fix is added to Generic image 0.1.0.16 which also is available now.

So basically with an updated NavContainerHelper or BcContainerHelper and new generic images you should be good to go.

Sorry for the inconvenience

**_Freddy Kristiansen_**  
Technical Evangelist
