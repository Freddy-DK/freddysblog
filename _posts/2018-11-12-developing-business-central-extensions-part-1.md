---
layout: post
title: "Developing Business Central Extensions (part 1) – Prerequisites"
date: 2018-11-12 15:28:58
categories: ["AL Development", "CI/CD"]
tags: ["AL", "Business Central Sandbox", "CD", "CI", "Continuous Delivery", "Continuous Integration", "Extensions"]
permalink: /2018/11/12/developing-business-central-extensions-part-1/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

# A professional development environment

When developing customizations for Microsoft Dynamics NAV, we have been used to have object modifications and source code stored in the database. A lot of partners have setup pools of databases with all their customer solutions. A lot of partners have developed mechanisms to use source code management by exporting the objects as text and using delta and merge tools – in general, partners have found a way to work with the complexity of code customizing NAV.

With development changing to extensions v2 in Visual Studio Code we are becoming a professional development environment. Source code is stored in files, matching the requirements of source code management systems very well. Git is an integral part of VS Code and using Azure DevOps or GitHub for source code becomes very easy.

When Developing for Business Central, you need a Business Central sandbox environment from which you can download symbols and where you can publish and run/test your extension.

In this blog series we will use Docker for setting up the Business Central Sandbox environments, either locally or on Azure. You will not be able to use the online Business Central Sandbox Environment for this.

# Prerequisites

In order to complete the steps in this blog series, you will need a few things.

## Get an Azure Subscription

The Azure subscription will be used to host a Key Vault for your secrets and any Azure VMs (optional) used for development environments. Open [https://azure.microsoft.com/en-us/](https://azure.microsoft.com/en-us/) to create a free subscription. We will also use the Azure Subscription to host a build agent when setting up continuous integration and continuous delivery (CI/CD).

## Get an Azure DevOps Account

The Azure DevOps account and organization is where you will create your projects and store your source code. Open [https://azure.microsoft.com/en-us/services/devops/](https://azure.microsoft.com/en-us/services/devops/) to create a free account. You will be able to create public or private projects in Azure DevOps.

## Setup a Windows Development computer

Even though you can run VS Code on MAC and you can host the Business Central Sandbox Environment on Azure – I will still require a Windows Development computer for this blog series. I will be running some PowerShell scripts, which I do not think will work on Mac and even though you might be able to run these on an Az Cli prompt on Azure, I have not gone through and tested this, nor do I intend to do so.

The Windows Computer can be Windows 10 or Windows Server – both should work.

### Install Docker (optional)

If you want to run your Business Central sandbox environment locally, you need Docker.

If you do not install Docker, you will need to setup your sandbox environment elsewhere. This walkthrough can setup the sandbox environment locally or in an Azure VM. To install Docker, navigate to [https://download.docker.com/win/stable/Docker%20for%20Windows%20Installer.exe](https://download.docker.com/win/stable/Docker%20for%20Windows%20Installer.exe) to download and install Docker.

**Remember** to select to use Windows Containers instead of Linux Containers.

### Install AzureRM PowerShell Module

On your Windows Development machine, open PowerShell ISE and run

Set-ExecutionPolicy unrestricted
Install-Module -Name “AzureRM” -Force

If you get a question about updating NuGet, you will need to answer Yes.

The reason for changing the execution policy is, that it is a requirement for a number of the AzureRM cmdlets.

### Install Visual Studio Code

Developing extensions for Business Central is done in Visual Studio Code. Navigate to [https://code.visualstudio.com/Download](https://code.visualstudio.com/Download) to download and install Visual Studio code.

After installing VS Code, Start VS Code open the extensions marketplace and install the extensions you need. The following VS Code Extensions are used in this blog series:

-   AL Language from Microsoft
-   PowerShell from Microsoft
-   GitLens from Eric Amodio
-   Insert GUID from Heath Stewart
-   Docker Explorer from Jun Han

### Install and configure Git

Git is the source code management tool used by Visual Studio Code to connect to your Azure DevOps repository. Navigate to [https://www.git-scm.com/download/win](https://www.git-scm.com/download/win) and click the download link to download and install Git.

Select Visual Studio Code as Git’s default editor during installation Wizard.

Configure your user name and email in git by starting a command prompt and type:

git config --global user.name ""
git config --global user.email ""

**Note**, in order to make authentication work against my Azure DevOps repository, I had to update my credential manager with the latest version from [https://github.com/Microsoft/Git-Credential-Manager-for-Windows/releases](https://github.com/Microsoft/Git-Credential-Manager-for-Windows/releases) and run

git config --global credential.helper manager

in a command prompt afterwards.

# Create an Azure Key Vault with content

We will need an Azure Key Vault for storing passwords, license file URLs, insider repository credentials and other things. You might think this is overkill, but if you think about it, then **any secret stored directly in Source Code or settings files is really not a secret anymore**.

Using Key Vaults allows me to share scripts without the risk of suddenly exposing certificates, license files or passwords.

Having Installed the other prerequisites, the easiest way to create your Azure Key Vault is to use PowerShell.

Open PowerShell ISE and run

Add-AzureRmAccount -Environment 'AzureCloud'
Set-AzureRmContext -SubscriptionID ''

where is the subscription id of your Azure Subscription. This will ask you to login to your Azure Subscription.

After this, you can create a Resource Group and an Azure Key Vault using:

New-AzureRmResourceGroup -Name "" -Location ""
New-AzureRmKeyVault -Name "" -ResourceGroupName "" -Location "" -Sku Standard

where you need to replace , and with the values of your choice.

Now, you need to set **Username** and **Password** secrets and you will need to create empty secrets for **LicenseFile**, **CodeSignPfxFile** and **CodeSignPfxPassword**:

$Username = "admin"
$Password = ""
Set-AzureKeyVaultSecret -VaultName "" -Name Username            -SecretValue (ConvertTo-SecureString -String $Username -AsPlainText -Force)
Set-AzureKeyVaultSecret -VaultName "" -Name Password            -SecretValue (ConvertTo-SecureString -String $Password -AsPlainText -Force)
Set-AzureKeyVaultSecret -VaultName "" -Name LicenseFile         -SecretValue (new-object System.Security.SecureString)
Set-AzureKeyVaultSecret -VaultName "" -Name CodeSignPfxFile     -SecretValue (new-object System.Security.SecureString)
Set-AzureKeyVaultSecret -VaultName "" -Name CodeSignPfxPassword -SecretValue (new-object System.Security.SecureString)

where should be replaced by the password you want to use for your containers. You can also modify the username if needed.

You can retrieve these values again using:

$Username = (Get-AzureKeyVaultSecret -VaultName "" -Name Username).SecretValueText
$Password = (Get-AzureKeyVaultSecret -VaultName "" -Name Password).SecretValue

Where $Username will be of type string and $Password will be of type SecureString.

If you need to use a specific license file, you will need to set the **LicenseFile** secret to a secure URL, from which your license file can be downloaded.

See [https://freddysblog.com/2017/02/26/create-a-secure-url-to-a-file/](https://freddysblog.com/2017/02/26/create-a-secure-url-to-a-file/) for how to create a secure URL to a file.

If you need to use a code signing certificate, you will need to set the **CodeSignPfxFile** secret to a secure URL, from which your code signing certificate can be downloaded and **CodeSignPfxPassword** secret should be the Pfx Password for this certificate.

# What’s next

Now we are done setting up our accounts, subscriptions and our Development Computer.

In the next Blog Post we will copy a template repository, configure this to your own setup, build and run the app.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
