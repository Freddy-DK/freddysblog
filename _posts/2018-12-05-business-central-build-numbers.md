---
layout: post
title: "Business Central Build Numbers"
date: 2018-12-05 12:00:07
categories: ["Docker"]
tags: ["bcsandbox", "bcsandbox-master", "Build Numbers", "Docker", "insider builds", "platform version"]
permalink: /2018/12/05/business-central-build-numbers/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

Now and then, I get questions about the build numbers – what do they all mean? I always try to tell people that they don’t really need to know but for some reason, that just makes people more curious. So instead of answering the same question over and over again – here is what I know…

As you might know, I wrote [this blog post](/2018/04/16/which-docker-image-is-the-right-for-you/) a while back, trying to describe that you shouldn’t really care about the build numbers, you should always go for the latest unless instructed otherwise.

But let me try to shed some light over the version numbering of these Docker Images. In order to do so, I will have to talk about public and insider builds separately, but before I do that, let me shortly describe what they are doing in Windows (as our strategy isn’t that different).

# Windows Build numbers

Windows 10 and Windows Server uses the same build numbers, i.e. 10.0… The major and the minor version number doesn’t change and the build numbers are public ([https://www.microsoft.com/en-us/itpro/windows-10/release-information](https://www.microsoft.com/en-us/itpro/windows-10/release-information)), the revision not so much.

Build 17763 is Windows 10 Fall Update 2018, also called 1809 and build 17763 is also Windows Server 2019.

Build 17134 is Windows 10 Spring Update 2018, also called 1803.

etc.

With Business Central we also use the major and the minor version numbers.

# Business Central Build numbers

In October 2018 we shipped Business Central Fall 2018 release.

## Fall 2018 release, update 19, 13.0, rtm

There are a lot of names for the Business Central Fall 2018 release, lets explain them:

-   **Fall 2018 release** – obviously because the release is a major release from fall of 2018.
-   Internally (and sometimes externally) this is called **update 19** because it is month 19 since we started our online services and this the 19th update to our SaaS offerings.
-   Dynamics NAV 2018 was 11.0, Business Central Spring 2018 was 12.0, Business Central Fall 2018 is **13.0**
-   **rtm** is only used for the on-premises release of Business Central as this was the first release of that

Business Central Fall 2018 release is major release **13**, minor release **0** and build number **24630** was the golden build.

Putting these numbers together gives **13.0.24630.0** which indeed is the version number for Business Central Fall 2018. If you want to pull the on-premises Docker image for this release you can use:

```
docker pull mcr.microsoft.com/businesscentral/onprem:13.0.24630.0[-country][-os]
```

where

-   **country** is one of the countries shipped for on-premises: **w1**, **at**, **au**, **be**, **ch**, **cz**, **de**, **dk**, **es**, **fi**, **fr**, **gb**, **is**, **it**, **na**, **nl**, **no**, **nz**, **ru**, **se**. Default is **w1**.
-   **os** is either **ltsc2016** or **ltsc2019**. Default is **ltsc2016**.

The on-premises release is also tagged with **rtm** instead of **13.0.24630.0**.

If you want to pull the sandbox  Docker image for this release you can use:

```
docker pull mcr.microsoft.com/businesscentral/sandbox:13.0.24630.0[-country][-os]
```

where

-   country  is one of the countries shipped online: **w1**, **at**, **au**, **be**, **ca**, **ch**, **de**, **dk**, **ee**, **es**, **fi**, **fr**, **gb**, **hk**, **hr**, **is**, **it**, **jp**, **kr**, **lt**, **lv**, **mx**, **nl**, **no**, **nz**, **pl**, **pt**, **rs**, **se**, **si**, **tw**, **us**. Default is **w1**.
-   **os** is either **ltsc2016** or **ltsc2019**. Default is **ltsc2016**.

For sandbox builds you will also find a country called **base**. Currently, this is the on-premises w1 database running in sandbox mode. This image is build as foundation for all the country versions.

The sandbox release is not tagged with any extra release tags

## Update 20, 13.1, cu1

One month after Business Central Fall 2018 release, in November, we shipped the first minor update. Internally, this is known as update 20. As it is a minor release, the minor part of the version number is updated to 1 and for on-premises we also call this cu1.

The build number for update 20 golden build was 25940 giving us this image for the onprem Docker image:

```
docker pull mcr.microsoft.com/businesscentral/onprem:13.1.25940.0[-country][-os]
```

and this image for the sandbox Docker image:

```
docker pull mcr.microsoft.com/businesscentral/sandbox:13.1.25940.0[-country][-os]
```

**country** and **os** have the same meaning as with the fall release, but we are likely to add new countries at minor updates as well.

## Update 21, 13.2, cu2

In December 2018, we will ship update 21. Another minor release, as cu2 on-premises. I do not have the build numbers for this yet, but you probably get the picture by now.

## What about revisions?

For Business Central online, we also ship revisions or hotfixes. These hotfixes will keep the same Major and Minor version number and also the same build number. It will however add a revision number. The revision number is not a counter starting from zero – it is an internal build number added as revision number to indicate that it is a newer revision.

We do NOT build Docker images of all these revisions as we also don’t roll out all revisions to our Business Central online servers.

For Fall release, we did create Docker images of 3 revisions: 24844, 25242 and 25789. You probably already guessed that this means that the latest sandbox Docker image you can get from update 19/13.0 is:

```
docker pull mcr.microsoft.com/businesscentral/sandbox:13.0.24630.25789[-country][-os]
```

For update 20, we have created Docker images of 2 revisions: 26108 and 26323, meaning that currently the latest sandbox Docker image available is:

```
docker pull mcr.microsoft.com/businesscentral/sandbox:13.1.25940.26323[-country][-os]
```

which is the same version you get by using:

```
docker pull mcr.microsoft.com/businesscentral/sandbox:[country][-][os]
```

These images are ONLY available for sandbox – we will not release revisions of the on-premises Docker images.

# Build numbers of insider builds

If you are in the ReadyToGo program, you have access to Microsoft Collaborate and should have access to our insider builds.

_**Note:** When you signed up for ReadyToGo, you signed a Non Disclosure Agreement (NDA) preventing you from telling about what you see and find in the insider builds. We (Microsoft) want to be the one disclosing our new features, but we would like you to know about it early and be ready with your offerings before we ship. You should be working on finding ways to leverage the innovations in the product instead of blogging about them._

Insider builds come in two flavours: **bcsandbox** and **bcsandbox-master**

-   **bcsandbox** contains functionality, which will ship in our **next minor release**. (While writing this blog post, this is January)
-   **bcsandbox-master** contains functionality, which will ship in our **next major release**. (While writing this blog post, this is Spring 2019)

Now you might wonder why the functionality from bcsandbox won’t be in the December release (13.2 / cu2)? The reason is simple, we branched out for the 13.2 / cu2 release a few days ago and are running final tests and quality assurance before shipping – this does take some time.

Even more confusing is it, that the version number of the current **bcsandbox insider build** (right now) is **13.2.26500.0**. The reason is again simple, we haven’t updated the version number yet in the branch.

And to complete the confusion. The version number of the current **bcsandbox-master insider build** (right now) is **14.0.26496.0**, indicating that **bcsandbox-master** is older than **bcsandbox**. The reason is as usual simple, the build number is a number series shared between all branches (Dynamics NAV 2017 and up) and it just means that the daily bcsandbox-master build was build before the daily bcsandbox build. You might have noticed that the Major and Minor version number has been updated to 14.0 though.

Revision is always **0** as we are not shipping the insider builds and the build number will just be updated.

If you are working on an **app for AppSource**, you should consider setting up **daily builds** checking that your app doesn’t break on the next minor update (**bcsandbox**). You should also consider setting up **weekly builds** to check that your app doesn’t break on the next major update (**bcsandbox-master**). This is one of the next blog posts in the CI/CD series.

Remember that you will always get the latest bcsandbox image by using:

```
docker pull bcinsider.azurecr.io/bcsandbox:[country][-os]
```

and the latest bcsandbox-master by using:

```
docker pull bcinsider.azurecr.io/bcsandbox-master:[country][-os]
```

For the list of countries, available you should check Microsoft Collaborate (until we ship)

# More version numbers

If you run docker inspect on a Docker image, you will under the Labels section under Config find not one, not two, not three, but **four version numbers**. Before getting that question, let me also describe the meaning of these

```
     "osversion": "10.0.17763.134"
     "tag": "0.0.8.0"
     "version": "14.0.26445.0"
     "platform": "13.0.26325.0"
```

**osversion** is the version of the Operating System. Which Windows Server Core version was used to build this image. Read more [here](https://www.microsoft.com/en-us/itpro/windows-10/release-information).

**tag** is the version of the Generic Image used to build this mage. You will find the source for the generic image [here](https://github.com/microsoft/nav-docker) together with information about what’s included in the version.

**version** is the version of Business Central. This version can be used as the tag to pull this image at a later time. This is the version number this blog post is all about.

**platform** is the version number of the platform (version of the Business Central binaries). These are following a different version numbering and the same platform version might be used for several releases of Business Central (especially on insider builds). We are likely to specify on our [monthly developer preview blog posts](https://aka.ms/moderndevtools) which platform version is needed for the features included and we need to know the platform version when you ask questions on [AL issues](https://github.com/Microsoft/al/issues). Note also that when an issue has been fixed, the AL Bot will add a reply to the issue with a text like this:

_“ The fix for this issue has been checked in to the master branch. It will be available in the bcinsider.azurecr.io/bcsandbox-master Docker image starting **from platform build number xxxxx**.”_

This means that you can check whether the fix is in your version of docker image. Unfortunately we cannot give any build version or date when this the fix will be in bcsandbox-master.

# Now you know… – but do you really need the info?

If you are developing a per tenant customization for an online tenant, you should always be using the **latest** public sandbox version – as this is what is being rolled out on our online servers. If your extension doesn’t run on the latest sandbox, it will not run on an online tenant.

If you are developing an app for **AppSource**, you should be developing either on the **latest** public sandbox version or using insider builds. If you are using insider builds you should **follow the insider builds (at least monthly)**.

If you are lifting a Dynamics NAV vertical ([https://docs.microsoft.com/en-us/dynamics365/business-central/dev-itpro/developer/readiness/readiness-embed-apps](https://docs.microsoft.com/en-us/dynamics365/business-central/dev-itpro/developer/readiness/readiness-embed-apps)) you might need to use a **specific build**, but you should be in contact with Microsoft on which build to use.

If you are developing for on-premises, they should use **rtm** or **cu1** tags – not the version numbers and you should use the onprem container, NOT the sandbox containers. Here you also have the option of downloading the DVD.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
