---
layout: post
title: "Trojan Warning when creating Containers…"
date: 2021-11-11 08:25:51
categories: ["Docker"]
tags: ["Docker", "Trojan", "Virus"]
permalink: /2021/11/11/trojan-warning-when-creating-containers/
---

**UPDATE: with defender update 1.353.1128.0 or later, this false positive is no longer.**

With the latest update to Windows 11 and defender, my computer is telling me that I have a Trojan Virus every time I create a container.

Well, I don’t – it is a false positive and there is probably no good workaround possible.

On my machine, running the Windows Server Core image in process isolation like:

docker run -it --isolation process mcr.microsoft.com/windows/servercore:ltsc2022 powershell

Immediately pops up a warning:

![](/assets/images/2021/trojan-warning-when-creating-containers/trojan.png)

The thread detected is a DllSearchOrderHijack

![](/assets/images/2021/trojan-warning-when-creating-containers/trojan2.png)

This file is inside the container in the amsi.dll (part of Windows) and it cannot be remediated as it is inside the container. I know that at least one partner has the same problem on his Windows 11 machine, but I also have an Azure VM, which doesn’t have the problem.

As soon as you remove the container, the Virus is gone again and cannot be detected.

It happens when launching PowerShell.exe – not if I launch cmd.exe, but there is no real way to avoid running PowerShell when we work with Business Central, so what can you do if you get this problem.

## Live with it

You can live with it – the warning pops up again and again, very annoying and you kind of get scared every time.

## Use hyperv

Running containers in hyperv mode causes the container to run isolated and even if there was a virus in the container, it would not be able to do any harm to the host computer.

## Allow the thread

In Windows Security, you can allow the thread, meaning that the defender won’t detect this virus anymore. This is of course a sub-optimal solution as you might get hit by this virus sometime in the future and now you have allowed the Trojan inside.

## Wait for Windows/Defender team to fix the issue

I have reported the issue and I assume that they will fix the issue. I do not however have any timeline as to when we will get this fixed.

Personally, I switched to use hyperv containers for now. If anybody finds a better workaround, let me know.

Thanks

**_Freddy Kristiansen_**  
Technical Evangelist
