---
layout: post
title: "Please check your version of Docker!"
date: 2019-12-03 07:34:52
categories: ["Docker", "NavContainerHelper"]
tags: ["Docker", "NavContainerHelper", "Security", "TLS"]
permalink: /2019/12/03/please-update-your-version-of-docker/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

In Microsoft, we are constantly in search of ways to improve security for our customers. Customers must feel safe when using our services and leaving their precious data in our hands. Sometimes this requires our customers and partners to update client software and this blog post is a warning about just that.

# Transport Layer Security (TLS v1.2)

## Within the next month…

## Before December 31st 2019…

Microsoft will harden security on our docker registries (both **mcr.microsoft.com** and **bcinsider.azurecr.io**) and set the required transport level security to v1.2.

This means that all computers running a version of Docker, which doesn’t support TLS v1.2, no longer will be able to pull any Microsoft official images from Microsoft registries.

# Only docker 18.03.0 or above

Open a command prompt and run **docker version**.

Client: Docker Engine - Community
 Version: 19.03.5
 API version: 1.40
 Go version: go1.12.12
 Git commit: 633a0ea
 Built: Wed Nov 13 07:22:37 2019
 OS/Arch: windows/amd64
 Experimental: false

Server: Docker Engine - Community
 Engine:
  Version: 19.03.5
  API version: 1.40 (minimum version 1.24)
  Go version: go1.12.12
  Git commit: 633a0ea
  Built: Wed Nov 13 07:36:50 2019
  OS/Arch: windows/amd64
  Experimental: false

if the docker client or server version is **lower than 18.03.0** – **you need to update**.

# Warning

NavContainerHelper also displays the docker version when creating containers:

NavContainerHelper is version 0.6.4.20
NavContainerHelper is running as administrator
Host is Microsoft Windows 10 Pro - ltsc2019
Docker Client Version is 19.03.5
Docker Server Version is 19.03.5

You should be able to see this in the build output of the agents used by your CI/CD pipelines. With the next version of NavContainerHelper, you will also see a warning if your version of docker is prior to 18.03.0:

WARNING: Microsoft container registries will switch to TLS v1.2 very soon and your version of Docker does not support this. You should install a new version of docker asap (version 18.03.0 or later)

This will be visible in the output of your build agent in CI/CD scenarios when using NavContainerHelper.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
