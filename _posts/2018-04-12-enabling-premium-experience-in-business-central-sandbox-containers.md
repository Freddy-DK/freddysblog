---
layout: post
title: "Enabling Premium Experience in Business Central Sandbox Containers"
date: 2018-04-12 14:48:00
categories: ["Demo Environments"]
tags: ["Business Central Sandbox", "Docker", "NavContainerHelper", "new-navcontainer", "Premium"]
permalink: /2018/04/12/enabling-premium-experience-in-business-central-sandbox-containers/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

A few people have discovered that manufacturing, production and other functionality which only is available when using Premium Plan is not available when running a Business Central Sandbox Container.

The reason for this is, that this is controlled by the User Plan and by default the admin user has the essential plan. In Business Central, the plan is something that is controlled by what plan you purchase and you will not be able to add or modify records in the User Plan table.

Business Central Sandbox Containers are for development and test and of course we need to be able to develop and test against Premium – but it is also important to be able to run with essentials.

This blog post will describe how to assign the premium plan to your default super user in the NavContainer. It will also describe how you can create a number of test users and assign user groups and permissions to these users, so that you can test your app using the different users.

```
Username             User Groups              Permission Sets
EXTERNALACCOUNTANT   D365 EXT. ACCOUNTANT     D365 BUS FULL ACCESS
                     D365 EXTENSION MGT       D365 EXTENSION MGT
                                              D365 READ
                                              LOCAL
PREMIUM              D365 BUS PREMIUM         D365 BUS PREMIUM
                     D365 EXTENSION MGT       D365 EXTENSION MGT
                                              LOCAL
ESSENTIAL            D365 BUS FULL ACCESS     D365 BUS FULL ACCESS
                     D365 EXTENSION MGT       D365 EXTENSION MGT
                                              LOCAL
INTERNALADMIN        D365 INTERNAL ADMIN      D365 READ
                                              LOCAL
                                              SECURITY
TEAMMEMBER           D365 TEAM MEMBER         D365 READ
                                              D365 TEAM MEMBER
                                              LOCAL
DELEGATEDADMIN       D365 EXTENSION MGT       D365 BASIC
                     D365 FULL ACCESS         D365 EXTENSION MGT
                     D365 RAPIDSTART          D365 FULL ACCESS
                                              D365 RAPIDSTART
                                              LOCAL
```

and… – I will describe how to do this, whether you use Azure VMs, navcontainerhelper or docker run.

# Azure VMs

If you use [http://aka.ms/bcsandbox](http://aka.ms/bcsandbox) to create your Business Central Sandbox Container Azure VM, you will find two new options in the Azure Resource Manager template, which by default are set to yes.

[![](/assets/images/2018/enabling-premium-experience-in-business-central-sandbox-containers/fc48a-premium-1.png)](/assets/images/2018/enabling-premium-experience-in-business-central-sandbox-containers/fc48a-premium.png)

The first option is whether or not your admin user should be assigned a premium plan. The second is whether or not you want the setup to include the test users described above – that’s it – by default you get premium plan and test users, as of today.

# NavContainerHelper

If you are using New-NavContainer to create your Business Central Sandbox Container, you should upgrade to version 0.2.8.3.

Now you will have a new switch called assignPremiumPlan on New-NavContainer, use it like this:

```
New-NavContainer -accept_eula -assignPremiumPlan -containerName test -imageName microsoft/bcsandbox
```

Adding this option will assign the premium plan to your default admin user. Internally this just adds a record to the User Plan table.

In order to create the test users you will have to call a function called

```
Setup-NavContainerTestUsers containerName test -tenant default -password $securePassword
```

and specify the container and the password you want to use for the new users.

Internally, the Setup-NavContainerTestUsers downloads a codeunit with ID=50000, imports it and run an external function called CreateTestUsers with the password needed. After this you can delete or overwrite the codeunit, it is not needed anymore. The implementation might change.

If you want to see the codeunit, or if you need to modify the codeunit for your needs, you can download it at [http://aka.ms/createtestusersfob](http://aka.ms/createtestusersfob).

# Docker run

When you are using docker run to run your containers, you have a little more work to do.

First of all, you need to override the SetupNavUsers.ps1 by sharing a local folder to c:\\run\\my in the container and place a file called SetupNavUsers.ps1 in that folder with this content:

```
# Invoke default behavior
. (Join-Path $runPath $MyInvocation.MyCommand.Name)
 
Get-NavServerUser -serverInstance NAV -tenant default |? LicenseType -eq "FullUser" | % {
    $UserId = $_.UserSecurityId
    Write-Host "Assign Premium plan for $($_.Username)"
    sqlcmd -S 'localhostSQLEXPRESS' -d $DatabaseName -Q "INSERT INTO [dbo].[User Plan] ([Plan ID],[User Security ID]) VALUES ('{8e9002c0-a1d8-4465-b952-817d2948e6e2}','$userId')" | Out-Null
}
```

This will assign the premium plan to the admin user in the database.

In order to setup test users, you should download the codeunit from [http://aka.ms/createtestusersfob](http://aka.ms/createtestusersfob) import it using the classic development environment and run the CreateTestUsers function in the codeunit with the password you want to set for the users.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
