---
layout: post
title: "The Microsoft Dynamics NAV Image in the Azure Gallery"
date: 2016-11-05 09:43:22
categories: ["Archive", "Azure"]
tags: ["Azure", "Gallery", "Image", "Microsoft Dynamics NAV", "NAV", "NAV 2016", "NAV 2017"]
permalink: /2016/11/05/the-microsoft-dynamics-nav-image-in-the-public-azure-gallery/
---

By now, we have had a Microsoft Dynamics NAV Image in the public Azure Gallery for 1-2 years and just last week, we shipped the Microsoft Dynamics NAV 2017 image and also the Microsoft Dynamics NAV 2016 Cumulative Update 13 is now available in the gallery.

### What is it?

In essence, it is just a Windows Server 2012R2 with Microsoft Dynamics NAV and SQL Express pre-installed, but there is of course more to it than that.

The Image also contains all 20 DVD images in a folder called **C:\\NAVDVD** and the image contains a series of sample PowerShell scripts in a folder called **C:\\DEMO**, which can help you setup the image for demo purposes and showcase how you can do a lot of the things, that nobody want to do manually.

Deploying the image is done like you deploy any other Virtual Machine on Azure, in the classic management portal or in the new azure portal. Both methods ends up giving you a Virtual Machine running on Azure with your very own NAV Service Tier and all clients installed and you can immediately connect to the machine and start any of the Clients on the machine.

[![vm1](/assets/images/2016/the-microsoft-dynamics-nav-image-in-the-public-azure-gallery/2acc5-vm1-1.png)](/assets/images/2016/the-microsoft-dynamics-nav-image-in-the-public-azure-gallery/2acc5-vm1.png)

This is of course not how you want to be demoing/using Microsoft Dynamics NAV.

You want to connect to the server using the Web Client, you want to use Tablets, Phones, Web Services, PowerBI, etc. etc.

For that, you need a few more steps.

_You need to ensure that the right endpoints have been created when you deploy the machine. You also need to re-configure NAV for SSL with a certificate, open the right ports in the firewall and you need to change the authentication mechanism from Windows to something more fitted for accessing your server from the internet._

**Don’t panic.** you don’t have to do all of this manually. If you remember the PowerShell scripts in the C:\\DEMO folder, which i mentioned earlier – they are intended to help you do all of this – just run the **C:\\DEMO\\Initialize Virtual Machine.ps1** script with PowerShell, answer all the questions and you will be up running.

I will create a couple of blog-posts, describing how to create the virtual machine and make sure that everything is configured the right way. For now, let me just mention the easiest way to get up running:

### The easiest way

Go to: [http://aka.ms/navdemodeploy](http://aka.ms/navdemodeploy)

Login to your Azure Subscription. Select your subscription, resource group, location and name and a VM Admin Password. Leave the remaining fields as their defaults. Accept the terms, Pin to dashboard and select Purchase:

**Note:** Name needs to be globally unique.

[![newportal1](/assets/images/2016/the-microsoft-dynamics-nav-image-in-the-public-azure-gallery/newportal1.png?w=436&h=600)](/assets/images/2016/the-microsoft-dynamics-nav-image-in-the-public-azure-gallery/newportal1.png)

After 5-10 minutes of waiting for:

[![deploying](/assets/images/2016/the-microsoft-dynamics-nav-image-in-the-public-azure-gallery/ad67e-deploying-1.png)](/assets/images/2016/the-microsoft-dynamics-nav-image-in-the-public-azure-gallery/ad67e-deploying.png)

You will have deployed the latest Microsoft Dynamics NAV Demo Environment on Azure. The server should be up running and you can locate the URL for the Landing page at:

[![locatednsname](/assets/images/2016/the-microsoft-dynamics-nav-image-in-the-public-azure-gallery/aee2d-locatednsname-1.png)](/assets/images/2016/the-microsoft-dynamics-nav-image-in-the-public-azure-gallery/aee2d-locatednsname.png)

Navigate to this URL and you will have all the info you need on how to connect to your Microsoft Dynamics NAV 2017 Virtual Machine on Azure:

[![landingpage](/assets/images/2016/the-microsoft-dynamics-nav-image-in-the-public-azure-gallery/landingpage.png?w=1024&h=678)](/assets/images/2016/the-microsoft-dynamics-nav-image-in-the-public-azure-gallery/landingpage.png)

Your Virtual Machine is now available in a lot of different ways:

[![vm2](/assets/images/2016/the-microsoft-dynamics-nav-image-in-the-public-azure-gallery/d5fe1-vm2-1.png)](/assets/images/2016/the-microsoft-dynamics-nav-image-in-the-public-azure-gallery/d5fe1-vm2.png)

**Remember** to follow the instructions on the landing page on how to install the NAV Servers self signed certificate on the device from which you want to connect to the NAV Server, else you will have certificate warnings all the time.

**BTW.** If you want to deploy a NAV 2016 Demo Environment you can use [http://aka.ms/nav2016demodeploy](http://aka.ms/nav2016demodeploy) which will take the latest CU from NAV 2016 and deploy.

### Can I use it for production?

Yes and No.

You of course **cannot** use SQL Express for production, but you can setup a SQL Server on Azure or you can use Azure SQL and then configure your NAV Service Tier to use that server instead.

You also shouldn’t use a self-signed certificate for production customers, you should use a “real” certificate from one of the Trusted Root Certificate Authorities.

You can use the Virtual Machine and you can use the demo scripts for configuring the server, if you validate that they do what you expect yourself. After all – they are only sample scripts.

### Does this have anything to do with Dynamics 365?

No.

This is Microsoft Dynamics NAV, the on-prem product, which also can be hosted in local hosting centers or on Azure. Microsoft is committed to an AND strategy, where we continue shipping Microsoft Dynamics NAV with everything customers and partners have loved for decades – AND – we will be shipping in Dynamics 365 Business Edition.

Microsoft Dynamics NAV will have monthly cumulative updates (and the Azure Image will be updated accordingly), Dynamics 365 Business Edition will follow a different update path – more like Office 365 – constantly evolving and adapting.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
