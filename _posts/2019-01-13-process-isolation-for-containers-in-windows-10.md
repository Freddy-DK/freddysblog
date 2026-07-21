---
layout: post
title: "Process Isolation for containers in Windows 10"
date: 2019-01-13 08:02:56
categories: ["Docker"]
tags: ["Docker", "Hyperv isolation", "NavContainerHelper", "Process Isolation", "Windows 10"]
permalink: /2019/01/13/process-isolation-for-containers-in-windows-10/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

For a few months we have known that Docker for Windows would get support for process isolation under Windows 10. Arend-Jan Kauffmann explained how to use nightly builds from Docker to get the feature early and test it. I of course jumped on this and have been running a nightly build of Docker since December 6th.

A few days ago the feature was released in the edge release of Docker Desktop.

# No updates?

I tried to check for updates, but this didn’t reveal any new release. I replaced the docker executables with the original files and tried again – still no updates.

I checked the edge release notes: [https://docs.docker.com/docker-for-windows/edge-release-notes/](https://docs.docker.com/docker-for-windows/edge-release-notes/) – no updates.

I did some investigation and found that this was probably caused by Docker for Windows being renamed to Docker Desktop and the version numbering schema has changed.

# This worked for me!

I decided to uninstall docker and install the edge release from here: [https://hub.docker.com/editions/community/docker-ce-desktop-windows](https://hub.docker.com/editions/community/docker-ce-desktop-windows)

Unfortunately this also didn’t go as smooth as expected. I ran into this issue: [https://success.docker.com/article/dockerforwin-install-fails-on-installationmanifestjson](https://success.docker.com/article/dockerforwin-install-fails-on-installationmanifestjson) – but fortunately, I could resolve this by following the resolution in this blog post.

After installing, the About Docker dialog shows that I am running engine 18.09.1, which is the first version supporting the process isolation.

[![](/assets/images/2019/process-isolation-for-containers-in-windows-10/4e045-18.09.11-1.png)](/assets/images/2019/process-isolation-for-containers-in-windows-10/4e045-18.09.11.png)

# NavContainerHelper support (as of today)

NavContainerHelper 0.4.3.0 or newer will default the isolation mode to process when running Windows 10 1809 and Docker 18.09.1 (or a daily build with support for process isolation) and will display this in the output

[![](/assets/images/2019/process-isolation-for-containers-in-windows-10/176bc-output-1.png)](/assets/images/2019/process-isolation-for-containers-in-windows-10/176bc-output.png)

# Proof

Inspecting the processes on the host and in the container reveals that you indeed are running process isolation:

[![](/assets/images/2019/process-isolation-for-containers-in-windows-10/6c833-process-1.png)](/assets/images/2019/process-isolation-for-containers-in-windows-10/6c833-process.png)

# Recommendation

If you are running Windows 10, I recommend you to update to 1809 and update Docker to the 18.09.1 release.

Running NAV/BC containers in process isolation is a **HUGE** win over hyperv isolation.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
