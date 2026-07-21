---
layout: post
title: "AAD authentication, Edit In Excel, Embedded PowerBi and http://aka.ms/GETNAV"
date: 2018-06-29 05:04:34
categories: ["Demo Environments"]
tags: ["AAD", "AAD Apps", "Authentication", "Edit In Excel", "Embedded Power BI", "Office 365", "Power BI", "PowerApps"]
permalink: /2018/06/29/aad-authentication/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

I do not have a count of how many time somebody have asked me, e-mailed me, sent messages asking for the “old” NAVDEMODEPLOY with NAV 2018 or Business Central Sandbox Containers.

What people are alluding to is really [this blog post](/2016/11/19/one-nav-2017-on-azure-loaded-please/) – and I am happy to announce that as of today you can now do most of the steps on that list.

Only things missing is the SharePoint Portal and the Demo Apps. The idea is to create the SharePoint portal functionality as an App – so stay tuned…:-)

# AAD authentication

For quite some time, it has been possible to spin up Azure VMs with AAD authentication, just by specifying your Office 365 admin credentials in the [http://aka.ms/getnav](http://aka.ms/getnav) template in these fields:

[![](/assets/images/2018/aad-authentication/cb0bf-o365creds-1.png)](/assets/images/2018/aad-authentication/cb0bf-o365creds.png)

Behind the scenes, the ARM template would invoke a function in the navcontainerhelper called Create-AadAppsForNav. This function actually was the same function used in navdemodeploy to setup AAD, but recently people started to have problems with this. The function was built using the tools that were available back then (some AzureRM powershell module and the microsoft graph) which might not have been built for this. But… – it has served us well for years, may that code rest in piece.

Now we have the AzureAD Powershell module, which is built for this purpose and creating an AAD App for Web Client single signon requires only one line of code:

```
$ssoAdApp = New-AzureADApplication -DisplayName "NAV WebClient for $appIdUri" `
                                   -Homepage $publicWebBaseUrl `
                                   -IdentifierUris $appIdUri `
                                   -ReplyUrls $publicWebBaseUrl
```

Wow.

_Note: For Business Central an additional replyUrl with SignIn added is required._

The Create-AadAppsForNav also used created an App for Edit In Excel and for embedded PowerBI and also that has become similarly easy. The code is available [here](https://github.com/Microsoft/navcontainerhelper/blob/master/AzureAD/Create-AadAppsForNav.ps1).

So, all in all – the new method is more stable, faster and the code is easier to read.

Having the AzureAD PowerShell module also made it easy to create a function called Create-AadUsersInNavContainer, so of course that was added and of course a field was added to the [http://aka.ms/getnav](http://aka.ms/getnav) template:

[![](/assets/images/2018/aad-authentication/c8206-createaadusers-1.png)](/assets/images/2018/aad-authentication/c8206-createaadusers.png)

# Edit In Excel

As mentioned above, the AAD App for Edit In Excel is also created by the Create-AadAppsForNAV and as of today, the [http://aka.ms/getnav](http://aka.ms/getnav) ARM template will also configure Edit In Excel in  the NAV Container or the Business Central Sandbox Container. This is done by setting _ExcelAddInAzureActiveDirectoryClientId_ in CustomSettings.config to the AdAppId field in the object returned from the New-AzureAdApplication creating the Excel Aad App.

```
Set-NAVServerConfiguration -ServerInstance nav -KeyName "ExcelAddInAzureActiveDirectoryClientId" -KeyValue "$ExcelAdAppId"
```

Having done this, the Open In Excel menu item in NAV or Business Central automagically changes to Edit In Excel:

[![](/assets/images/2018/aad-authentication/424b3-editinexcel-1.png)](/assets/images/2018/aad-authentication/424b3-editinexcel.png)

# PowerBI dashboards

With navdemodeploy, we could add another service tier, providing insecure web services access and through this create dashboards. With [http://aka.ms/getnav](http://aka.ms/getnav), we can use LetsEncrypt and get a free 3 months trusted certificate, which works with Power BI. As of today, there is also a function in the navcontainerhelper to renew the certificate and get another 3 months – more about that in a seperate blog post.

The means that for Power BI dashboards, you just enter the OData Web Services URL and you will have the nicest dashboards:

[![](/assets/images/2018/aad-authentication/bb257-dashboard-1.png)](/assets/images/2018/aad-authentication/bb257-dashboard.png)

No extra steps needed, nice:-)

# Embedded PowerBI

For embedded PowerBI we have to configure the app in NAV / Business Central. This is done by adding the AppID and the Key to table 6300 (Azure Ad App Setup). There are several ways to do this in PowerShell, but I decided to reuse the mechanism from navdemodeploy and download a .fob file with a codeunit and invoke the codeunit from PowerShell.

In the navcontainerhelper, we have functions to help with this:

```
$fobfile = Join-Path $env:TEMP "AzureAdAppSetup.fob"
Download-File -sourceUrl "http://aka.ms/azureadappsetupfob" -destinationFile $fobfile
Import-ObjectsToNavContainer -containerName $containerName -objectsFile $fobfile -sqlCredential $sqlCredential
Invoke-NavContainerCodeunit -containerName $containerName -tenant "default" -CodeunitId 50000 -MethodName SetupAzureAdApp -Argument ($AdProperties.PowerBiAdAppId+','+$AdProperties.PowerBiAdAppKeyValue)
```

And of course this is automagically done when using [http://aka.ms/getnav](http://aka.ms/getnav).

With that, you can now click the “Get Started with Power BI” link and you will get this:[![](/assets/images/2018/aad-authentication/7aa36-authorizepowerbi-1.png)](/assets/images/2018/aad-authentication/7aa36-authorizepowerbi.png)

Click the authorize and you should see:

[![](/assets/images/2018/aad-authentication/a491b-permission-1.png)](/assets/images/2018/aad-authentication/a491b-permission.png)

Accept and you should see:

[![](/assets/images/2018/aad-authentication/b08d2-noreports-1.png)](/assets/images/2018/aad-authentication/b08d2-noreports.png)

Now, you can click the drop down, select a report (if you have one), enable it and voila – embedded PowerBI.

# NAV Containers or Business Central Sandbox Containers

Note, that everything in this blog post works with NAV Containers and with Business Central Sandbox Containers. When using [http://aka.ms/getnav](http://aka.ms/getnav) you just specify microsoft/bcsandbox: in the NAV Docker Image field and your VM will become a Business Central Sandbox environment with AAD authentication, Edit in Excel and everything.

You can also use [http://aka.ms/bcsandboxext](http://aka.ms/bcsandboxext) which defaults to bcsandbox containers and have a few other settings – the end result is the same: An Azure VM with everything:-)

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
