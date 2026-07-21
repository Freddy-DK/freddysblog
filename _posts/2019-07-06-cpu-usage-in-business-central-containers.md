---
layout: post
title: "CPU usage in Business Central Containers"
date: 2019-07-06 10:26:40
categories: ["Docker", "NavContainerHelper"]
tags: ["Containers", "CPU", "Docker", "NavContainerHelper", "new-navcontainer"]
permalink: /2019/07/06/cpu-usage-in-business-central-containers/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

I have had a few people come back and complain about high CPU usage when using Business Central Containers, so I decided to look into this, and see what could be done.

Beside the normal usage of the container, there are two things, which causes CPU usage behind the scenes:

-   The main loop in the container checks the event log every 2nd second for new entries for NAV/BC related entries and dumps those into the container log
-   The container health is checked every 30 seconds and if the container is reported unhealthy 3 consecutive times, the container is restarted

Of these two, the first is by far the one consuming most CPU. I never really noticed this myself (due to a very powerful laptop), but monitoring the CPU on my machine with one running container looks like this:

![cpu-before](/assets/images/2019/cpu-usage-in-business-central-containers/cpu-before.png)

where the lower line indicates the CPU usage of the container, pretty significant.

# Event log – breaking change

The dumping of the event log is the behavior of the generic base image and was introduced even before NavContainerHelper was created in order to have a way to monitor the event log. Later versions of Business Central (especially insider builds) causes a lot more entries in the event log and as a result of this, the container will spend quite a lot of time just dumping the event log.

With NavContainerHelper, you have a function called Get-BCContainerEventLog (or Get-NavContainerEventLog), which will export the event log and open it up in the event viewer on the host. This really minimizes the need for dumping the event log and as a result of this, containers created by NavContainerHelper 0.6.2.5 will by default NOT dump the event log to the container log every 2nd second.

**Note**, that the container will always consume high CPU during creation of the container and during a short period after creating the container. This is normal, but after 1-2 minutes, the CPU usage should be minimal.

Monitoring the CPU on my computer when running one container, created with version 0.6.2.5 of the NavContainerHelper looks like this:

![cpu-1](/assets/images/2019/cpu-usage-in-business-central-containers/cpu-1.png)

It looks like not only the container uses less CPU, but also the host. You see very small spikes every 30 seconds, which is the health check. Again – it is only the lower line, that indicates the container CPU usage, the upper line is everything else running on my computer.

**Note**, you can still get the default container behavior by specifying -dumpEventLog when creating a new container.

# Disabling the Health Check

When running containers in a hosted environment, health checks and automatic restart of containers is important. Note that automatic restart of containers requires docker swarm or other mechanisms that are not out-of-the-box supported by NavContainerHelper (yet).

f you are running containers on your laptop or during build pipelines, you might not need the health check.

You can get rid of some of the time spend by the health check by overriding the CheckHealth.ps1 in the container, but it will still cause the container to launch a PowerShell script, just to tell docker that the container is OK.

With 0.6.2.5 of NavContainerHelper, you can also specify -doNotCheckHealth to minimize the impact of the healthcheck altogether. On my laptop, using this switch causes the CPU monitoring of one container on my laptop to show no activity for the container at all and only show the CPU usage from the host:

![cpu-2](/assets/images/2019/cpu-usage-in-business-central-containers/cpu-2.png)

Currently the implementation of -doNotCheckHealth is to replace the healthcheck setting with an EXIT 0 command. I might change the implementation of doNotCheckHealth if I find a better way to disable health check altogether.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
