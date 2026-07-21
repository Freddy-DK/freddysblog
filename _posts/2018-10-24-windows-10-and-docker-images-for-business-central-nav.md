---
layout: post
title: "Windows 10 and Docker Images for Business Central / NAV"
date: 2018-10-24 07:56:33
categories: ["Docker"]
tags: ["Docker", "Generic Image", "ltsc", "NAV on Docker", "NavContainerHelper", "new-navcontainer", "Windows 10", "Windows Server 1709", "Windows Server 2016", "Windows Server 2019"]
permalink: /2018/10/24/windows-10-and-docker-images-for-business-central-nav/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

It feels like a lot more, but actually it is only about one year ago, we started shipping NAV Container Images on Docker.

Business Central Containers followed shortly after and today a lot of our partners are using Docker and our images for spinning up NAV and Business Central containers to do their daily work.

Most people have great success in running containers, but a few people are struggling. Not because they are doing anything wrong, but because of the version of Windows they are running combined with the hardware they are running it on doesn’t work well with the Container images we are shipping.

# The compatibility matrix

The table below shows what versions of containers are supported on the various versions of Windows, people are running as host.

[![](/assets/images/2018/windows-10-and-docker-images-for-business-central-nav/47854-compat1-1.png)](/assets/images/2018/windows-10-and-docker-images-for-business-central-nav/47854-compat1.png)

Today, all NAV and Business Central containers are built on Windows Server LTSC2016 (first column) and as such should be compatible with all hosts.

And it is – I know for a fact that we have people running all these hosts who successfully are running our images.

But… – I also know, that the further down you go, the bigger the chance of running into issues. In fact, the majority of issues are found in the area I have painted red below, and the green cells are where compatibility is at its best.

[![](/assets/images/2018/windows-10-and-docker-images-for-business-central-nav/9395f-compat2-1.png)](/assets/images/2018/windows-10-and-docker-images-for-business-central-nav/9395f-compat2.png)

So for our images (NAV / Business Central), the best compatible host is Windows Server 2016 and the big problem is really that people running Windows 10 are automatically updated every 6 months and some people are getting into problems with this and the worst thing is, that there often is no way around these problems.

Note, that I am NOT saying that it is a problem in general that Windows is updated regularly – I think that is necessary in the world we live in.

# Our strategy

Our strategy is to build all images on the LTSC (Long Term Servicing Channel). We currently have over 1000 images and we have daily builds from at least 2 branches being published and there is no way we can build all of these for every single supported version of the host.

That strategy has not changed and we could wait until everybody are running 1809 and hope that we won’t run into the same problem towards the next LTSC. It will work, but that seems to be a rather passive strategy and won’t help partners for the next month or two (while waiting for 1809 rollout).

# Generic image to the rescue

As stated in earlier blog posts, the generic image is actually generated for all supported container OS’ and you can pull these images:

-   microsoft/dynamics-nav:generic-ltsc2016
-   microsoft/dynamics-nav:generic-1709
-   microsoft/dynamics-nav:generic-1803

and

microsoft/dynamics-nav:generic, which points to ltsc2016.

The generic image is the layer, which is underneath all NAV or Business Central images and you can (if you have a DVD) run any supported version of NAV or Business Central on Docker using the generic image and the DVD.

# Generic image 0.0.7.0

A few days ago, the generic image was updated to version 0.0.7.0 and the biggest change in this was, that the built-in installer is now able to use a partial DVD image, meaning a DVD image without the prerequisites and a DVD image without the database .BAK file, but instead a .mdf/.ldf file pair (database that can be attached).

Combining this with a new function in NavContainerHelper 0.4.0.0 called Extract-FilesFromNavContainerImage, you probably can see where I am going. You can get a partial DVD image from a container and use that to spin up a container which is more compatible with your host OS.

Update your NavContainerHelper and try:

PS C:\\Windows\\System32> Extract-FilesFromNavContainerImage -imageName microsoft/bcsandbox:us -path c:\\temp\\bcsandbox -extract all
Creating temp container from microsoft/bcsandbox:us and extract necessary files
Extracting Service Tier and WebClient Files
Extracting Windows Client Files
Extracting Configuration packages
Extracting Test Assemblies
Extracting Test Toolkit
Extracting Upgrade Toolkit
Extracting Extensions
Extracting Files from Run folder
Extracting Database Files
Performing cleanup
Removing temp container

You should now have the following in your c:\\temp\\bcsandbox folder:

[![](/assets/images/2018/windows-10-and-docker-images-for-business-central-nav/d86f5-bcsandbox-content-1.png)](/assets/images/2018/windows-10-and-docker-images-for-business-central-nav/d86f5-bcsandbox-content.png)

Which is exactly the files needed to do:

New-NavContainer -accept\_eula -navDvdPath c:\\temp\\bcsandbox -containerName test -auth NavUserPassword -updateHosts

and among the other output you should see:

Using image microsoft/dynamics-nav:generic-1803
Creating Nav container test
NAV Version: 13.0.24623.24800
Generic Tag: 0.0.7.0
Container OS Version: 10.0.17134.345 (1803)
Host OS Version: 10.0.17763.0 (1809)

Note, that it automatically selects the best generic image for your host. I am running 1809 and we do not have a generic image for that yet, but it will automatically pick the latest supported image. Yes – it takes 3-6 minutes extra startup time, but it should make it possible to run containers for a number of people who couldn’t run containers before.

The Extract-FilesFromNavContainer function can also extract the .vsix or the database without the rest of the files by specifying that in the -extract parameter.

Using NavContainerHelper, you can also just specify -UseBestContainerOS to New-NavContainer.

# \-UseBestContainerOS

Try:

New-NavContainer -accept\_eula -imageName microsoft/bcsandbox:us -containerName test -auth NavUserPassword -updateHosts -UseBestContainerOS

and you should see that the NavContainerHelper automatically extracts the needed files into a folder in the containerspecific folder (c:\\programdata\\navcontainerhelper\\extensions\\navdvd) and uses that. This means that the dvd content will automatically be cleaned up when you remove the container.

The output could look like this:

...
Using image microsoft/bcsandbox:us
Creating Nav container test
NAV Version: 13.0.24630.24844-US
Generic Tag: 0.0.6.6
Container OS Version: 10.0.14393.2430 (ltsc2016)
Host OS Version: 10.0.17763.0 (ltsc2019)
A better Container OS exists for your host (1803)
Creating temp container from microsoft/bcsandbox:us and extract necessary files
Extracting Service Tier and WebClient Files
...

UseBestContainerOS will be check whether the host and container OS’ match before running the generic image, meaning that when 1809 ships, it will automatically select the new images and only switch to hyperv isolation if necessary.

# NavContainerHelper 0.4.0.0

NavContainerHelper 0.4.0.0 contains a lot of other enhancements and fixes, especially in the area of CI/CD and can be found [here](https://www.powershellgallery.com/packages/navcontainerhelper/0.4.0.0). CI/CD functions are still work in progress and will be documented later.

You can install the NavContainerhelper by using:

install-module navcontainerhelper -force

or update it to .04.0.0 using:

update-module navcontainerhelper -force

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
