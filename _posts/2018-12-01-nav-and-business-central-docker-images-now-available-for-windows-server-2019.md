---
layout: post
title: "NAV and Business Central Docker images now available for Windows Server 2019"
date: 2018-12-01 01:21:57
categories: ["Docker"]
tags: ["1809", "ARM", "ltsc2016", "ltsc2019", "NavContainerHelper", "Windows Server 2016", "Windows Server 2019"]
permalink: /2018/12/01/nav-and-business-central-docker-images-now-available-for-windows-server-2019/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

As of today, all NAV and Business Central Docker images are available for Windows Server 2019 as well as for Windows Server 2016!

It doesn’t sound like a big deal, but believe me, it is…:-)

# What does that mean?

As you probably know, Microsoft ships major updates to Windows every 6 months and every 2-3 years a version will be marked as **LTSC** (Long Term Servicing Channel).

For Windows Server, only the LTSC builds will include a UI, which allows you to connect to the Desktop of the Server. All updates between the LTSC releases will only ship as new versions of Windows Server Core, giving you only a Command prompt and/or PowerShell prompt.

You can read more about the Windows Server Semi Annual Channel [here](https://docs.microsoft.com/en-us/windows-server/get-started/semi-annual-channel-overview) and what that means to Windows 10 [here](https://docs.microsoft.com/en-us/windows/deployment/update/waas-overview).

Up until today, **specific** images containing **NAV** or **Business Central** have only been available based on **Windows Server Core 2016**.

**Generic** images (images, which can be used to run **NAV** or **Business Central** if you have an install media or files extracted from another image) are available for most **Semi annual builds** and you can select to use these by using **-useBestContainerOS** on **New-NavContainer**. [This](/2018/10/24/windows-10-and-docker-images-for-business-central-nav/) blog post explains how to do this.

As of today, specific images containing **NAV** or **Business Central** are also available based on **Windows Server Core 2019**.

# Image Tags

Some sample image names for NAV and Business Central:

-   microsoft/dynamics-nav:2018-cu4-dk
-   mcr.microsoft.com/businesscentral/onprem:cu1-de
-   mcr.microsoft.com/businesscentral/sandbox:13.1.25940.26323

If you use **docker pull** and **docker run** on these images, you will get a **ltsc2016** (Windows Server Core 2016) version of the images. That is the same behavior as before today.

If you append **\-ltsc2016**, you will get exactly the same images:

-   microsoft/dynamics-nav:2018-cu4-dk-ltsc2016
-   mcr.microsoft.com/businesscentral/onprem:cu1-de\-ltsc2016
-   mcr.microsoft.com/businesscentral/sandbox:13.1.25940.26323\-ltsc2016

If you append **\-ltsc2019**, you will get the same versions of NAV/Business Central, but based on **ltsc2019** (Windows Server Core 2019)

-   microsoft/dynamics-nav:2018-cu4-dk-ltsc2019
-   mcr.microsoft.com/businesscentral/onprem:cu1-de-ltsc2019
-   mcr.microsoft.com/businesscentral/sandbox:13.1.25940.26323-ltsc2019

Using docker pull and docker run, you need to determine yourself which version of the image to pull and run.

# Compatibility

One reason for using ltsc2019 images is compatibility. It is my experience, that the further away we came from Windows Server 2016, the more issues we would see when people were running Windows 10 and using ltsc2016 images. Almost all issues that I have been unable to resolve for people have been due to Windows 10 1803 running Docker and ltsc2016 images.

# Image Size

Another reason for using ltsc2019 images is image size. After having pulled a number of images, we can inspect the sizes they occupy:

[![](/assets/images/2018/nav-and-business-central-docker-images-now-available-for-windows-server-2019/ac847-images-1.png)](/assets/images/2018/nav-and-business-central-docker-images-now-available-for-windows-server-2019/ac847-images.png)

-   On WindowsServerCore, the ltsc2019 version is 7.02GB smaller.
-   The Generic layer on ltsc2019 is another 2.39GB smaller than the ltsc2016 equivalent, making it 9.41GB smaller in total.
-   Even the W1 layer is smaller. 300MB is saved on ltsc2019, making the image 9.71GB smaller in total.
-   The country layer is only a fraction smaller. Still the combined savings of 9.73GB is massive.

Note that these sizes are after extraction of the compressed images. The download sizes are way smaller, but the difference between the download size of ltsc2016 images and ltsc2019 images are even bigger.

As an example, the download size of the ltsc2016 version of NAV 2018 CU5 DK is 8,159,631,982 bytes and the ltsc2019 equivalent is 3,208,188,273 bytes. The ltsc2016 image is more than 2½ times the size of the ltsc2019 image.

# NavContainerHelper 0.4.2.2

A new version of **NavContainerHelper** will try to help you get the right image for your host. If you specify that you want to use the Danish CU1 version of Business Central on prem, you would specify **mcr.microsoft.com/businesscentral/onprem:cu1-dk** as the imagename. With NavContainerHelper, you should still just specify the imagename like this and NavContainerHelper will then help you determine the best container OS.

The strategy for NavContainerHelper to determine which imagename to use is as follows:

1.  If the imagename doesn’t specify platform (ltscxxxx), then calculate the best imagename (append -ltsc2016 for Windows Server 2016 or -ltsc2019 for Windows Server 2019)
2.  If you have not specified -alwaysPull, use the best imagename or the imagename in that order if they exist on the host
3.  If the images didn’t exist on the host or you have specified -alwaysPull, try to pull the best imagename and if that fails, revert back to the imagename specified

You can also specify -useBestContainerOS if you want to allow NavContainerHelper to use a generic image if that is better, in fact I recommend that you always do that. This allows NavContainerHelper to spin up a better generic image matching your OS in the future as well. -useBestContainerOS won’t do anything if you are already using the best container os for your host.

# ARM templates

When using [http://aka.ms/getbc](http://aka.ms/getbc), [http://aka.ms/getnav](http://aka.ms/getnav) or any of the other ARM templates to spin up Azure VMs with Business Central or NAV, you will now have an option to select your operating system:

[![](/assets/images/2018/nav-and-business-central-docker-images-now-available-for-windows-server-2019/0a4e9-os1-1.png)](/assets/images/2018/nav-and-business-central-docker-images-now-available-for-windows-server-2019/0a4e9-os1.png)

Since the ARM templates uses the NavContainerHelper to create the container, you do not have to specify platform version after the image name. Just specify **mcr.microsoft.com/businesscentral/onprem:cu1-dk** and you will get the version matching the operating system you selected.

The total time it takes to spin up the Azure VM, initialize the VM, pull the image and run the image varies depending on a number of things, but I did a test of spinning up the same image on these 3 host operating systems and the total time for initialization was:

1.  Windows Server 2016 = 42 minutes
2.  Windows Server 2019 = 22 minutes
3.  Windows Server 2019 with Containers = 20 minutes

The primary difference between 2 and 3 is, that #3 has docker pre-installed and the WindowsServerCore 1809 pre-downloaded. This normally works just fine, but I have kept option #2 if for some reason #3 isn’t running the latest Docker or isn’t updated. #1 is kept as an option until we decide to stop building ltsc2016 images.

# Issues?

If you encounter any issues with the images, with NavContainerHelper or with the ARM templates, please file the issues on GitHub:

-   [https://github.com/Microsoft/navcontainerhelper/issues](https://github.com/Microsoft/navcontainerhelper/issues)
-   [https://github.com/Microsoft/nav-arm-templates/issues](https://github.com/Microsoft/nav-arm-templates/issues)
-   [https://github.com/Microsoft/nav-docker/issues](https://github.com/Microsoft/nav-docker/issues)

It is much better than emailing me. An email might disappear in the inbox, a GitHub issue will also notify me with an email and it will be there for other people to see, comment on and learn from.

# What next

First of all – it’s late Friday evening – or rather early Saturday morning. I will enjoy my weekend and then next week I will continue the blog post series on Continuous Integration and Continuous Delivery – part #6 coming up…

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
