---
layout: post
title: "BcContainerHelper version 2.0.1"
date: 2021-01-24 08:00:41
categories: ["BcContainerHelper", "CI/CD"]
tags: ["AAD", "BcAuthContext", "BcContainerHelper", "CI/CD"]
permalink: /2021/01/24/bccontainerhelper-version-2-0-1/
---

Version 2.0.1 of BcContainerHelper was shipped with a number of very exciting new featuers. This blog post will list the features and I will have to document them elsewhere.

-   **Run-AlPipeline and Compile-AppInBcContainer with -failOn ‘error’** will now mark a build as succeeded with Warnings if there are warnings in the build.
-   **New-BcContainer with -FilesOnly** will create a container with no service tier, no sql database, no IIS, just all the files from a normal docker container copied to a container
-   **Compile-AppInBcContainer with -CopySymbolsFromContainer** will copy symbols from the container instead of downloading them from the service tier.
-   **New function New-BcAuthContext** is a new function to create an authorization context (with an AccessToken) for a Business Central online tenant. Invoke the function using -includeDeviceLogin and you will have an AccessToken in seconds. Store the refresh token in a keyvault and you will have access to a new AccessToken in your pipeline for 90 days.
-   **New function Renew-BcAuthContext** will renew the authorization context (if the AccessToken is about to expire). This function is used within functions in BcContainerHelper to ensure that the token is still valid even if the pipeline/function takes more than 60 minutes.
-   **Compile-AppInBcContainer with -bcAuthContext and -environment** will compile an app and use an online Business Central sandbox environment for downloading symbols.
-   **Publish-BcContainerApp with -bcAuthContext and -environment** will publish an app to an online Business Central sandbox environment.
-   **Import-TestToolkitToBcContainer with -TestRunnerOnly** will import Test Runner only (no framework, no libraries, no tests)
-   **Import-TestToolkitToBcContainer with -bcAuthContext and -environment** will import the test toolkit (-includeTestRunnerOnly or -includeTestFrameworkOnly – not Test Libraries and not Tests) to an online Business Central sandbox (Test Framework (Any, Assert, Variable Storage) will be available online within the next weeks – today only Test Runner is possible)
-   **Run-TestsInBcContainer with -bcAuthContext and -environment** will run tests in an online Business Central sandbox environment
-   **New function Install-BcAppFromAppSource** with -bcAuthContext and -environment uses page 2503 to install an AppSource App in an online Business Central environment. The implementation of this function will be changed to use the Admin Center API once that API supports installing AppSource Apps.
-   **Run-AlPipeline with -bcAuthContext and -environment** will run a full Al Pipeline using an online Business Central sandbox environment. This is useful for running pipelines for PTEs with dependencies on 3rd party extensions you don’t control.
-   **Get-BcArtifactUrl has two new parameters (before and after)** to only take artifacts created before or after a certain timestamp into consideration. Using Before or After makes the function considerably slower.
-   **New function Convert-BcAppsToRuntimePackages** used to convert an array of Apps to runtime packages for a given version (ArtifactUrl). Goal is to make it possible for partners to automate the sharing of 3rd party runtime apps for people using PTEs without sharing IP, wait for the blog post.
-   **Stacktrace** when an error happens within the scriptblock of Invoke-ScriptInBcContainer now contains a lot more useful information

I will create a number of blog posts describing these things in greater detail.

Also, based on feedback, I better spend some time going through old blog posts and ensure that the content matches current functionality AND create an actual reference documentation on current functionality instead of blog posts only…

# Feedback/Questions I am 100% sure that I will get

Why did I add functionality in the BcContainerHelper to work with online environments? Why not create these functions in a separate PowerShell module? and why not enable compiling and publishing without using containers?

_The reason that the new functions exists inside BcContainerHelper is that I did not want to create another PowerShell module and have BcContainerHelper have a dependency on that, simply due to time constraints and the reason for still using containers is that with containers, I can be sure that all the pre-requisite components are installed in the right locations and that I do not contaminate the host. In the end, the changes to the compile-, publish-, run-tests- etc. functions to work with online environments is very limited – it is really just the authentication and then calculating the URL to connect to._

Why is New-BcAuthContext not using ADAL (Active Directory Authentication Library) or MSAL (Microsoft Authentication Library)?

_ADAL is history, nobody should use ADAL today when creating new functionality. MSAL is future and I did investigate using MSAL, but I really didn’t want to add a dependency in BcContainerHelper at this time. I didn’t want to struggle with people having wrong versions of MSAL or having other things conflicting with MSAL. Also MSAL might go to dotnet core before I can and stop supporting dotnet framework altogether. So, I decided to go native. I might change the implementation in the future though._

Stay tuned for a number of upcoming blog posts.

**_Freddy Kristiansen_**  
Technical Evangelist
