---
layout: post
title: "devpreview builds now also available on the Docker Hub"
date: 2017-10-31 12:25:52
categories: ["Docker", "Not Archived"]
tags: ["Developer Preview", "devpreview", "Docker", "NAV on Docker"]
permalink: /2017/10/31/devpreview-builds-now-also-available-on-the-docker-hub/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

Saturday we posted all NAV 2016 and NAV 2017 public builds to the Docker Hub.

Today the [October update of the developer preview](http://aka.ms/moderndevtools) is also on the Docker Hub to run locally, meaning that you now can choose between two options for the developer preview.

1.  [http://aka.ms/navdeveloperpreview](http://aka.ms/navdeveloperpreview) (Azure VM with NAV on Docker inside)
2.  docker run microsoft/dynamics-nav:devpreview-finus

The Image Tags available are found on the [Docker Hub](https://hub.docker.com/r/microsoft/dynamics-nav/tags/):  
[![](/assets/images/2017/devpreview-builds-now-also-available-on-the-docker-hub/eb50d-devpreview-1.png)](/assets/images/2017/devpreview-builds-now-also-available-on-the-docker-hub/eb50d-devpreview.png)

# Why so many tags?

There are only 5 images – why so many tags?

The images are really just following the tag strategy laid out [here](https://hub.docker.com/r/microsoft/dynamics-nav/):

-   \[version\[-cu\]\[-country\]\]

Where

-   **Version** is devpreview – if you omit version you will get 2017 which is the newest released version
-   **cu** is october (omit to get newest)
-   **country** is w1, finus, finca, fingb, findk (omit to get w1)

So

-   **docker pull microsoft/dynamics-nav:devpreview-finus** gives you the latest devpreview with financials US localization.
-   **docker pull microsoft/dynamics-nav:devpreview-october-findk** gives you the october release of devpreview with financials DK localization.
-   **docker pull microsoft/dynamics-nav:devpreview** gives you the latest devpreview with W1 localization
-   **docker pull microsoft/dynamics-nav** still gives you NAV 2017 latest CU, w1 localization

So the number of tags is really to be able to always address a specific image and also have a way of always asking for the latest.

# Where do I find the AL Extensions for VS Code

You can grab it from the container, then you know it is the right version, which matches the version of the service tier inside the container:

[![](/assets/images/2017/devpreview-builds-now-also-available-on-the-docker-hub/vsix.png?w=982&h=606)](/assets/images/2017/devpreview-builds-now-also-available-on-the-docker-hub/vsix.png)

You will also find the values to use for Development server and Development ServerInstance in launch.json right above the .vsix file.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
