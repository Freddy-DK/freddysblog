---
layout: post
title: "Changing the way you run Business Central in docker…"
date: 2020-06-25 11:20:12
categories: ["Docker"]
tags: ["Artifacts", "Docker", "PowerShell"]
permalink: /2020/06/25/changing-the-way-you-run-business-central-in-docker/
---

**Update 2021/2/10:** BcContainerHelper has replaced NavContainerHelper. This blog post still reference NavContainerHelper, which is outdated.

Just as you thought you were getting the hang of running Business Central in docker, then you see this title! Your first thought is probably how big the changes are to your pipelines and to your scripts and how much time you will have to perform these changes.

# Easy now!

Nothing will have to change overnight – everything that worked yesterday should still work and you will just be able to run Business Central in docker in a new way starting today…

# What?

The big change is, that Microsoft will stop producing docker images and instead publish builds as artifacts, which can be used together with the generic docker image to run the image you want.

# Why?

There are a number of reasons why this change is being done. If we look at how Business Central images are built, then the generic image is SQL Express, IIS and some PowerShell scripts added to dotnet/framework/runtime.

On top of that we add a w1 layer for every version of Business Central and on top of that a number of local images (20 for onprem).

![](/assets/images/2020/changing-the-way-you-run-business-central-in-docker/layers.png)

When running an image like this, it works best if you are using the same OS Version for windows/servercore and he host (see more here: [https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility)). In fact we have had multiple problems with people running different versions causing all kinds of issues. For some time, we tried to build the images on the version of windows/servercore giving the least amount of issues, but our MS security team is telling us, that we cannot use old and un-patched versioins of windows/servercore – it is a security risk, and I cannot really argue against that.

Even worse – I know of partners who are trying to not update their host machines in order to avoid compatibility problems with new updates.

The fact sheet for explaining the support for Windows OS’ can be found here: [https://support.microsoft.com/en-us/help/13853/windows-lifecycle-fact-sheet](https://support.microsoft.com/en-us/help/13853/windows-lifecycle-fact-sheet) – ltsc builds are supported for 10 years, sac builds for 18 months. Every supported build will get new cumulative updates every month. While writing this, there are 8 supported versions of Windows supported for Docker (host and container OS).

Every time a new windows/servercore is shipped (8 every month), the dotnet framework team ships a new image based on that windows/servercore, and every time a new dotnet/framework/runtime image is published we publish a new generic image (the layer with SQL Express, IIS and PS Scripts).

_**That is all goodness, that is manageable.**_

The problem comes if we are to continue the rebuild up the chain. While writing this, there are 176 w1 on premises images out there and there are 20 localizations on top of these – giving a total of 3500+ images and new images are added every month (and then we haven’t even counted the sandbox images and the daily builds). If we were to build these for every new version of the generic layer, every month – we would have to build 28.000+ images every month and publish these (that would be ~280 Terabytes of images every month) and a lot of these images would probably never be used.

So, lets go back to where things were manageable – we build the generic images and use these. This is where I added the -useBestContainerOS parameter. This parameter worked like this:

![](/assets/images/2020/changing-the-way-you-run-business-central-in-docker/usebest.png)

1.  You pull the image you want to run
2.  You extract the artifact from that image (binaries, database, apps etc.)
3.  You pull the generic docker image
4.  You share the artifacts with the container
5.  You run the generic image with these artifacts

That works, but isn’t really efficient.

The specific docker image is ~10Gb (~15Gb for ltsc2016). The Generic image is ~6Gb and since the generic is build on a different windows/servercore – you always have to download both.

_**You are actually downloading ~10Gb of docker image for the soul purpose of extracting artifacts (which are around 1Gb in size**_)

Adding to that, docker images are not the best artifact cache and cleanup etc. is cumbersome.

So in the future, we will be publishing artifacts instead of docker images. Artifacts will be available through a CDN for very fast download and is fully integrated in NavContainerHelper and the generic docker image, the process will instead look like this:

![](/assets/images/2020/changing-the-way-you-run-business-central-in-docker/artifacts.png)

1.  Pull the generic image
2.  Run the image with a parameter specifying where to find the artifacts
3.  Download the artifacts during run

Using the ContainerHelper, it will actually download the artifacts beforehand and cache them on the host computer.

## How?

The very first thing you need to do is, to update the NavContainerHelper to 0.7.0.7 or later. The next thing you need to do is to find the artifactUrl. There is a function for this:

```
$artifactUrl = Get-BCArtifactUrl -version 16.2 -country us -select Latest
```

This will give you the artifactUrl for the latest 16.2 Business Central sandbox artifacts. This artiactUrl can be passed directly to New-BcContainer instead of the imagename:

The running container should work EXACTLY like a running container based on an image, it just takes a little longer to start.

# When can I try this?

You can try this out today. The generic images are out there and NavContainerHelper is supporting this. All existing on-premises artifacts have been published (-type onprem on Get-BcArtifacts) and a number of the later sandbox artifacts.

Do play around with Get-BcArtifactUrl and Get-NavArtifactUrl to get artifactUrls for the various versions of Business Central and NAV. Try to run them and see if everything works as it used to do – it should.

If you experience issues, please file issues here: [https://github.com/microsoft/navcontainerhelper/issues](https://github.com/microsoft/navcontainerhelper/issues) and remember to pass the full script and the full output of the script – that will help me identify why things didn’t work as expected.

# When do we switch over?

If everything goes as planned, July update will be the last image, which gets published as a docker image. After this, the docker images will stay for a while to ensure that everybody using them have had a chance to switch.

Insider images will also become available as artifacts and will be published more frequently, faster and more reliable than docker images. Insider images stop being published as docker images in July as well.

# What now?

As you can imagine, a lot of documentation will need to change. A lot of blogs and how-tos will need to be updated, CI/CD documentation etc. etc. – I will work on that for the next weeks.

I will also monitor the NavContainerHelper issues list to see if things are running as expected.

and… – the next blog posts will go more into detail about the new functions and how things really works.

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
