---
layout: post
title: "The 10 command(ment)s of Docker (NAV on Docker #6)"
date: 2017-11-03 18:10:59
categories: ["Docker", "Not Archived"]
tags: ["Docker", "NAV", "NAV on Docker", "NavContainerHelper"]
permalink: /2017/11/03/10-commands-of-docker/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

I recommend that you read [this blog post](/2017/11/03/multiple-ways-to-run-a-nav-on-docker-image/) before reading this.

In this blog post I will describe the 10 docker commands I use most frequently and what I use them for. The commands can be executed in a Command Prompt, PowerShell or PowerShell ISE on a machine with Docker installed. In any case, you need to be running as administrator.

# docker images

docker images will show a list of the current images available on the host:  
[![](/assets/images/2017/10-commands-of-docker/51e24-dockerimages-1.png)](/assets/images/2017/10-commands-of-docker/51e24-dockerimages.png)

In this list, you can see, that I only have one image on my machine, which is the w1 version of the devpreview.

```
microsoft/dynamics-nav:devpreview
```

As you probably know, docker is using a layering technology, meaning that when I have this image I also have all the components for the images, on which this relies. I just haven’t combined the tags of these versions. If I perform a docker pull of microsoft/windowsservercore (which is one of the base images of the NAV on Docker image) you will see that Docker says that the layers already exists and now we have another image in the list:  
[![](/assets/images/2017/10-commands-of-docker/033ac-dockerimages2-1.png)](/assets/images/2017/10-commands-of-docker/033ac-dockerimages2.png)

# docker pull

docker pull is used to pull images from a docker repository. Pulling from the public Docker Hub (for NAV on Docker images, that is images starting with microsoft/dynamics-nav) you do not need credentials to pull an image.

If you are trying to pull an image from a private registry (like navdocker.azurecr.io), you will need to authenticate to that Docker registry before being able to pull images.

If you are pulling an image, where you already have all layers (from other images), then the pull will just map the tags with the layers and you are done. This is what happened in the sample provided under docker images.

If you however pull the financials US version of the developer preview, you will see that Docker reuses a number of layers and proceeds by downloading the missing pieces. The financials US version is of course built on top of the W1 version which we already had:  
[![](/assets/images/2017/10-commands-of-docker/4ea34-dockerpull-1.png)](/assets/images/2017/10-commands-of-docker/4ea34-dockerpull.png)

# docker rmi

docker rmi will remove an image:  
[![](/assets/images/2017/10-commands-of-docker/dc42e-dockerrmi-1.png)](/assets/images/2017/10-commands-of-docker/dc42e-dockerrmi.png)

Note, that you specify the image id as a parameter and it is only necessary to specify as many characters until the id is unambiguous (docker rmi 5 would have been enough in this example).

Note, that Docker only actually deletes the layers that are unused. Like when you pull the windowsservercore and it discovers that all layers already exists, then deleting the windowsservercore will only remove the tag, but not delete any layers as they are still used by the devpreview image.

# docker run

docker run is described in a little more detail in [this blog post](/2017/11/03/multiple-ways-to-run-a-nav-on-docker-image/).

One important thing to notice is, that docker run automatically pulls the image if the image doesn’t exist. If the image already exists, the image is reused. That means that even if you remove a container and issue a new docker run microsoft/dynamics-nav:devpreview after the next update is available, it will NOT download the new version. You have to issue a docker pull specifically in order to pull a new version of an image.

[![](/assets/images/2017/10-commands-of-docker/8bfd9-dockerrun3-1.png)](/assets/images/2017/10-commands-of-docker/8bfd9-dockerrun3.png)

# docker ps

docker ps shows currently running containers.

docker ps -a shows all containers on the host (running or exited).

Lets say you have issued a docker run and forgot to set the accept\_eula to Y (I am sure you will never make that mistake), then you might see something like this:  
[![](/assets/images/2017/10-commands-of-docker/5fdc8-dockerps-1.png)](/assets/images/2017/10-commands-of-docker/5fdc8-dockerps.png)

The f5 container (pensive\_roentgen) exited immediately with an error specifying that you need to accept the EULA, but the container still exists. It is just in an exited state. Docker ps without the -a would not include the f5 container.

# docker rm

docker rm will remove a container. If the container is exited (stopped), then you can just issue the command directly. If the container is running you need to specify -f to docker rm:  
[![](/assets/images/2017/10-commands-of-docker/4b17a-dockerrm-1.png)](/assets/images/2017/10-commands-of-docker/4b17a-dockerrm.png)

# docker inspect

docker inspect is used to inspect a container or an image:  
[![](/assets/images/2017/10-commands-of-docker/a6ff9-dockerinspect1-1.png)](/assets/images/2017/10-commands-of-docker/a6ff9-dockerinspect1.png)

