---
layout: post
title: "The ARM Templates now supports artifacts… (and images)"
date: 2020-07-02 12:05:58
categories: ["Azure", "Demo Environments", "NavContainerHelper"]
tags: ["ARM", "Artifacts", "Docker", "Get-BcArtifactUrl", "NavContainerHelper"]
permalink: /2020/07/02/the-arm-templates-now-supports-artifacts-and-images/
---

**Update 2021/2/10:** BcContainerHelper has replaced NavContainerHelper. This blog post still reference NavContainerHelper, which is outdated.

I know a lot of partners and customers are using the Business Central ARM templates to create an Azure VM, which runs a specific version of Business Central (or NAV). This blog post describes what changed.

# Images still supported

First of all, the ARM templates still supports docker images. This means that if you are provisioning Azure VMs using the ARM templates through a PowerShell script, that script should still run.

By default, the artifactUrl is specified to return the latest version and the docker image setting is blank. If you however specify a docker image in the template, it will ignore the artifactUrl setting (hence allowing your script to still run).

## The ARM template short URLs

There are currently 6 short Urls for creating Azure VMs with Business Central or NAV. In all templates you can replace the artifactUrl to a different artifact, but by default, the shortUrls will give you the following:

<table><tbody><tr><td><a href="http://aka.ms/getbc" rel="nofollow">http://aka.ms/getbc</a></td><td>Creates an Azure VM with the latest on-premises W1 build of Business Central (simplified template)</td></tr><tr><td><a href="http://aka.ms/getbcext" rel="nofollow">http://aka.ms/getbcext</a></td><td>Creates an Azure VM with the latest on-premises W1 build of Business Central (extended template)</td></tr><tr><td><a href="http://aka.ms/getnav" rel="nofollow">http://aka.ms/getnav</a></td><td>Creates an Azure VM with the latest W1 build of NAV (simplified template)</td></tr><tr><td><a href="http://aka.ms/getnavext" rel="nofollow">http://aka.ms/getnavext</a></td><td>Creates an Azure VM with the latest W1 build of NAV (extended template)</td></tr><tr><td><a href="http://aka.ms/bcsandboxazure" rel="nofollow">http://aka.ms/bcsandboxazure</a></td><td>Creates an Azure VM with the latest US build of Business Central (sandbox)</td></tr><tr><td><a href="http://aka.ms/getnavworkshopvms" rel="nofollow">http://aka.ms/getnavworkshopvms</a></td><td>Creates a number of Azure VMs with the latest US build of Business Central (sandbox)</td></tr></tbody></table>

As a special option, you can add the number 2 to all Urls to get an insider version (the dev branch). While writing this blog post, the 2 branches are identical and I recommend that you use the Urls above.

# The ArtifactUrl setting

When you launch any of those short Urls, you will be asked to login to the your Azure Subscription (i.e. the subscription which pays the cost of your Azure VM). After this, you will see a template, which you need to fill out. see this blog post for more general information about this: [https://freddysblog.com/2019/07/26/the-arm-templates-for-dynamics-365-business-central-and-microsoft-dynamics-nav/](/2019/07/26/the-arm-templates-for-dynamics-365-business-central-and-microsoft-dynamics-nav/)

The only new field is artifactUrl and it is defaulted as described under Get-BcArtifactUrl in this blog post: [https://freddysblog.com/2020/06/27/ci-cd-and-artifacts/](/2020/06/27/ci-cd-and-artifacts/)

![](/assets/images/2020/the-arm-templates-now-supports-artifacts-and-images/artifacturl.png)

Again – if you enter a docker image, the artifactUrl is ignored. You can either enter a full artifact Url (returned by Get-BcArtifactUrl) or you can enter the parameters for the function (like above) separated by a slash like:

**storageAccount/type/version/country/select/sasToken**

This means that this value:

**[https://bcartifacts.azureedge.net/onprem/14.9.39327.0/dk](https://bcartifacts.azureedge.net/onprem/14.9.39327.0/dk)**

and this:

**bcartifacts/onprem/14.9.39327.0/dk**

Yields the same Business Central version (but you probably already guessed that)

# Internally

If you login to the Azure VM to see what is going on, you will discover that the ContainerHelper is using the mode, where is generates the image and saves it locally on the machine before running it:

![](/assets/images/2020/the-arm-templates-now-supports-artifacts-and-images/images.png)

The image is named mybc:<version>-<country>

# The github repository

All the ARM templates are open source and you are very welcome to clone and change them for your own needs. The github repository is here: [https://github.com/microsoft/nav-arm-templates](https://github.com/microsoft/nav-arm-templates).

The short Urls are calling an Azure Function to embed parameters into the template. You can reuse that, or you can create a direct Url to deploy your template. You will need a URL to your template json file, like:

[https://raw.githubusercontent.com/microsoft/nav-arm-templates/master/getbc.json](https://raw.githubusercontent.com/microsoft/nav-arm-templates/master/getbc.json)

You need to data string encode this url and append it to [https://portal.azure.com/#create/Microsoft.Template/uri/](https://portal.azure.com/#create/Microsoft.Template/uri/) – then you will have a direct URL, that launches your template:

[https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmicrosoft%2Fnav-arm-templates%2Fmaster%2Fgetbc.json](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmicrosoft%2Fnav-arm-templates%2Fmaster%2Fgetbc.json)

You can also use the azure function, which also is used by the short urls:

[https://freddyk.azurewebsites.net/api/deploy?template=https%3A%2F%2Fraw.githubusercontent.com%2Fmicrosoft%2Fnav-arm-templates%2Fmaster%2Fgetbc.json](https://freddyk.azurewebsites.net/api/deploy?template=https%3A%2F%2Fraw.githubusercontent.com%2Fmicrosoft%2Fnav-arm-templates%2Fmaster%2Fgetbc.json)

This offers the advantage to specify parameters and build out a string, which can deploy exactly the Azure VM you need (example setting the VmName)

[https://freddyk.azurewebsites.net/api/deploy?template=https%3A%2F%2Fraw.githubusercontent.com%2Fmicrosoft%2Fnav-arm-templates%2Fmaster%2Fgetbc.json&vmName=myamazingvm](https://freddyk.azurewebsites.net/api/deploy?template=https%3A%2F%2Fraw.githubusercontent.com%2Fmicrosoft%2Fnav-arm-templates%2Fmaster%2Fgetbc.json&vmName=myamazingvm)

You get the picture…

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
