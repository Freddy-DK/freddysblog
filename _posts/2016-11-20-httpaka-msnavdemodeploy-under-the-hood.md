---
layout: post
title: "http://aka.ms/navdemodeploy – under the hood"
date: 2016-11-20 19:41:30
categories: ["Archive", "Azure"]
tags: ["Azure", "DEMO scripts", "Deployment", "Gallery", "Image", "NAV", "NAV 2017"]
permalink: /2016/11/20/httpaka-msnavdemodeploy-under-the-hood/
---

You might be wondering what actually is happening when you navigate to [http://aka.ms/navdemodeploy](http://aka.ms/navdemodeploy), so I thought I would lift the hood and show you what is underneath.

### Short URL

As you probably know, the aka.ms Short URL’s are just simple redirects and the navdemodeploy is redirecting to:

[https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FNAVDEMO%2FNAV2017%2Fmaster%2Fdeploydemo.json](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FNAVDEMO%2FNAV2017%2Fmaster%2Fdeploydemo.json)

Looking a little closer, this is actually just asking the Azure Portal to create a deployment based on the Microsoft Template on this Uri:

[https://raw.githubusercontent.com/NAVDEMO/NAV2017/master/deploydemo.json](https://raw.githubusercontent.com/NAVDEMO/NAV2017/master/deploydemo.json)

If you read the blog post about PowerShell from yesterday ([https://freddysblog.com/2016/11/19/httpaka-msnavdemodeploy-using-powershell/](/2016/11/19/httpaka-msnavdemodeploy-using-powershell/)), you will recognize this as the very same URL as we used as the TemplateUri. This means that launching the Azure Portal and fill out the parameters manually will cause exactly the same deployment as we do with PowerShell and everything below still counts.

Clicking the link, will show you the template definition, which as the extension reveals is just a json file.

You might even notice that the URL smells like github and yes, indeed, on github, under the NAVDEMO organization, there is a repository called NAV2017 and a branch called master:

[https://github.com/NAVDEMO/NAV2017](https://github.com/NAVDEMO/NAV2017)

Open Source, No Secrets.

Currently, this branch has only one contributor (me), but I welcome other people to contribute if they would like to make our Azure Deployment better.

### What is JSON?

If you don’t speak json, it is about time.

JSON stands for _JavaScript Object Notation_ and was invented by a guy called [Douglas Crockford](http://www.crockford.com/) (Pretty fun guy, read his blog).

You can open [http://json.org](http://json.org) to understand the syntax or [http://json.org/example.html](http://json.org/example.html) to see some examples and there are a million web sites explaining the difference between json and XML.

I think the biggest difference is, that json is more lightweight and easier human readable/writeable (having said that, I don’t think any of them are easy.

### JSON is not just JSON

Like XML, JSON consists of a hierarchy of objects and arrays.  In order for the Azure Resource Manager (ARM) Deployment system, the JSON file of course have to adhere to a certain schema, which is described in the beginning of the JSON file.

This schema dictates what needs to be there and how the relationships between things are.

I am not going to make a tutorial on how to build templates for ARM, there are a log of info already out there, which can help:

-   Good Starting point: [https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-template-best-practices](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-template-best-practices)
-   A number of quickstart templates (samples): [https://azure.microsoft.com/en-us/resources/templates/](https://azure.microsoft.com/en-us/resources/templates/)
-   Nice description about the basics: [https://govindkanshi.wordpress.com/2015/05/03/azure-arm-a-journey-to-understand-basics/](https://govindkanshi.wordpress.com/2015/05/03/azure-arm-a-journey-to-understand-basics/)
-   11 post series about ARM: [http://www.ravichaganti.com/blog/building-azure-resource-manager-templates-an-introduction/](http://www.ravichaganti.com/blog/building-azure-resource-manager-templates-an-introduction/)

### It is not easy, but it doesn’t have to be hard

JSON is just a text file and you can of course edit it in notepad, but I wouldn’t recommend that. There are so many tools out there that understands JSON and can help, but only a few that actually understands the ARM template schema.I use Visual Studio Code or Visual Studio 2015.

#### Visual Studio Code

Visual Studio Code natively supports JSON editing and can help you create JSON which is formattet correctly. Moreover, you can install a few extensions for VS Code, which understands the ARM Template Schema:

-   armsnippet (from Art of Shell) will give you a number of json snippets to choose between.
-   Azure Resource Manager Tools (from Microsoft) will give you warnings if you have unused or unknown variables/parameters and a lot of other good stuff.

#### Visual Studio 2015

Probably the best tool for ARM template editing. Visual Studio has a project template for Azure Resource Group, which will give you all the tools, snippets, expand/collapse etc. – everything you need for editing ARM templates.

### Inside the deploydemo.json

Lets look a little closer at the deploydemo.json file. I have collapsed the variables, the resources and the outputs and this lists the parameters. These parameters are the same as you see when you launch the deployment. I have expanded a few of the parameters to give an indication of how the parameters are defined:

```
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": ...,
    "vmAdminUsername": ...,
    "navAdminUsername": ...,
    "adminPassword": {
      "type": "securestring",
      "metadata": {
        "Description": "Specify Administrator Password (for VM and NAV)"
      }
    },
    "country": ...,
    "CertificatePfxUrl": ...,
    "CertificatePfxPassword": ...,
    "PublicMachineName": ...,
    "bingMapsKey": ...,
    "clickonce": {
      "type": "string",
      "defaultValue": "No",
      "allowedValues": [ "Yes", "No" ],
      "metadata": {
        "Description": "Install Clickonce Support?"
      }
    },
    "powerBI": ...,
    "Office365UserName": ...,
    "Office365Password": ...,
    "Office365CreatePortal": ...,
  },
  "variables": ...,
  "resources": ...,
  "outputs": ...
}
```

Under variables, you will find a list of variables needed for the deployment. Stuff like the StorageAccountName is calculated here based on the name you have specified. The VM Size is hardcoded to be a D2 and the name of virtual networks, security groups etc. are all hardcoded here.

Under resources, you will find all the resources, which will be deployed by the template:

1.  StorageAccount for usage by the Virtual Machine
2.  Public IP address with a DNS Name
3.  Network Security Group with all the necessary endpoints (80, 443, 7046, 7047, 7048, 49000)
4.  Virtual Network for the Virtual Machine
5.  Network interface for the Virtual Machine
6.  Virtual Machine based on the NAV Public Image with diagnostics and perf. counters
7.  Virtual Machine PowerShell extension for running the demo scripts on the Virtual Machine.

and under outputs, you will find outputting the url of the landing page.

If we remove resource #7 – then you will basically just deploy the NAV Image on Azure and open the necessary ports.

### Microsoft.Compute.CustomScriptExtension

If you navigate to the last resource, you will find the CustomScriptExtension (the PowerShell extension). This extension depends on the Virtual Machine, meaning that it won’t start until the Virtual Machine is complete. The Virtual Machine depends on some of the other artefacts, which basically means that the script will be executed when everything else is done.

```
{
  "apiVersion": "2015-06-15",
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "name": "[concat(variables('server1Name'),'/vmextension1')]",
  "location": "[resourceGroup().location]",
  "tags": {
    "displayName": "PowerShellScript2"
  },
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', variables('server1Name'))]"
  ],
  "properties": {
    "publisher": "Microsoft.Compute",
    "type": "CustomScriptExtension",
    "typeHandlerVersion": "1.4",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "fileUris": [
        "[concat(variables('ScriptFilePath'), 'initialize.ps1')]"
      ],
      "commandToExecute": "[concat(variables('ScriptCommandToExecute'),'initialize.ps1',' -PatchPath "',variables('ScriptFilePath'),'" -StorageAccountName "...
    },
    "protectedSettings": {
    }
  }
}
```

The script to execute is called initialize.ps1 and it is downloaded from the ScriptFilePath (which is the same folder as the .json file).

You can see the script here: [https://github.com/NAVDEMO/NAV2017/blob/master/initialize.ps1](https://github.com/NAVDEMO/NAV2017/blob/master/initialize.ps1)

### The script

The script will be downloaded to _C:\\Packages\\Plugins\\Microsoft.Compute.CustomScriptExtension\\1.8\\Downloads\\initialize.ps1_ and executed from this location with PowerShell. All goodness you think, so why does the script seem so complicated???

Well, there are a few issues.

-   The script is running as a SYSTEM user and when it is running.
-   The vmadmin user has not been created at the time, when the script is running.
-   The script requires a reboot (after installing the new Azure PowerShell CmdLets), we need to find a good way to continue installation after the reboot.

So basically what happens in the script today is, that it will create some scripts and a Task in the Task Scheduler for running these scripts as “NT AUTHORITYSYSTEM”. Then it will reboot and launch the first of these scripts. This script will create a new Task in the Task Scheduler (running as the VM Admin which has been created) and reboot again. Now the remaining installation scripts will be performed and the system will be setup.

Part of the script will also patch some of the demo scripts with newer versions, which supports the embedded PowerBI AAD App and the Edit in Excel AAD App. All of this will go away with CU1 (also the double reboot thing).

Sorry about the complexity, it will be a little simpler in CU1, but still, the job of the initialize script is to do what it takes to make things work, so it is just doing its job – don’t blame it for that…

### Contributions welcome

If you have good ideas to what we can add to the NAV Image on Azure – new scripts, other integration areas, new innovations, then please clone the github project, try it out and send a pull request. Just remember, that the NAV Image on Azure supports all 20 country versions and your idea needs to do the same.

Thanks

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
