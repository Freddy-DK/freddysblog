---
layout: post
title: "The ARM templates for Dynamics 365 Business Central and Microsoft Dynamics NAV"
date: 2019-07-26 14:35:05
categories: ["Azure", "Demo Environments", "Docker", "NavContainerHelper"]
tags: ["ARM", "DEMO", "Docker", "NavContainerHelper"]
permalink: /2019/07/26/the-arm-templates-for-dynamics-365-business-central-and-microsoft-dynamics-nav/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

Moving quickly towards a world, where C/SIDE and C/AL development in Dynamics 365 Business Central is history, I have made some cleanup of the ARM templates and this blog posts describes the purpose of the templates and what they support.

# ARM templates

Azure Resource Management templates are templates, which describes a set of resources, which needs to be deployed and initialized in order for the ARM template to be deployed successfully.

An ARM template can result in one or many resource groups and resources and these resources can be Azure VMs, Storage Accounts, Web Apps, Network Security Groups etc. etc. all resources supported by Azure are supported in ARM templates.

You will find a general description of ARM templates here: [https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates)

All the ARM templates for Dynamics 365 Business Central and Microsoft Dynamics NAV are placed on GitHub here: [https://github.com/Microsoft/nav-arm-templates](https://github.com/Microsoft/nav-arm-templates)

When using a URL like [http://aka.ms/getbc](http://aka.ms/getbc), you will be redirected to the Azure Portal and use the ARM template specified by [https://github.com/microsoft/nav-arm-templates/blob/master/getbc.json](https://github.com/microsoft/nav-arm-templates/blob/master/getbc.json).

# getbc or getnav

Given the same properties, these templates will produce **exactly the same VMs** – there is no difference. Everything has to do with the values you enter into the properties.

So, as mentioned, I have modified the templates a bit (and removed some, so there are 6 left)

I have removed all support for C/SIDE, Symbol loading and other NAV and C/AL related stuff from getbc. You can still deploy a Business Central Spring release with **getnav** and get these features in – but **getbc** follows the direction of Business Central – AL only.

If you need Windows Client, C/SIDE, Symbol Loading, C/AL – use **getnav** or **getnavext**.

# All properties explained…

Below, you will find a list of all properties and a brief explanation of what they are used for. Please note, that things can change – new properties can be created and properties can disappear. I will try to maintain full functionality in the **ext** templates and simpler functionality in the standard templates.

# Common for all ARM templates

Common for all Dynamics 365 Business Central or Microsoft Dynamics NAV ARM templates is, that you can specify the subscription in which you want to deploy your resources, the resource group, which will contain all resources and the location in which you want to deploy all resources.

**Vm Name** specifies the name of the VM or the prefix of the VMs if the template deploys multiple VMs.

**Timezone Id** specifies the timezone of the VM.

**Accept Eula** is a dropdown, where you have to specify **Yes** in order to accept the EULA. Hover over the (i) to get a URL to read the EULA.

**Remote Desktop Access** specifies a list of IP numbers or IP number ranges which should have access to connect to the VM via Remote Desktop (port 3389). Specify a \* to indicate that you allow access from everywhere and – to indicate that you do not want any access via remote desktop. Example: **176.23.9.66** or **176.23.9.0/24**.

**Operating System** is a dropdown, where you can select between **Windows Server 2016**, **Windows Server 2019** or **Windows Server 2019 with Containers** and this basically determines which Azure VM Image is used as a starting point for your VM. The default is **Windows Server 2019 with Containers**, which also is the fastest and you should need a good reason for selecting another OS.

**Vm Size** specifies the Size of the VM. Read more about the VM sizes here: [https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sizes](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sizes)

**Vm Admin Username** specifies the username of the admin user for the Virtual Machine.

**BC/NAV Admin Username** specifies the username of the admin user for Business Central or NAV. The BC/NAV container is setup for Username/Password authentication.

**Admin Password** specifies the password for both VM admin user and BC/NAV admin user.

**BC/NAV Docker Image** specifies the docker image used for this deployment. for **getbc** this defaults to the latest public Business Central image, for **getnav**, this defaults to the latest public NAV image.

**License File Uri** should specify a **secure Url to a license file**, which will get downloaded and used for the BC/NAV container.

**Enable Task Scheduler** should be **Yes** if you want the Task Scheduler to run in the BC/NAV container.

**Run Windows Update** determines whether or not you want to run Windows Updates on the VM after deployment.

**Contact EMail for Lets Encrypt** when setting up an Azure VM, the BC/NAV container is always using SSL. Lets Encrypt provides a free 90 day SSL certificate if you specify your email address in this field. You then also accept the license agreement here: [https://letsencrypt.org/repository/](https://letsencrypt.org/repository/). If not, you will get a self signed certificate. Note that LetsEncrypt doesn’t work with NAV 2017 or earlier.

**Auto Shutdown** will determine whether you enable or disable automatic shutdown of your container at a certain time of day (in the timezone above).

**Auto Shutdown Time** is the time-of-day where the VM automatically will shut down if Auto Shutdown is enabled.

**Add Traefik** is a Yes/No dropdown on whether or not you want to use traefik for hosting multiple docker containers on the same VM.

# [http://aka.ms/getbc](http://aka.ms/getbc)

The getbc template is used to get an Azure VM with Business Central. This VM is targeted at AL development and **does NOT support C/SIDE nor Windows Client**. There are no options to select ClickOnce deployment of Windows Client in this template, nor is EnableSymbolLoading for hybrid development possible. If you need Business Central for C/AL development, hybrid development or you need the Windows Client, you need to use [http://aka.ms/getnav](http://aka.ms/getnav). getbcext (described below) is the extended version of getbc with more and more advanced properties.

Properties specific to getbc (and getbcext) are

**Assign Premium Plan** determines whether the admin user will be assigned the premium plan. This setting should only affects **sandbox containers** (mcr.microsoft.com/businesscentral/sandbox) and should not have any effect on onprem containers.

**Create Test Users** should be set to **Yes** if you want to create a set of test users for testing your app for appsource validation. The setting causes the function **Setup-BcContainerTestUsers** to be called, which will download, publish and install an app, invoke an API and then uninstall and unpublish the app again. The source for this app is here: [https://dev.azure.com/businesscentralapps/CreateTestUsers](https://dev.azure.com/businesscentralapps/CreateTestUsers)

**Office 365 Username** should be an Office 365 administrator account if you want to setup AAD authentication with your container. This functionality uses the function **Create-AadAppsForBc** from NavContainerHelper to create an app in the AAD for single-signon.

**Office 365 Password** should be the password of the Office 365 administrator account. While writing this post, this functionality doesn’t support setting up AAD authentication with accounts requiring dual factor authentication.

**Create AAD users** determines whether the setup scripts will enumerate the AAD users and add all users to the container. It uses the function **Create-AadUsersInBcContainer** from the NavContainerHelper to do this.

**Bing Maps Key** can be a developer key for a BingMaps account if you want to download, publish and install a BingMaps sample app. The source for this app is here: [https://dev.azure.com/businesscentralapps/BingMaps](https://dev.azure.com/businesscentralapps/BingMaps).

# [http://aka.ms/getbcext](http://aka.ms/getbcext)

This is the extended version of getbc – adding a number of advanced fields. This template has all fields from getbc + some more:

**Storage Account Type** can be set to Standard\_LRS or Premium\_LRS depending on the preformance level of the storage account you wan to use. Premium is the more performant and more expensive selection.

**includeAL** should be set to **Yes** if you are going to be modifying the base app or the system application of Business Central (as explained in [this blog post](/2019/04/15/c-al-to-al-code-customizations/)) With this flag, the Business Central container will be prepared for compilation of the base app. This is enabled with Business Central Spring 2019 (14.x) and later.

**SQL Server Type** determines which SQL Server your container will be connecting to. SQLExpress is the built-in SQLExpress in the container. If you select **AzureSQL** you are required to provide **App and Tenant Bacpac Uri**, which will be restored to Azure SQL and your container will be connected to that.

**Azure Sql Admin Username** is the username of the Azure SQL Server (if you select AzureSQL as SQL Server Type).

**App Bacpac Uri** must be provided if you have selected Azure SQL as SQL Server Type. The Uri must point to an app database, which will be restored to Azure SQL. The Uri must point to a blob in Azure Blob Storage, which is either **Public** or a **Shared Access Signature Url** (can be created using Azure Storage Explorer or PowerShell).

**Tenant Bacpac Uri** must be provided if you have selected Azure SQL as SQL Server Type. The Uri must point to a tenant database, which will be restored to Azure SQL. The Uri must point to a blob in Azure Blob Storage, which is either **Public** or a **Shared Access Signature Url** (can be created using Azure Storage Explorer or PowerShell).

**include App Uris** is the property in which you can provide a comma or semicolon separated **list of extensions** you want to install. Every app is downloaded, published and installed in the order of which they appear in the list.

**Multitenant** should be set to **Yes** if you want to create a multitenant container in your Azure VM. The container will only have one tenant called **default**. You can create more tenants using functions in NavContainerHelper.

**Certificate Pfx Url** must be a **secure Url to a certificate .pfx file** if you want to use a certificate purchased by a trusted certificate authority. Beside this setting, you also need to set the **Certificate Pfx Password** and the **Public Dns Name**. [This blog post](/2017/02/26/create-a-secure-url-to-a-file/) describes how to create a secure Url.

**Certificate Pfx Password** must be the **Password** of the password protected certificate .pfx file.

**Public Dns Name** must be the public Dns name you are using for your VM. If you own the domain name **mydomain.net** and create a **CNAME** record **myvm.mydomain.net** pointing to the DNS name of the VM, then this setting should be **myvm.mydomain.net** and the certificate .pfx file should either be a \* certificate for mydomain.net or a certificate for the exact domain name.

**Final Setup Script Url** can be a Url or a secure Url to an optional setup script, which will be downloaded and invoked after setting up the container and the desktop on the VM. A sample final setup script, which installs **Choco**, **Git**, **Chrome**, **Firefox** and a number of **VS Code extensions** can be found here: [https://github.com/microsoft/nav-arm-templates/blob/master/additional-install.ps1](https://github.com/microsoft/nav-arm-templates/blob/master/additional-install.ps1)

**Request Token** describes a **token/password**, which is used to invoke pre-defined scripts on the VM. Use: _http://<landingpage>/request.aspx?cmd=Demo&requesttoken=<token&gt;_. Cmd equals the name of a PowerShell script in the c:\\demo\\request folder and you can add your own scripts to this folder. Parameters should be added as query parameters. Default scripts includes **Demo**, **RestartComputer** and **ReplaceNavServerContainer**. Note that this is using http (not https) – everything will be transferred over the wire in clear text.

**Create Storage Queue** determines whether or not the resource group will contain a storage account with an Azure Storage Queue for invoking pre-defined scripts on the VM. Place a json formatted message like **{ “cmd”: “Demo” }** in the queue to invoke the Demo command. Cmd equals the name of a PowerShell script in the c:\\demo\\request folder and you can add your own scripts to this folder. Parameters should be added in the json message. Transcript is added to a table storage upon completion and you can specify an id in the json file to identify the results.

# [http://aka.ms/getnav](http://aka.ms/getnav)

**Enable Symbol Loading** should be set to **Yes** if (and only if) you are going to make code customizations to the Base App in C/AL **AND** want to use these code changes in AL extensions afterwords (known as **hybrid development**). In this case, the Service Tier needs to know that it should re-generate symbols for your changes. Read more about symbols in [this blog post](/2019/03/16/symbols-demystified/). This is only supported from NAV 2018 and in Business Central versions up until Spring 2019 (14.x).

**Include CSIDE** determines whether or not the VM has support for **C/SIDE development**. If this setting is **Yes**, you will have a shortcut for C/SIDE on the desktop and all pre-requisites installed. You will be able to open and connect to the container with this.

**Click Once** determines whether the Windows Client from Microsoft Dynamics NAV will be available for click-once installation from the VM. This setting only works with NAV containers or Business Central containers before and including spring 2019 (14.x).

**Office 365 Username** should be an Office 365 administrator account if you want to setup AAD authentication with your container. This functionality uses the function **Create-AadAppsForBc** from NavContainerHelper to create an app in the AAD for single-signon.

**Office 365 Password** should be the password of the Office 365 administrator account. While writing this post, this functionality doesn’t support setting up AAD authentication with accounts requiring dual factor authentication.

**Create AAD users** determines whether the setup scripts will enumerate the AAD users and add all users to the container. It uses the function **Create-AadUsersInBcContainer** from the NavContainerHelper to do this.

**Bing Maps Key** can be a developer key for a BingMaps account if you want to download, publish and install a BingMaps sample app. The source for this app is here: [https://dev.azure.com/businesscentralapps/BingMaps](https://dev.azure.com/businesscentralapps/BingMaps).

# [http://aka.ms/getnavext](http://aka.ms/getnavext)

**Storage Account Type** can be set to Standard\_LRS or Premium\_LRS depending on the preformance level of the storage account you wan to use. Premium is the more performant and more expensive selection.

**includeAL** should be set to **Yes** if you are going to be modifying the base app or the system application of Business Central (as explained in [this blog post](/2019/04/15/c-al-to-al-code-customizations/)) With this flag, the Business Central container will be prepared for compilation of the base app. This is enabled with Business Central Spring 2019 (14.x) and later.

**SQL Server Type** determines which SQL Server your container will be connecting to. SQLExpress is the built-in SQLExpress in the container. If you select **AzureSQL** you are required to provide **App and Tenant Bacpac Uri**, which will be restored to Azure SQL and your container will be connected to that.

**Azure Sql Admin Username** is the username of the Azure SQL Server (if you select AzureSQL as SQL Server Type).

**App Bacpac Uri** must be provided if you have selected Azure SQL as SQL Server Type. The Uri must point to an app database, which will be restored to Azure SQL. The Uri must point to a blob in Azure Blob Storage, which is either **Public** or a **Shared Access Signature Url** (can be created using Azure Storage Explorer or PowerShell).

**Tenant Bacpac Uri** must be provided if you have selected Azure SQL as SQL Server Type. The Uri must point to a tenant database, which will be restored to Azure SQL. The Uri must point to a blob in Azure Blob Storage, which is either **Public** or a **Shared Access Signature Url** (can be created using Azure Storage Explorer or PowerShell).

**include App Uris** is the property in which you can provide a comma or semicolon separated **list of extensions** you want to install. Every app is downloaded, published and installed in the order of which they appear in the list.

**Multitenant** should be set to **Yes** if you want to create a multitenant container in your Azure VM. The container will only have one tenant called **default**. You can create more tenants using functions in NavContainerHelper.

**Certificate Pfx Url** must be a **secure Url to a certificate .pfx file** if you want to use a certificate purchased by a trusted certificate authority. Beside this setting, you also need to set the **Certificate Pfx Password** and the **Public Dns Name**. [This blog post](/2017/02/26/create-a-secure-url-to-a-file/) describes how to create a secure Url.

**Certificate Pfx Password** must be the **Password** of the password protected certificate .pfx file.

**Public Dns Name** must be the public Dns name you are using for your VM. If you own the domain name **mydomain.net** and create a **CNAME** record **myvm.mydomain.net** pointing to the DNS name of the VM, then this setting should be **myvm.mydomain.net** and the certificate .pfx file should either be a \* certificate for mydomain.net or a certificate for the exact domain name.

**Final Setup Script Url** can be a Url or a secure Url to an optional setup script, which will be downloaded and invoked after setting up the container and the desktop on the VM. A sample final setup script, which installs **Choco**, **Git**, **Chrome**, **Firefox** and a number of **VS Code extensions** can be found here: [https://github.com/microsoft/nav-arm-templates/blob/master/additional-install.ps1](https://github.com/microsoft/nav-arm-templates/blob/master/additional-install.ps1)

**Request Token** describes a **token/password**, which is used to invoke pre-defined scripts on the VM. Use: _http://<landingpage>/request.aspx?cmd=Demo&requesttoken=<token&gt;_. Cmd equals the name of a PowerShell script in the c:\\demo\\request folder and you can add your own scripts to this folder. Parameters should be added as query parameters. Default scripts includes **Demo**, **RestartComputer** and **ReplaceNavServerContainer**. Note that this is using http (not https) – everything will be transferred over the wire in clear text.

**Create Storage Queue** determines whether or not the resource group will contain a storage account with an Azure Storage Queue for invoking pre-defined scripts on the VM. Place a json formatted message like **{ “cmd”: “Demo” }** in the queue to invoke the Demo command. Cmd equals the name of a PowerShell script in the c:\\demo\\request folder and you can add your own scripts to this folder. Parameters should be added in the json message. Transcript is added to a table storage upon completion and you can specify an id in the json file to identify the results.

# [http://aka.ms/bcsandboxazure](http://aka.ms/bcsandboxazure)

Very much like getbc but used by container sandbox in Business Central. Defaults to the latest sandbox container and supports Azure SQL (from getbcext), but not Office 365 authentication.

This template doesn’t have any properties, which are not explained above.

# [http://aka.ms/getnavworkshopvms](http://aka.ms/getnavworkshopvms)

Used for spinning up multiple VMs and is really explained already here: [https://freddysblog.com/2017/11/05/aka-ms-getnavworkshopvms/](/2017/11/05/aka-ms-getnavworkshopvms/)

This template is slightly different from the other templates as it can spin up multiple VMs – and it should support both NAV and Business Central containers (including 15.x).

Hope this is helpful

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
