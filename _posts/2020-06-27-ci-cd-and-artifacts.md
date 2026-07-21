---
layout: post
title: "CI/CD and artifacts"
date: 2020-06-27 08:46:13
categories: ["AL Development", "CI/CD", "Docker"]
tags: ["Artifacts", "CD", "CI", "Docker", "Get-BcArtifactUrl"]
permalink: /2020/06/27/ci-cd-and-artifacts/
---

**Update 2021/2/10:** BcContainerHelper has replaced NavContainerHelper. This blog post still reference NavContainerHelper, which is outdated.

Only a few days have gone since releasing information about how to run Business Central in docker using artifacts and already a lot of people have tried this out and a lot of people are looking into changing their CI/CD pipelines to use artifacts.

# On premises / Specific version

If your pipeline is running using an om-premises build or using a specific version of NAV or Business Central, it is really easy. Replace your imagename setting with an artifact setting pointing to the artifact you want to use and change the function creating the container to use this setting as -artifacturl when calling New-BcContainer.

# PTE / AppSource App

For PTEs (Per Tenant Extensions) or appsource apps it is a little more complex. You probably have a CI pipeline running either on a specific version or on a docker image which is the latest current build, and then you might have two pipelines testing your app against the next minor insider builds and the next major insider builds and since all artifacts are stored using version number and country how do you determine next minor and next major artifact url.

![](/assets/images/2020/ci-cd-and-artifacts/pipelines.png)

I thought of a lot of different ways to do this. During the preview phase, I had redirection artifacts which would point out the right artifact and I also tried using a few redirection urls (aka.ms). In the end the answer was simple.

# Get-BcArtifactUrl

You should of course use Get-BcArtifactUrl. Yes, I know that the function cannot be used in Yaml, but it can be used in PowerShell and I am going to use New-BcContainer anyway so hey, what’s the problem.

In my BingMaps sample app, I changed the imagename setting to an artifact setting, which is a “/” separated string containing of:

**<storageAccount>/<type>/<version>/<country>/<select>**

You can use the same – or use other mechanisms, but the goal is to have one string, in which I can specify the parameters to Get-BcArtifactUrl.

My code to get the artifactUrl based on this is then:

```
$segments = "$artifact////".Split('/')
$artifactUrl = Get-BCArtifactUrl -storageAccount $segments[0] -type $segments[1] -version $segments[2] -country $segments[3] -select $segments[4] -sasToken $env:InsiderSasToken | Select-Object -First 1
if (-not ($artifactUrl)) {
    throw "Unable to locate artifactUrl from $artifact"
}
```

This totally allows me to “call” Get-BcArtifactUrl in my pipeline and I can now specify my artifacts to match my needs.

A few samples of artifact settings:

<table><tbody><tr><td>A specific version</td><td>bcartifacts/sandbox/16.2.13509.14082/us/Latest</td></tr><tr><td>Latest 16.2 sandbox</td><td>bcartifacts/sandbox/16.2/us/Latest</td></tr><tr><td>Latest sandbox</td><td>bcartifacts/sandbox//us/Latest</td></tr><tr><td>Next major</td><td>bcinsider/sandbox//us/Latest</td></tr><tr><td>Next minor</td><td>bcinsider/sandbox//us/SecondToLastMajor</td></tr></tbody></table>

**BTW** – if you need a sasToken for insider builds it is available on [http://aka.ms/collaborate](http://aka.ms/collaborate) – the actual package is here: [https://partner.microsoft.com/en-us/dashboard/collaborate/packages/9387](https://partner.microsoft.com/en-us/dashboard/collaborate/packages/9387) (which you only can see if you have access to the Ready! for Dynamics 365 Business Central program). The package looks like this:

![](/assets/images/2020/ci-cd-and-artifacts/package.png)

# The BingMaps sample

The BingMaps sample which already have been changed to use artifacts can be found here: [https://dev.azure.com/businesscentralapps/BingMaps](https://dev.azure.com/businesscentralapps/BingMaps). I also have an imagename parameter, which I use to allow the pipeline to build the image and reuse that next time.

One very interesting new thing with the BingMaps sample is. that it used the free Azure Hosted agents to run the builds. I do NOT have any build agent setup myself.

Due to the new way of running with artifacts, that is now possible – yes it takes a little longer – but for open source projects it is free. You can read more about Microsoft hosted agents here: [https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops&tabs=yaml](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops&tabs=yaml) (see capabilities and limitations to see what you get for free).

# The HelloWorld sample

The Hands On Lab ([http://aka.ms/cicdhol](http://aka.ms/cicdhol)) and the HelloWorld sample ([https://dev.azure.com/businesscentralapps/HelloWorld](https://dev.azure.com/businesscentralapps/HelloWorld)) will also be updated to use artifacts and Microsoft hosted agents.

# Open Source Apps

This actually opens up for something I have been thinking about for a very long time. Open Source Apps. With source code control, devops, pipelines and all free for open source projects – it is now possible to create open source apps, on which people can collaborate and use.

More about this idea at a later time…

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
