---
layout: post
title: "Zombie containers -> Skip This Build?"
date: 2019-08-07 06:51:34
categories: ["Docker", "NavContainerHelper"]
tags: ["Docker"]
permalink: /2019/08/07/zombie-containers-skip-this-build/
---

_**Yet Another Update:** A cure is found… [http://freddysblog.com/2019/09/05/a-cure-for-zombie-containers/](http://freddysblog.com/2019/09/05/a-cure-for-zombie-containers/)_

_**Another Update:** Docker engine 19.03.2 (and docker Desktop 2.1.0.2) should be released first week of September with a cure for Zombie containers._

_**Update:** A way to bring your Zombie Container back to live is in the comments by **Mick Carr (THANKS MICK)** – just tried this and it works on my Computer with 2.1.0.1 as well. Basically modify the containers config.v2.json, change running to false, restart docker and now your container is dead (not living dead). Use docker start to start the container and it comes back to life…_

Over the last few days I have experienced a strange behavior with the latest version of **Docker Desktop Community edition (2.1.0.0 (36874) released July 31, 2019)** on my Windows 10 1903 machine. Thinking this was a problem with my machine, I decided to postpone the investigation, while working on other issues. Yesterday I had two partners contact me with the same behavior, it was going to be a long night…

# The problem

It is very simple to reproduce the problem. If you restart Windows while your NAV or Business Central containers are running, they will (after the restart) still report as running (as they should due to restart setting), but the containers will be like zombies.

-   docker ps will say they are running
-   docker exec -it container cmd will say “no such container”
-   docker stop container will freeze and not be able to stop the container
-   docker rm container will freeze and if you break this and retry it will say that it is being removed.
-   Trying to ping the container or in other way contact the container fails

A living dead container – a zombie.

I can reproduce this using New-NavContainer or Docker Run – no difference and I am in contact with the team in Microsoft, who are working with Docker on this issue.

I don’t think this is related to NAV or Business Central containers. It also doesn’t matter whether it is running Hyper-V or Process isolation nor does it make any difference if I have updatehosts, volume shares or any other things enabled.

The only thing that worked for me, was to uninstall Docker and reinstall this version: **Docker Community Edition 2.0.0.3 2019-02-15**, which you can find and download here: [https://docs.docker.com/docker-for-windows/release-notes/](https://docs.docker.com/docker-for-windows/release-notes/)

I will continue working with the team in Microsoft and with Docker to find a solution and see whether we need to change anything in our containers, but for now, my recommendation is to **Skip This Build**.

Note that when you uninstall, it will remove all containers and images. Since I haven’t found any way to bring the Zombie containers back to life, I guess this is the only thing to do.

When you start this version (or if you haven’t upgraded yet) you will be met with this dialog:

![skipthisversion](/assets/images/2019/zombie-containers-skip-this-build/skipthisversion.png)

On this dialog, I pressed **Skip This Build**.

# One more “workaround”

Only other workaround I found is to stop the containers before restarting Windows and start them afterwards manually. This of course led me to believe that I could set the restart option to **no**, but that also doesn’t work. The new Docker version will still report it as running after Windows restart.

# What’s next

I will update this blog post as soon as I know more – for now, I just wanted to make sure that our partners aren’t wasting valuable time troubleshooting a problem, which there might be no solution to.

If you read this and have information on this subject, which you want to share, please create an issue here: [https://github.com/microsoft/navcontainerhelper/issues](https://github.com/microsoft/navcontainerhelper/issues) or email me with your information.

Thanks

**Freddy Kristiansen**  
_Technical Evangelist_
