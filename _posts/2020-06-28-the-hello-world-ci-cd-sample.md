---
layout: post
title: "The Hello World CI/CD sample"
date: 2020-06-28 22:06:46
categories: ["AL Development", "CI/CD", "Docker"]
tags: ["Artifacts", "CD", "CI", "Docker", "YAML"]
permalink: /2020/06/28/the-hello-world-ci-cd-sample/
---

**Update 2021/2/10:** BcContainerHelper has replaced NavContainerHelper. This blog post still reference NavContainerHelper, which is outdated.

Quite a few partners have build their CI/CD pipelines based on the HelloWorld CI/CD sample repository here: [https://dev.azure.com/businesscentralapps/HelloWorld](https://dev.azure.com/businesscentralapps/HelloWorld) (the one used in the Hands On Lab – [http://aka.ms/cicdhol](http://aka.ms/cicdhol)).

I have just finished the upgrade of this repository to use artifacts instead of docker images. This blog post describes the changes done.

![](/assets/images/2020/the-hello-world-ci-cd-sample/pipelines2.png)

# Settings.json

[https://dev.azure.com/businesscentralapps/HelloWorld/\_git/HelloWorld?path=%2Fscripts%2Fsettings.json](https://dev.azure.com/businesscentralapps/HelloWorld/_git/HelloWorld?path=%2Fscripts%2Fsettings.json)

Settings.json was changed to include artifacts instead of container images. Also I have removed the alwaysPull setting, which is really designed to indicate whether or not to check if there is a newer image available – that is not needed anymore.

# Create-Container.ps1

[https://dev.azure.com/businesscentralapps/HelloWorld/\_git/HelloWorld?path=%2Fscripts%2FCreate-Container.ps1](https://dev.azure.com/businesscentralapps/HelloWorld/_git/HelloWorld?path=%2Fscripts%2FCreate-Container.ps1)

Create-Contailer was changed to look at the artifact setting instead of the ContainerImage setting. Furthermore, if the imagename is set in settings, it transfers this to the container to pre-build a specific image for subsequent usages.

# Read-Settings.ps1

[https://dev.azure.com/businesscentralapps/HelloWorld/\_git/HelloWorld?path=%2Fscripts%2FRead-Settings.ps1](https://dev.azure.com/businesscentralapps/HelloWorld/_git/HelloWorld?path=%2Fscripts%2FRead-Settings.ps1)

Read-Settings.ps1 was changed to use the new settings and set devops variables accordingly.

# CI.yaml

[https://dev.azure.com/businesscentralapps/HelloWorld/\_git/HelloWorld?path=%2Fscripts%2FCI.yml](https://dev.azure.com/businesscentralapps/HelloWorld/_git/HelloWorld?path=%2Fscripts%2FCI.yml)

In the pipeline, I removed the task, which performs the insider docker login and I changes the pool to use azure hosted agents.

# Local-Build and AzureVM-Build

[https://dev.azure.com/businesscentralapps/HelloWorld/\_git/HelloWorld?path=%2Fscripts%2FLocal-Build.ps1](https://dev.azure.com/businesscentralapps/HelloWorld/_git/HelloWorld?path=%2Fscripts%2FLocal-Build.ps1) and [https://dev.azure.com/businesscentralapps/HelloWorld/\_git/HelloWorld?path=%2Fscripts%2FAzureVM-Build.ps1](https://dev.azure.com/businesscentralapps/HelloWorld/_git/HelloWorld?path=%2Fscripts%2FAzureVM-Build.ps1)

These functions are used to simulate/run a pipeline locally or in an Azure VM. The functions have been changed to use the new settings (and cache docker images if imageName was specified)

# Local-Sandbox and AzureVM-Sandbox

[https://dev.azure.com/businesscentralapps/HelloWorld/\_git/HelloWorld?path=%2Fscripts%2FLocal-Sandbox.ps1](https://dev.azure.com/businesscentralapps/HelloWorld/_git/HelloWorld?path=%2Fscripts%2FLocal-Sandbox.ps1) and [https://dev.azure.com/businesscentralapps/HelloWorld/\_git/HelloWorld?path=%2Fscripts%2FAzureVM-Sandbox.ps1](https://dev.azure.com/businesscentralapps/HelloWorld/_git/HelloWorld?path=%2Fscripts%2FAzureVM-Sandbox.ps1)

These functions are used to create a sandbox container for the project locally or in an Azure VM. The functions have been changed to use the new settings (and cache docker images if imageName was specified)

# Initialize.ps1

[https://dev.azure.com/businesscentralapps/HelloWorld/\_git/HelloWorld?path=%2Fscripts%2FInitialize.ps1](https://dev.azure.com/businesscentralapps/HelloWorld/_git/HelloWorld?path=%2Fscripts%2FInitialize.ps1)

Initialize is used by the functions which locally or in Azure VMs creates sandboxes or runs build pipelines to read settings and initialize values. Kind of the local version of Read-Settings.

# Adopt the changes

I updated the scripts for all three versions (hybrid, 14.x and 16.x). If you haven’t modified the PowerShell scripts, you can probably just download new versions and override the old. The settings and yaml file you will have to change by hand.

The hands on lab will also be updated to adopt the changes to artifacts.

Hope this helps

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
