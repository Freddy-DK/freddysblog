---
layout: post
title: "Ready To Go Office Hours, 2020-02-04"
date: 2020-02-05 14:09:57
categories: ["CI/CD"]
tags: ["CD", "CI", "DevOps", "Office Hours"]
permalink: /2020/02/05/ready-to-go-office-hours-2020-02-04/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

Yesterday, I had the pleasure of co-hosting the Office Hours on the Ready To Go Program with Peter Borring and Dmitry Chadayev.

If you are unfamiliar with the Ready To Go program, you should read here: [http://aka.ms/readytogo](http://aka.ms/readytogo) and you can read more about the Office Hours calls here: [http://aka.ms/readytogoofficehours](http://aka.ms/readytogoofficehours)

We have changed the format of the Office Hours calls to be topic based and yesterdays topic was CI/CD. Office Hours are not a deep dive into functionality, but instead we did a short presentation and demo and then we opened up for questions.

A number of the questions was: Will we be able to find the links from the presentation after the call?

We decided that I would include the essence of the presentation in a blog post, so here you go…

# What is CI/CD?

Having read a number of different definitions of CI/CD, I kind of like this one: [https://stackify.com/what-is-cicd-whats-important-and-how-to-get-it-right/](https://stackify.com/what-is-cicd-whats-important-and-how-to-get-it-right/)

Christian Melendez explains about the benefits of CI and CD and why every professional software development company should have these processes implemented. Read it, it is worth while.

# How do I get started?

OK, so far so good, but how to get started?

Even though it seems like an easy question to answer, the answer isn’t straightforward. It really depends on whether you want to do everything yourself or you want to oursource your CI/CD to a partner.

Microsoft provides a number of building blocks you can use to setup CI/CD. These building blocks will make your life much easier, but you actually could setup CI/CD without these

-   Docker Images which all shipped versions of Business Central and all shipped versions of Microsoft Dynamics NAV since 2016.
-   NavContainerHelper with a set of PowerShell functions to assist with creating Containers, compiling applications, running tests etc. etc.
-   Sample yaml files for a pipeline

Microsoft also releases insider builds of next minor and next major release of Business Central. You will get access to these through Microsoft Collaborate ([http://aka.ms/collaborate](http://aka.ms/collaborate)) and through this, you can setup pipelines, which will test your app towards future versions of Business Central to see whether there are any breaking changes planned.

Microsoft doesn’t provide any plug and play toolbox for setting up Continuous Integration and Continuous Delivery for your Dynamics 365 Business Central solutions, but a few partners have created solutions, which you could use or use as inspiration. These are included in the below table as 3rd party tools.

During the presentation I showed a sample repository. You can find that here: [https://dev.azure.com/businesscentralapps/BingMaps](https://dev.azure.com/businesscentralapps/BingMaps).

# Pick your investment level

![investment](/assets/images/2020/ready-to-go-office-hours-2020-02-04/investment.png)

The above table shows that doing things yourself comes with a high price in effort, but less cost, whereas outsourcing obviously is the other way around.

The following list of services and tools are the ones we new of at the time of the presentation. I will happily add other solutions to this list if I hear about them.

# Outsourcing example: NavBaaS

NAVBaaS (Build as a Service) is a SaaS product that supports the entire build process for AL as well as for C/AL without the need to write scripts or deploy/manage infrastructure.

In practice this means that an Azure DevOps task will become available in your Azure DevOps instance, the infrastructure is automatically deployed and you’ll be able to run your first build in +/- 30 minutes.

Unlike other solutions NAVBaaS is a service, meaning that it consists of code (everyone runs on the same/latest version), infrastructure, monitoring and support.

In addition to the out of the box CI solution, you can also get help on a consultancy basis to set-up your CI/CD pipelines or Azure DevOps as a whole.

**Links**  
[https://robberse-it-services.nl/blog/2018/11/07/build-as-a-service-for-dynamics-365-bc-nav-now-released/](https://robberse-it-services.nl/blog/2018/11/07/build-as-a-service-for-dynamics-365-bc-nav-now-released/)

# Tools Example: ALOps

ALOps is an extension in Azure DevOps that enables you to easily create build and release pipelines.

Instead of using generic PowerShell steps, you’re using dedicated AL(Ops) steps to Build or Release your AL Apps.

This even facilitates a visual help by the “DevOps Step Assistant” that guides you through all possible parameters, or you could use the yaml editor with IntelliSense.

All this makes it super easy and stable to set up your own simple or complicated pipeline however you want.

**Links**  
[https://alops.hodor.be/](https://alops.hodor.be/)  
[https://github.com/HodorNV/ALOps](https://github.com/HodorNV/ALOps)  
[https://www.youtube.com/channel/UC8Vpdew5Ca86gGQmiq4Lvrg](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Fwww.youtube.com%2Fchannel%2FUC8Vpdew5Ca86gGQmiq4Lvrg&data=02%7C01%7CFreddy.Kristiansen%40microsoft.com%7C4535ab95f7074fdb459e08d7a664f5ff%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637160824866312246&sdata=1pNKle1b%2Fzx3BxVmmzDKU2H%2BezUuXotzs7RMsi4hBuo%3D&reserved=0)

# Tools Example: AdvaniaGIT

AdvaniaGIT delivers Source Control and Development Environment Management for Microsoft Dynamics NAV and Dynamics 365 Business Central.

For CI/CD there is some assistance for C/AL projects and AL projects is using yaml and the functions shipped in NavContainerHelper.

AdvaniaGIT is OpenSource and available on github through the links below.

AdvaniaGIT has a VS Code add-in for managing and maintaining things directly from VS Code.

**Blog**  
[https://dynamics.is/?tag=AdvaniaGIT](https://dynamics.is/?tag=AdvaniaGIT)

**Link**  
[https://marketplace.visualstudio.com/items?itemName=advaniagit.advaniagit](https://marketplace.visualstudio.com/items?itemName=advaniagit.advaniagit)

# Tools Example: NaverticAl Tasks

Tools and Pipeline samples shared by Kamil Sacek.

Tools were used in Kamil’s TechDays 2019 session (also shared below)

**TechDays 2019 session**  
[https://www.youtube.com/watch?v=](https://www.youtube.com/watch?v=eBytbBNCd3w)[eBytbBNCd3w](https://www.youtube.com/watch?v=eBytbBNCd3w)

**Materials shared**  
[https://dev.azure.com/KineNavTechDays/NavTechDays2019](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Fdev.azure.com%2FKineNavTechDays%2FNavTechDays2019&data=02%7C01%7CFreddy.Kristiansen%40microsoft.com%7C454d62a077f94e9b62bd08d7a9745123%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637164189359703004&sdata=hSxrlyou43mUHXhRx7NxVWBtlzj8cAF2zxHH5sQNZB4%3D&reserved=0)

**Links**  
[https://marketplace.visualstudio.com/items?itemName=Kine.](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Fmarketplace.visualstudio.com%2Fitems%3FitemName%3DKine.naverticaltasks&data=02%7C01%7CFreddy.Kristiansen%40microsoft.com%7C454d62a077f94e9b62bd08d7a9745123%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637164189359682999&sdata=EMUdf6aEYY33tV%2FnGzKpItuDeo4rsSH00X50YgeAK1w%3D&reserved=0)[naverticaltasks](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Fmarketplace.visualstudio.com%2Fitems%3FitemName%3DKine.naverticaltasks&data=02%7C01%7CFreddy.Kristiansen%40microsoft.com%7C454d62a077f94e9b62bd08d7a9745123%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637164189359682999&sdata=EMUdf6aEYY33tV%2FnGzKpItuDeo4rsSH00X50YgeAK1w%3D&reserved=0)

# Do it yourself

In this section we shared some links to blogs, where you can read more from people who tried this and succeeded. Note, that the posts on my blog are the oldest and some details might have changed since

**Blogs**  
[https://freddysblog.com/category/ci-cd](https://freddysblog.com/category/ci-cd)  
[https://never-stop-learning.de](https://never-stop-learning.de/) (Michael Megel)  
[http://www.mynavblog.com](http://www.mynavblog.com/) (Krzysztof Bialowas)

**Hands on lab**  
[http://aka.ms/cicdhol](http://aka.ms/cicdhol)

**DevOps Pipelines Documentation**  
[https://docs.microsoft.com/en-us/azure/devops/pipelines](https://docs.microsoft.com/en-us/azure/devops/pipelines)

**Support for Docker**  
[https://github.com/microsoft/navcontainerhelper/issues](https://github.com/microsoft/navcontainerhelper/issues)

# Q&A

During the Q&A I think we managed to answer most questions, but in some cases the questions involved very deep knowledge about Azure DevOps, which we are unable to answer.

We are very happy to share our experiences (as are a number of our partners), but as stated earlier, we do not provide a plug-and-play CI/CD toolkit, you will need to select how you want to do things and get going…

Thanks

**_Freddy Kristiansen_**  
Technical Evangelist
