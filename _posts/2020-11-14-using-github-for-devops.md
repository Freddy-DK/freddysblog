---
layout: post
title: "Using GitHub for DevOps"
date: 2020-11-14 21:07:43
categories: ["AL Development", "CI/CD"]
tags: ["AL", "CD", "CI", "DevOps"]
permalink: /2020/11/14/using-github-for-devops/
---

For the last 2 years we have had the Hands On Lab ([https://aka.ms/cicdhol](https://aka.ms/cicdhol)) in a few iterations using Azure DevOps as the platform. Latest revision was published a few weeks ago.

There are however multiple platforms for DevOps out there and I wanted to investigate the functionality of [Github](https://github.com/about), which also is part of the Microsoft family and probably the biggest provider of devops services to developers in the world.

# Azure DevOps vs. GitHub

This blog post is not going to do an in-depth comparison on Azure DevOps vs. GitHub – I haven’t worked enought with any of the products to do so. I will only mention a few things I found, when trying to setup a CI/CD pipeline in an AL repo. I did however stumble over a few things, which I want to mention here.

GitHub is natively file based. Almost everything in the repo is in files and very little is in metadata. This makes the CI/CD workshop so much easier.

Azure DevOps does have yaml based pipelines, but there are a lot of metadata around the setup of the pipeline and the release pipelines are almost purely metadata. This makes it fairly complex to complete the hands on lab.

Azure DevOps does however seem more professional and this blog post should not be seen as a recommendation for using GitHub instead of Azure DevOps. I see them more as two options, which can do the same thing.

# Setup a GitHub based AL project with CI/CD in 5 minutes…

Login to your Github account

![](/assets/images/2020/using-github-for-devops/image.png)

Navigate to [https://github.com/businesscentralapps/helloworld](https://github.com/businesscentralapps/helloworld)

![](/assets/images/2020/using-github-for-devops/image-1.png)

Click **Use this template**, fill out the repository name and click **Create repository from template**:

![](/assets/images/2020/using-github-for-devops/image-2.png)

Click **Settings** -> **Secrets** and click **New repository secret**:

![](/assets/images/2020/using-github-for-devops/image-3.png)

Add 3 secrets: **LicenseFile** (secure Url to your license file), **InsiderSasToken** (from aka.ms/collaborate) and optionally **StorageAccountConnectionString** (connection string to an azure Storage Account).

![](/assets/images/2020/using-github-for-devops/image-4.png)

Navigate to **Code**, Click the **Code** button and copy the **Git Clone url** to the clipboard:

![](/assets/images/2020/using-github-for-devops/image-5.png)

Start **VS Code**, press Ctrl+Shift+P and run the command **Git Clone** and paste the **Git Clone Url**:

![](/assets/images/2020/using-github-for-devops/image-6.png)

Select a **location for your repository**, **clone the repo**, **open the repo** and **open the workspace**:

![](/assets/images/2020/using-github-for-devops/image-7.png)

Open the **MySolution.ps1** script and **Run the script** by clicking the **Run Code** button:

![](/assets/images/2020/using-github-for-devops/image-8.png)

Click the **Source Control** area, **Stage all changes**, **Commit** and **Push**:

![](/assets/images/2020/using-github-for-devops/image-9.png)

Go back to **GitHub** and see that your **CI** pipeline is running:

![](/assets/images/2020/using-github-for-devops/image-10.png)

Click **Details** and see the CI pipeline running:

![](/assets/images/2020/using-github-for-devops/image-11.png)

And when the pipeline is over, you will be able to **download the artifacts and see the test results**.

![](/assets/images/2020/using-github-for-devops/image-12.png)

And if you define another secret called **StorageAccountConnectionString**, the CD pipeline will make the artifacts available on that storage account (these urls belongs to my storage account: businesscentralapps):

[https://businesscentralapps.blob.core.windows.net/githubhelloworld-preview/latest/apps.zip](https://businesscentralapps.blob.core.windows.net/githubhelloworld-preview/latest/apps.zip)

and if you run the **release pipeline** you get:

[https://businesscentralapps.blob.core.windows.net/githubhelloworld/latest/apps.zip](https://businesscentralapps.blob.core.windows.net/githubhelloworld/latest/apps.zip)

This URL from the release pipeline should be added to the **settings.json** file as **previousApps**.

That’s it – I will add this to the hands on lab to give people a chance of choice.

# Make it your own…

Removing the app, base and test folders and adding your own folders with your apps and tests and then modifying the settings.json file in the scripts folder specify appFolders, testFolders, name and other settings should make the repo work for your own project in another few minutes.

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
