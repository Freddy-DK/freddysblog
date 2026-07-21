---
layout: post
title: "Publishing apps to a Business Central Online Environment"
date: 2021-01-27 08:11:18
categories: ["AL Development", "BcContainerHelper", "CI/CD"]
tags: ["App", "AppSource", "BcAuthContext", "Extension", "Install-BcAppFromAppSource", "Online", "PTE", "Publish", "Publish-BcContainerApp", "Publish-PerTenantExtensionApps"]
permalink: /2021/01/27/publishing-apps-to-a-business-central-online-environment/
---

There are 2 kinds of apps: AppSource Apps and Per Tenant Extensions (PTEs). These apps can be installed in two kinds of environments: Sandbox and Production. In Production, AppSource Apps are installed in the global scope and Per Tenant Extensions are installed in the Tenant Scope. In Sandbox environments you can also install apps to the development scope (like what VS Code does).

This blog post will describe how you can use BcContainerHelper 2.0.2 to install apps in any of those combinations.

The reason why these functions are placed in BcContainerHelper is that these functions are used in CI/CD pipelines. Some to install dependency AppSource Apps before compilation, some to publish PTEs to online production or sandbox environments after compilation, some to make the initial installation of xx PTEs to prepare for VS Code development.

This blog post will describe how to publish/install the apps, a later blog post will tie things together in a pipeline and more.

# Obtaining a BcAuthContext

Please read [this blog post](/2021/01/25/bcauthcontext/) in order to learn how to authenticate to your online Business Central environment and obtain a BcAuthContext.

In the samples below, I have read the $refreshToken from a keyvault and will use that to create an auth context and run the code to perform the scenario.

# Publishing PTEs to the Tenant Scope in a Production or Sandbox Environment

