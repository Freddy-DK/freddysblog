---
layout: post
title: "Running tests on Business Central 16.0"
date: 2020-04-02 23:36:24
categories: ["Docker", "NavContainerHelper"]
tags: ["Docker", "Tests"]
permalink: /2020/04/02/running-tests-on-business-central-16-0/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

On April 1st, we shipped Microsoft Dynamics 365 Business Central 2020 Wave 1. It is a great release with a lot of exciting new features, which I will leave to others to write about and describe.

I will instead write about a small issue on the 19 local 16.0 DVDs you can download. They all contain test framework apps and our Microsoft test apps which ships for W1.

Yes, you read it right. The local test apps wasn’t included on to the DVDs, meaning that when you try to create a container with -includeTestToolkit, then it will fail in some of the local versions, but you also won’t be able to download the DVD and make it work for you.

# Breaking pipelines

Obviously this caused pipelines using Business Central on-premises containers to fail as a number of the countries cannot run tests:-( Sandbox containers are NOT affected.

The bug will be fixed in CU1, were the apps on the DVD will be the right ones, but in the meantime we need a solution for all the pipelines out there, which are failing.

The build process of containers are building images based on DVDs and I am not very keen to modify the build process to grab some files from another location when building. the image. That is a disaster waiting to happen.

Instead I have added a check in NavContainerHelper, which will download the right .app files and patch the files in the container just-in-time (only if it is the 16.0 rtm container of course).

# NavContainerHelper 0.6.5.3

So, if you will be running tests in non-w1 16.0 containers, you are best off by updating your NavContainerHelper to 0.6.5.3 and recreate your containers.

I hope this is valuable.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
