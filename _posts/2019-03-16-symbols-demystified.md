---
layout: post
title: "Symbols demystified"
date: 2019-03-16 15:12:52
categories: ["AL Development", "NavContainerHelper", "Not Archived"]
tags: ["AL", "Docker", "NavContainerHelper", "Symbols", "VS Code"]
permalink: /2019/03/16/symbols-demystified/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

Over the last months, I have received a lot of questions on Symbols:

-   Why does Compile-AppInNavContainer get different symbols than VS Code?
-   When I import new objects to Business Central, why don’t I get the symbols?
-   Why do I have the Assert Codeunit twice?
-   Why can’t Business Central find the Assert Codeunit?

Just to name a few.

This blog post will try to explain how symbols works and give an answer these questions, by showing some samples.

# What are symbols

You can look at the symbols as the application programming interface to the system, the application and the test objects.

Try to create a very simple container called mytest:

$credential = New-Object pscredential -ArgumentList 'admin', (ConvertTo-SecureString -String 'P@ssword1' -AsPlainText -Force)
$imageName = "mcr.microsoft.com/businesscentral/onprem:dk-ltsc2019"
New-NavContainer -accept\_eula -imageName $imageName -containerName mytest -auth NavUserPassword -Credential $credential -updateHosts

Then run the command

Get-NavContainerAppInfo -containerName mytest -symbolsOnly

and you should see something like:

ServerInstance : MicrosoftDynamicsNavServer$NAV
AppId : 9512e682-f74f-4213-8d3a-d7e19f3f0260
Name : Application
Publisher : Microsoft
Version : 13.4.28874.0
ExtensionType : ModernDev
Scope : Global

ServerInstance : MicrosoftDynamicsNavServer$NAV
AppId : 51e87475-45a5-4d70-b77f-2749a4c24270
Name : Test
Publisher : Microsoft
Version : 13.4.28874.0
ExtensionType : ModernDev
Scope : Global

ServerInstance : MicrosoftDynamicsNavServer$NAV
AppId : 8874ed3a-0643-4247-9ced-7a7002f7135d
Name : System
Publisher : Microsoft
Version : 13.0.12929.0
ExtensionType : ModernDev
Scope : Global

You get the same if you make a standard installation of Business Central and run Get-NavAppInfo -symbolsOnly, this is nothing special for Containers.

Let’s start VS Code, run AL Go!, create a new extension, modify launch.json and download the symbols. You will get two symbol files:

![symbols1](/assets/images/2019/symbols-demystified/symbols1.png)

Running your extension should say Hello World when the Customer List opens because you have extended the Customer List Page – all of this is probably well known.

**The symbols we have matches the application. If we make any changes to the application, either in C/SIDE or by importing objects using PowerShell, our symbols will no longer match the application and we will get runtime errors and there is NO way we can make the symbols match the application.**

# Test Symbols

You might have noticed that the container also had Test symbols in the list of Symbols.

If you try to add a reference to the test symbols in app.json by adding:

"test": "13.0.0.0"

and Download the symbols. You will now have:

![symbols2](/assets/images/2019/symbols-demystified/symbols2.png)

Change the OnOpenPage trigger to:

trigger OnOpenPage();
var
    Assert: Codeunit Assert;
begin
    Message('App published: Hello world');
    Assert.AreEqual(2, 2, 'Works!');
end;

Build the app and run using F5 and you should see an error in the debugger. If you press F5 to that error, you should see the error in the UI:

![symbols3](/assets/images/2019/symbols-demystified/symbols3.png)

**The reason for this error is, that the Test objects are NOT in the database by default. Referencing the symbols just means that you assume that the objects are in the database. In this case, we have a difference between the objects in the database and the symbols.**

Now run this command in PowerShell:

Import-TestToolkitToNavContainer -containerName mytest -sqlCredential $credential -includeTestLibrariesOnly

and then retry F5. The runtime error is gone.

The symbols we are referencing matches the objects in the database – well – almost. We actually have symbols for all the tests as well and the above command only imports the Test Libraries (which includes Codeunit Assert).

**As long as you do not need to change objects in C/AL – you shouldn’t run into other issues than this and everything should run smooth.**

# EnableSymbolLoading

If you need to import a .fob file or in other ways change the objects in the container with anything that will make changes to the application, you will need to start the container with the **\-enableSymbolLoading** switch.

Starting a Container with **EnableSymbolLoading** will set the **EnableSymbolLoadingAtserverStartup** to **true** in CustomSettings.config. It will also add **generatesymbolreference=1** to all calls to **C/SIDE** in the NavContainerHelper in order to update symbols when you change objects.

**Running with EnableSymbolLoadingAtServerStartup set to true will cause the service tier to ignore the published Symbols and instead create a new set of symbols based on the actual content of the objects in the database.**

The generated symbols will get the application version number from the database, and if VS Code or other processes requests to download symbols, the Service Tier will give out the generated symbols instead of the published symbols.

**Note that there is an error in Business Central On Prem (at least until CU3), where the version number of application symbols are wrong when using EnableSymbolLoading. This means that you will have two sets of symbols in the database and Compile-AppInNavContainer actually would download the published Application symbols instead of the generated symbols.  
**

Let’s try to recreate the container with EnableSymbolLoading:

$credential = New-Object pscredential -ArgumentList 'admin', (ConvertTo-SecureString -String 'P@ssword1' -AsPlainText -Force)
$imageName = "mcr.microsoft.com/businesscentral/onprem:dk-ltsc2019"
New-NavContainer -accept\_eula -imageName $imageName -containerName mytest -auth NavUserPassword -Credential $credential -updateHosts -enableSymbolLoading -includeTestToolkit -includeTestLibrariesOnly

Then switch back to VS Code and re-download the symbols. The first thing you might see is, that now (due to the error described earlier) you might have 2 sets of application symbols:

![symbols4](/assets/images/2019/symbols-demystified/symbols4.png)

The generated set is 13.0.28871.0 – not the same version as the published app. Remove all the symbols and download the symbols again.

If you are using the latest NavContainerHelper, you will get an error saying:

The request for path /NAV/dev/packages?publisher=Microsoft&appName=Test&versionText=13.0.0.0 failed with code NotFound. Reason: No published package matches the provided arguments.

This is because NavContainerHelper from version 0.5.0.5 will unpublish the Application and the Test symbols when running with EnableSymbolLoading – they are not intended to be used.

If you are using NavContainerHelper 0.5.0.4 (or earlier), you will not get an error when downloading symbols, but when you try to build the application you will get:

error AL0265: Reference to object Assert is ambiguous. Make sure you are not referencing packages containing objects with the same names.

**Using EnableSymbolLoading will rebuild ALL symbols into the Application symbols including the Test Objects. This means that** **you will have the Test symbols twice if you also reference the Test Symbols App and download the symbols from this.**

Remove the test reference from app.json and you should be able to download symbols, build the app and run the app.

# So, what is the recommendation?

Here’s my recommendation:

-   Use the latest NavContainerHelper (Fixes the issue with the wrong Application Version and unpublishes the not-needed symbols apps from the Container)
-   Use EnableSymbolLoading only if you are going to make C/AL changes to the database.
-   Do NOT include a reference to test symbols if you are using EnableSymbolLoading.

With that, symbols should work as expected.

**Note**, that while writing this, the CI/CD template repository and scripts are hardcoded to use EnableSymbolLoading. This will be changed and made optional.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
