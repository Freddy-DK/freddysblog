---
layout: post
title: "Weekend cleanup… – done"
date: 2019-12-12 11:13:40
categories: ["CI/CD", "Docker"]
tags: ["Docker"]
permalink: /2019/12/12/weekend-cleanup/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

_**Update:** Weekend cleanup is done and the latest daily builds from master (next major) and 15.x (next minor) are updated. I have also updated the number below from 45 days to 7 days as I don’t see any reason to have older insider builds. Let me know if you think differently, thanks._

It is cleanup time. Our insider repository has become very very big and it is time to clean up. The problem however is, that the current insider registry has reached a size, where it is very hard to clean up, so…

# Remove, Recreate and Rebuild…

**December 14th 2019, Saturday morning (UTC)**, I will **remove the bcinsider registry**, **recreate it** (empty) and **rebuild the latest images**.

In the future we will have daily builds for the latest ~7 days on bcinsider.azurecr.io and setup automatic removal of images older than 7 days.

If you by Saturday evening (UTC) are **missing images from the insider registry**, please open an issue here: [https://github.com/microsoft/nav-docker/issues](https://github.com/microsoft/nav-docker/issues) and we will investigate.

# Failing pipelines all over the world…

This means that **CI/CD pipelines all over the world might be failing** due to unknown registry, unknown repositories or unknown images, but this should only occur for a few hours on Saturday morning, then the latest images should be available again.

# Sorry…

So, sorry for the inconvenience, I hope you can use your Saturday to buy some presents for your family instead:-)

Thanks

_**Freddy Kristiansen**_  
Technical Evangelist
