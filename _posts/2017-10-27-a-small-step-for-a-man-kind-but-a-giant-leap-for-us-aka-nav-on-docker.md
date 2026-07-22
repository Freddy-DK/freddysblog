---
layout: post
title: "A small step for man kind but a giant leap for us! (aka NAV on Docker)"
date: 2017-10-27 13:42:39
categories: ["Docker"]
tags: ["Directions", "Docker", "NAV 2016", "NAV 2017", "NAV Tech Days"]
permalink: /2017/10/27/a-small-step-for-a-man-kind-but-a-giant-leap-for-us-aka-nav-on-docker/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

Some months ago, a colleague tried to convince me that Docker was the new black and I really had to look at that.

Today, we have started deploying Official Microsoft Docker Images to the Docker Hub – [https://hub.docker.com/r/microsoft/dynamics-nav/](https://hub.docker.com/r/microsoft/dynamics-nav/) and over the next days, all CUs and all country versions for NAV 2016 and NAV 2017 will be published on Docker Hub (Please read the description at the Docker Hub site)

Going forward, we will release images on the Docker Hub for every public version we ship.

We will also be creating Docker Images for preview builds but access to these will be given through the various partner programs we have and will be through MS Collaborate.

So how did we get from A to B?

# Denial

At first I was in denial.

What could this thing possibly do, which we couldn’t do with our Azure VMs?

What is the difference really between Docker and a Virtual Machine?

Isn’t it just a virtual machine in disguise? How do I connect to the Remote Desktop inside that Docker Image?

But usage scenarios kept coming up, which point towards Docker as being the solution and not the problem, so, I decided to have a look.

# Partner collaboration

Not surprisingly, some of our partners already started looking at Docker as well and they actually had succeeded in running NAV on Docker. I set up a call with Tobias Fenster from Axians Infoma and Jakub Vañák from Marqués Olivia and listened to their ideas and recommendations on what a Microsoft supported Docker image could do for out partners.

So, the journey started and I want to express my thanks to the help and support I got from Tobias and Jakub, but also from other partners, who along the way joined in and started testing the Docker images.

# Hard work… – some stats

3 – the number of projects created for this journey:

-   [http://www.github.com/microsoft/nav-docker](http://www.github.com/microsoft/nav-docker) – the actual docker base image for the [Official images for Microsoft Dynamics NAV](https://hub.docker.com/r/microsoft/dynamics-nav/)
-   [http://www.github.com/microsoft/navcontainerhelper](http://www.github.com/microsoft/navcontainerhelper) – the NAV Container Helper, a [PowerShell Module](https://www.powershellgallery.com/packages/navcontainerhelper), which makes life using Docker much easier
-   [http://www.github.com/microsoft/nav-arm-templates](http://www.github.com/microsoft/nav-arm-templates) – the Azure Resource Manager templates – ex. [http://aka.ma/navdeveloperpreview](http://aka.ma/navdeveloperpreview)

4650 – the number of PowerShell lines in these three projects

2400 – the number of Docker images created in the “private” Docker registry: navdocker.azurecr.io

143 – the number of Partners with access to the “private” docker registry, who has provided feedback and recommendations (not counting NAV Developer Preview deployments)

4 – the number of sessions already held on NAV on Docker at [Directions NA](http://www.directionsna.com/) and [Directions EMEA](http://www.directionsemea.com/)

90 – the number of minutes Tobias, Jakub and I will be talking about Docker at [NAV Tech Days 2017](https://www.navtechdays.com/2017) (friday 11:00)

1 – the number of partners who have expressed concerns about using Docker

740 – the number of NAV images which will be on the Docker Hub within the next week or so

# The future

In the coming days, I will post at least a blog post every day on how to use the Docker images – right from the most simple usage scenarios to the more advanced usage scenarios and i hope this will help the partners who are coming on to Docker to get a smooth onboarding process. (it hasn’t all been fun and games)

I’ll be back – enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
