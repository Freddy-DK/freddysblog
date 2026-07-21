---
layout: post
title: "What is docker? What are Containers? What can I use containers for? (NAV on Docker #3)"
date: 2017-10-31 15:22:38
categories: ["Docker", "Not Archived"]
tags: ["Containers", "Docker", "isolation", "Microsoft Dynamics NAV", "NAV", "NAV on Docker"]
permalink: /2017/10/31/what-is-docker-what-are-containers/
---

![](/assets/images/2017/what-is-docker-what-are-containers/6f438-docker.png)

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

You have probably heard that all released Microsoft Dynamics NAV versions since NAV 2016RTM now are available as Docker images on the public [Docker Hub](https://hub.docker.com/r/microsoft/dynamics-nav/).

**If not – then you have now!**

But what does that mean? What is Docker and what are Containers?

Lets start with the easy question

# What is Docker?

Docker is the company driving the container movement and the only container platform provider to address every application across the hybrid cloud. It is also the name of a software handling Containers on Linux and the name of the software which handles containers on Windows.

# What are Containers?

![](/assets/images/2017/what-is-docker-what-are-containers/zx81-300x221.png)To explain containers I will start by moving back in time, back in the days in 1980 when I got my first computer (Yes, I am THAT old).

It was a Sinclair ZX 81, and it has 1Kb of RAM and 1Kb of ROM and it was an 8 bit single threaded, single processor, single kernel machine with a Z80 processor. The 1024 bytes of memory was from address 0000 – 03FF – and only one app was able to on this machine. The app owned the memory, the screen, the CPU, IO ports, etc. etc. – and then the computer was instant on (life was much easier back then).

_Stay with me here…_

[![](/assets/images/2017/what-is-docker-what-are-containers/62788-vmm-1.png)](/assets/images/2017/what-is-docker-what-are-containers/62788-vmm.png)Since then, computers have gotten more and more complex and can run multiple apps at the same time – but if you think about it, every time you add an app, there are chances of conflicts. Yes we have a virtual memory manager so that one app couldn’t access memory from other apps, but we really didn’t get much further. If two apps wants to listen on port 80 on the public network adapter of a machine – there is a conflict and it is up the every individual app to be able to configure all kinds of things.

If two apps are having the same name and utilizing the same folder structure on the hard drive, there is a conflict and if 2 apps are using different versions of the same DLL, there might be problems. We have tried to fix this in various ways, by allowing DLL’s with different versions to run side by side and to setup port sharing services etc. etc. – but the fundamental problem still exists.

When you are running an app on a computer, you stand the risk that it will conflict with everything else on the machine.

_We are getting there…_

With a container you achieve a higher level of isolation. Each container has its own file system, its own registry, its own network layer with its own published ports, and of course its own memory space – kind of like a virtual machine, just without the fat guest operating system. Kind of like my good old Sinclair ZX 81:-). The container is also almost instant on, when you run a container it doesn’t have to perform all kind of initialization’s like a normal computer. It knows that everything is right where it was a moment ago.

You can pull multiple different versions of containers, you can clone containers and you can run multiple instances of containers side by side and every container will think that it is alone in the world. Every container can listen on port 80 on the container but you will of course run into a conflict if you ask two container to expose the same port publicly on the host.

# 2 kinds of isolation when running containers!

![](/assets/images/2017/what-is-docker-what-are-containers/45b91-container-morph.gif)As if this wasn’t enough, I also need to tell you can run a container in two different ways:

**Windows Server containers** – multiple container instances can run concurrently on a host, with isolation provided through namespace, resource control, and process isolation technologies. Windows Server containers share the same kernel with the host, as well as each other. A Windows Server Container will only allocate memory as needed and you can specify the max. amount of memory every container can allocate. By default every Windows Server container can allocate all memory available on the host (no max.)

**Hyper-V containers** – multiple container instances can run concurrently on a host; however, each container runs inside of a special virtual machine. This provides kernel level isolation between each Hyper-V container and the container host. A Hyper-V container allocates the amount of memory given at startup and cannot exceed this amount. By default Windows 10 will give each container 1Gb of memory which isn’t enough for the NAV Container.

Windows 10 currently only supports Hyper-V containers and Windows Server 2016 supports both. Specify **–isolation=hyperv** to the docker run command on Windows Server 2016 to run the container as a Hyper-V container.

The image on the right hand side really refers to Windows Server containers and I recommend you to run Windows Server 2016 if you want to run multiple containers on your host.

# What can you use containers for?

Well, that is a good question and I really would like to ask you that question? I have a lot of use case scenarios (requirements from partners just like you) and we have built out the images to support these, but please follow the blog posts and please share with us what you would like to use the containers for.

What we have provided is basically the ability to configure and run NAV in a container in a homogeneous setup, which causes a lot of infrastructure issues to disappear and allow you to run many containers side by side without any issues.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
