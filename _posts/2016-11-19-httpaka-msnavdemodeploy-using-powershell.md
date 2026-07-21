---
layout: post
title: "http://aka.ms/navdemodeploy – using PowerShell"
date: 2016-11-19 21:39:42
categories: ["Archive", "Azure"]
tags: ["Azure", "DEMO scripts", "Deployment", "Gallery", "NAV 2017", "That Was Easy"]
permalink: /2016/11/19/httpaka-msnavdemodeploy-using-powershell/
---

It is part of my DNA, that no matter how easy things have become, it can always become easier.

In [this](https://freddysblog.com/2016/11/19/one-nav-2017-on-azure-loaded-please/) post, I have stated, that the easiest way to create a NAV 2017 environment on Azure is using [http://aka.ms/navdemodeploy](http://aka.ms/navdemodeploy). But what if you want to create the same environment again and again or you need to create an environment in 5 different languages?

Then there is a lot of typing in names, and passwords and a lot of things that can go wrong.

Can’t we automate that part as well?

Of course we can, and the answer is PowerShell.

In the new Azure PowerShell CmdLets there are a set of CmdLets for handling Azure Resource Manager deployments. The following code will produce demo environments with 5 different country versions of NAV:

$VerbosePreference = "Continue"
$ErrorActionPreference = "stop"
Import-Module -Name "AzureRM.Resources"

# Login to your Azure Subscription
Login-AzureRmAccount

# Select subscription
$mysubscriptionID = ""
Set-AzureRmContext -SubscriptionID $mysubscriptionID

# ARM template
$templateUri = "https://raw.githubusercontent.com/NAVDEMO/NAV2017/master/deploydemo.json"

"W1", "DK", "GB", "NA", "DE" | % {
  # Setup parameter array for ARM template
  $country = $\_
  $name = "mynav2017$country"
  $Parameters = New-Object -TypeName Hashtable
  $Parameters.Add("vmName", $name)
  $Parameters.Add("adminPassword", (ConvertTo-SecureString -String "" -AsPlainText -Force))
  $Parameters.Add("country", $country)
  $Parameters.Add("clickonce", "Yes")
  $Parameters.Add("powerBI", "Yes")
  $Parameters.Add("bingMapsKey", "")
  $Parameters.Add("Office365UserName", "")
  $Parameters.Add("Office365Password", (ConvertTo-SecureString -String "" -AsPlainText -Force))
  $Parameters.Add("Office365CreatePortal", "Yes")

  # GO!
  $location = "West Europe"
  $resourceGroup = New-AzureRmResourceGroup -Name $name -Location $location
  $resourceGroup | Test-AzureRmResourceGroupDeployment -TemplateUri $templateUri -TemplateParameterObject $Parameters
  $resourceGroup | New-AzureRmResourceGroupDeployment -TemplateUri $templateUri -TemplateParameterObject $Parameters -Name $name
}

and the following code will remove all 5 environments again:

"W1", "DK", "GB", "NA", "DE" | % {
  # Setup parameter array for ARM template
  $country = $\_
  $name = "mynav2017$country"
  Remove-AzureRmResourceGroup -Name $name -Force
}

It might not be much faster, but it is a whole lot easier.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