The output of my [sample HelloWorld PTE example](https://github.com/BusinessCentralApps/HelloWorld) can be downloaded here: [https://businesscentralapps.blob.core.windows.net/githubhelloworld/latest/apps.zip](https://businesscentralapps.blob.core.windows.net/githubhelloworld/latest/apps.zip).

You can publish and install the hello world sample to an environment called Sandbox in the tenant authenticated to by the refreshtoken by running this script:

```
$authContext = New-BcAuthContext -refreshToken $refreshToken
$environment = 'Sandbox'

$apps = "https://businesscentralapps.azureedge.net/githubhelloworld/latest/apps.zip"
Publish-PerTenantExtensionApps `
    -bcAuthContext $authContext `
    -environment $environment `
    -appFiles $apps
```

The .zip file will be unzipped and the apps in the .zip file will be installed in order of dependencies. You can also specify an array of app files/url or a comma separated string of app files/urls.

The output of this command could look like this:

![](/assets/images/2021/publishing-apps-to-a-business-central-online-environment/publish1.png)

Where the … represents a number of other apps installed in my environment.

If you run the same function again, it will fail. When using Publish-PerTenantExtensionApps you need to increase the version number in order to publish a new version of your app.

# Installing AppSource Apps in the Global Scope in a Production or Sandbox environment

My [sample BingMaps app](https://dev.azure.com/businesscentralapps/BingMaps) also exists in an AppSource version available in US and DK. The App ID for the BingMaps App is: **d22ff4b4-eff0-46cd-8d63-f28e7f646e33**.

At this time, the [App Management part of the Business Central Admin Center Api](https://docs.microsoft.com/en-us/dynamics365/business-central/dev-itpro/administration/administration-center-api#app-management) does not support installing AppSource Apps (it is in the backlog). It is however possible through a well known page in Business Central if you know the App ID. The URL for installing an app in a Business Central tenant is:

```
https://businesscentral.dynamics.com/[TenantID]/?noSignUpCheck=1&filter=%27ID%27%20IS%20%27[AppID]%27&page=2503 
```

where \[TenantID\] is your AAD Tenant ID and \[AppID\] is the App ID of the App you want to install. In BcContainerHelper 2.0.2 there is a function called Install-BcAppFromAppSource, which uses this page to install an app in a Business Central online environment.

The function uses a DLL called Microsoft.Dynamics.Framework.UI.Client.dll (like Run-TestsInBcContainer) to connect to the Business Central Client Services endpoint and emulate a user. This DLL is present in the Artifacts and on the DVD. Easiest way to ensure we have the right versions of his DLL and dependencies is to create a container with the same version as your online environment and connect from within that container.

The code to install the BingMaps.AppSource app in the production environment looks like this:

```
$authContext = New-BcAuthContext -refreshToken $refreshToken
$environment = "Production"
$appId = "d22ff4b4-eff0-46cd-8d63-f28e7f646e33"
$appName = "BingMaps.AppSource"
$env = Get-BcEnvironments -bcAuthContext $authContext |
    Where-Object { $_.Name -eq $environment }
$app = Get-BcPublishedApps -bcAuthContext $authContext -environment $environment |
    Where-Object { $_.Name -eq "Base Application" }
$artifactUrl = Get-BCArtifactUrl `
    -country $env.countryCode `
    -version $app.Version `
    -select Closest
New-BcContainer `
    -containerName proxy `
    -accept_eula `
    -artifactUrl $artifactUrl `
    -filesOnly
Install-BcAppFromAppSource `
    -containerName proxy `
    -bcAuthContext $authContext `
    -environment $environment `
    -appId $appId `
    -appName $appName `
    -allowInstallationOnProduction
Remove-BcContainer `
    -containerName proxy
```

Note that you need to add -allowInstallationOnProduction to Install-BcAppFromAppSource in order to Install on a production environment, else the function will fail. This is to make sure that you don’t install any apps in the wrong environment by accident.

Once the admin center supports this, I will refactor the Install-BcAppFromAppSource function and you won’t need to create the container anymore, meaning that a lot of the code above can be deleted. The output of the function today could look like this:

![](/assets/images/2021/publishing-apps-to-a-business-central-online-environment/publish2-1.png)

As you can see, majority of the output is creating a filesOnly container, which is much faster than creating a “real” container. More about what a filesOnly container can be used for in a later blog post.

If the app is already installed, this function will return. This function cannot be used to upgrade an app. The primary reason for this function is to be able to install dependencies in an online environment for your development or to install the Test Runner in a pipeline in order to run automated testing on an online environment.

I will modify this blog post when the container is no longer needed.

# Publishing apps to the Development scope in a Sandbox environment

Both AppSource apps and PTEs can be published to the Development scope in a Sandbox environment. This is what VS Code does when you press F5 and obviously you cannot publish to the development scope in a Production environment.

Publishing Apps to the development scope in a container is done using a function called Publish-BcContainerApp. This function is used extensively in pipelines, so I decided to add two parameters to this function (BcAuthContext and environment) and with these parameters, allow the function to publish to an online Business Central sandbox environment. If publishing to an online environment, you do not need to specify a container.

```
$authContext = New-BcAuthContext -refreshToken $refreshToken
$environment = "Sandbox"

$apps = "https://businesscentralapps.azureedge.net/githubhelloworld/latest/apps.zip"
Publish-BcContainerApp `
    -bcAuthContext $authcontext `
    -environment $environment `
    -appFile $apps
```

This should provide an output like this:

![](/assets/images/2021/publishing-apps-to-a-business-central-online-environment/publish3.png)

The function downloads the .zip file, unpacks, sort by dependencies, publishes one by one.

If you run the exact same function again, it will fail. You cannot publish the same app twice. If you recompile, it will succeed. You do not need to update the version number, it just needs to be a different package id (a new compilation)

One way you can use this function is when you setup a new sandbox environment and you have a series of PTEs to publish in order to start developing. You can publish the lot using this function and VS Code is ready for RAD afterwards.

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