When inspecting an image, one of the interesting pieces are the Labels:

```
"Labels": {
    "country": "W1",
    "created": "201710262345",
    "cu": "october",
    "eula": "https://go.microsoft.com/fwlink/?linkid=861843",
    "legal": "http://go.microsoft.com/fwlink/?LinkId=837447",
    "maintainer": "Freddy Kristiansen",
    "nav": "devpreview",
    "osversion": "10.0.14393.1770",
    "tag": "0.0.3.1",
    "version": "11.0.18712.0"
 }
```

Here you will find some useful information like:

-   **country** is the localization of NAV (if this starts with fin, then you are running a Financials database)
-   **osversion** is the version of the microsoft/windowsservercore which was used to build this image
-   **tag** is the tag of the generic nav image (the base image for all NAV images)
-   **version** is the version number of the NAV executables
-   **nav** and **cu** is the version and cumulative update of the image
-   **eula** is the NAV on Docker EULA
-   **legal** is the legal link for the installed version of Microsoft Dynamics NAV

If you inspect a container instead of an image, you will have other things to inspect like healthcheck, environment variables and network:

```
"Networks": {
    "nat": {
    "IPAMConfig": null,
    "Links": null,
    "Aliases": null,
    "NetworkID": "cd2b370039b6d44e35db95971ce7f120eed0b3509bdb6d3b1e1c0c36f8ea6a9d",
    "EndpointID": "f094d82a29797ef17c3537b6047232981512ca6cfc4417a9759f6bc9cb16db57",
    "Gateway": "172.19.144.1",
    "IPAddress": "172.19.154.78",
    "IPPrefixLen": 16,
    "IPv6Gateway": "",
    "GlobalIPv6Address": "",
    "GlobalIPv6PrefixLen": 0,
    "MacAddress": "00:15:5d:f4:8d:56",
    "DriverOpts": null
}
```

**Note:** every issue created on the github repository: [http://www.github.com/microsoft/nav-docker](http://www.github.com/microsoft/nav-docker) should be accompanied with a docker inspect output of the container not working.  If it isn’t included, it is probably the first things you will be asked for.

**Note also:** if you have provided your password in clear text to docker run (insecure), then the password will be visible in docker inspect. If you have provided your password through securepassword (which is what navcontainerhelper always does), then you will only see an encrypted string without any chance of getting to the key to decrypt it.

# docker logs

If docker inspect is the first thing you will be asked for, then docker logs is the second.

docker logs will write the logs from your container – much like the startup text. If you are running your containers with -d (detached), docker logs is the only way to inspect the installation phase/output of your container:  
[![](/assets/images/2017/10-commands-of-docker/70c46-dockerlogs-1.png)](/assets/images/2017/10-commands-of-docker/70c46-dockerlogs.png)

**Note:** docker logs will never display the password you provided whether you did so securely or insecurely.

# docker start/stop/restart

So these ones are very very simple and I almost do not want to describe them, but what the heck…

-   **docker stop**  stops a running container
-   **docker start**  starts a stopped container
-   **docker restart**  restarts a running container

Stopping a container means that it will stop using CPU and memory, but will still reside on disk, ready to be started again.

# docker commit

docker commit is by far the easiest way to create your own docker image.

What it does is, to save the current state of a stopped container as a new image – and now you can use docker run to run the new image.

Start out by inspecting the running images and stop the one you want to clone:  
[![](/assets/images/2017/10-commands-of-docker/abe5b-dockercommit1-1.png)](/assets/images/2017/10-commands-of-docker/abe5b-dockercommit1.png)

Now simply use docker commit, to create a new image called myimage:  
[![](/assets/images/2017/10-commands-of-docker/1ad25-dockercommit21-e1509732501208-1.png)](/assets/images/2017/10-commands-of-docker/1ad25-dockercommit21-e1509732501208.png)

and last, but not least, run the new image:  
[![](/assets/images/2017/10-commands-of-docker/8b9dd-dockercommit3-1.png)](/assets/images/2017/10-commands-of-docker/8b9dd-dockercommit3.png)

As you can see, no need to set the Accept\_eula environment variable, as you have already accepted the eula when running the container in the first place.

Now, you might be wondering why it has to do so many things on startup? Create NAV Web Server instance – wasn’t that already created?

Well yes, it was – but the NAV on Docker image will discover that the hostname changed, and therefore, it will rerun all the setup routines which is depending on the hostname. If instead you start the NAV on Docker image without WebClient and without Http file download site, then your image will start in approx. 10 seconds:  
[![](/assets/images/2017/10-commands-of-docker/093fd-dockercommit4-1.png)](/assets/images/2017/10-commands-of-docker/093fd-dockercommit4.png)

Don’t blink or you will miss the match…:-)

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
