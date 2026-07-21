---
layout: post
title: "You are running a container which is xx days old…"
date: 2019-07-26 15:09:24
categories: ["Docker", "NavContainerHelper", "Uncategorized"]
tags: ["accept_outdated", "Docker", "NAV on Docker", "NavContainerHelper", "new-navcontainer"]
permalink: /2019/07/26/you-are-running-a-container-which-is-xx-days-old/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

Recently, when running an instance of a Microsoft Dynamics NAV Docker image from the docker hub, you will get a message like:

```
You are running a container which is 78 days old.
Microsoft recommends that you always run the latest version of our containers.
```

What does this mean? what happens if you just ignore this? and what can you do about it?

# What does this mean?

Well, it means that the image you are about to use is based on a generic image, which is 78 days old. It might sound like it implies that there is a newer image – but that is not necessarily true.

The reason for the warning is basically that your container might be based on an old windows server core image which might need security updates or other updates.

# What happens if you just ignore this?

Note that if the image is more than 90 days old – the warning will turn into an error.

# What can you do about it?

When using NavContainerHelper, you can specify **\-alwaysPull** to always make sure that you are running the latest image version and you can use **\-accept\_outdated** in order to turn the error into a warning.

If you are using docker run – you need to use **–env accept\_outdated=Y** in order to not be blocked by the error.

Hope this is helpful

**_Freddy Kristiansen_**  
Technical Evangelist
