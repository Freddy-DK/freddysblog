---
layout: post
title: "Debugging “hangs” in Business Central Spring 2019 RTM and CU1"
date: 2019-06-03 14:54:11
categories: ["AL Development", "Docker"]
tags: ["AL", "Debgging", "Docker"]
permalink: /2019/06/03/debugging-hangs-in-business-central-spring-2019-rtm-and-cu1/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

During the last week, we have seen an increasing number of people having issues with debugging in VS Code against a Docker Container running Business Central. At the same time, we had a LOT of people using VS Code and the debugger every day with absolutely no issue.

# The common denominator

As always, when trying to find the root cause of things is to find the common denominator for the people who are having problems – and they were in all areas. Some were using Windows 10 1609, some 1809 or 1903. Some where using Windows Server 2019. Some process isolation, some hyperv isolation. Some Windows Authentication, some NavUserPassword – really not easy to find any common ground.

In the end, Torben Wind Meyhoff and Kalman Beres managed to find the common denominator – in all situations, the container had only one CPU core (2 logical processors) assigned and with this information immediately turned focus on the thread throttling feature.

# Thread Throttling

In Business Central Spring 2019 release, we advanced the thread throttling mechanism. It allows us to lower the priority on threads in order to allow other threads to complete faster. This new prioritization mechanism, should not be enabled when you run with only 2 logical processors and we will fix the platform to use the prior thread throttling if you have less than 4 logical processors available. The prior version of thread throttling mechanism, did not prioritize between the threads and did not have the issue.

# In the meantime

While waiting for the fix, you have a few options:

1.  You can disable Thread Throttling
2.  You can make sure that you have 4 logical processors

# Disable Thread Throttling

You need to set EnableThreadThrottling and EnablePrioritizedThreadThrottling to false in CustomSettings.config.

When using New-NavContainer, you can specify additional native Docker paramaters using additionalParameters. Using the **–env** docker parameter, you can transfer parameters like CustomNavSettings to the Business Central Container to set custom settings for the Service Tier:

```
 -additionalParameters @("--env CustomNavSettings=EnableThreadThrottling=False,EnablePrioritizedThreadThrottling=False")
```

During startup of the Container, it should display:

```
Modifying Service Tier Config File with settings from environment variable
Creating EnableThreadThrottling and setting it to False
Creating EnablePrioritizedThreadThrottling and setting it to False
Starting Service Tier
```

# Assign 4 logical processors

The other way of fixing the issue is to allow the container to use more logical processors. This will of course only work if the host has 4 or more logical processors.

When using New-NavContainer, you can specify additional native Docker paramaters using additionalParameters. Using the **–cpu-count** you can set the number of logical processors available to the container:

```
-additionalParameters @('--cpu-count 4')
```

You will not see this during startup of the container, but if you want to see how many logical processors your container has available you can issue this command in the Cmd prompt:

```
C:\>docker exec <containername> wmic cpu get NumberOfLogicalProcessors
NumberOfLogicalProcessors
4
```

# NavContainerHelper

I am considering to help partners using NavContainerHelper to automatically disable Thread Throttling if you have less than 4 logical processors (this will be the behavior in next version anyway).

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
