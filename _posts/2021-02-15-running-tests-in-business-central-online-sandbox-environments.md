---
layout: post
title: "Running tests in Business Central Online Sandbox environments"
date: 2021-02-15 10:13:54
categories: ["AL Development", "BcContainerHelper", "CI/CD"]
tags: ["Run-TestsInBcContainer", "Running Tests", "Tests"]
permalink: /2021/02/15/running-tests-in-business-central-online-sandbox-environments/
---

For quite some time, it has been possible to run automated tests in Docker using the Run-TestsInBcContainer function and it is my strong belief that this is used by a lot of partners today. Since 17.2, the Test Runner is available in Online Business Central Sandbox environments for installation from AppSource. From Extension Marketplace, you can install the Test Runner, open page 130451 and run your test manually. With BcContainerHelper 2.0.4 or later, you can also run tests in online sandbox environments, this blog post explains how.

# Obtaining a BcAuthContext

Please read [this blog post](/2021/01/25/bcauthcontext/) in order to learn how to authenticate to your online Business Central environment and obtain a BcAuthContext.

In the samples below, I have read the $refreshToken from a keyvault and will use that to create an auth context and run the code to perform the scenario.

# Installing the Test Runner app

As already explained, the Test Runner app can be installed from Extension Marketplace to your tenant but this doesn’t really fly if you want to automate this. Using the Install-BcAppFromAppSource function (read more about it [here](/2021/01/27/publishing-apps-to-a-business-central-online-environment/)), you can install the Test Runner from AppSource automated (if you have the AppID):

```
Install-BcAppFromAppSource `
    -containerName $containerName `
    -bcAuthContext $bcauthcontext `
    -environment $environment `
    -appId '23de40a6-dfe8-4f80-80db-d70f83ce8caf' `
    -appName 'Test Runner'
```

Note that the Install-BcAppFromAppSource currently requires a container (filesonly is ok) in order to publish an app. The reason for this is that the function emulates a client opening the install app url on page 2503. As soon as the install app functionality is available in Admin Center API, I will modify the function to not require this.

and… to make pipelines easier, I have also added the bcAuthContext and environment parameters to Import-TestToolkitToBcContainer, which will make this function import the test runner to the online saas environment. Note that you have to specify includeTestRunnerOnly to the Import-TestToolkitToBcContainer in order to only install the Test Runner:

```
Import-TestToolkitToBcContainer `
    -containerName $containerName `
    -bcAuthContext $bcauthContext `
    -environment $environment `
    -includeTestRunnerOnly
```

Again the proxy container is needed in order to install the app.

# Running tests

After installing the Test Runner, we can install my Hello World app and the corresponding test app, by running this code:

```
$apps = "https://businesscentralapps.azureedge.net/githubhelloworld-preview/latest/apps.zip"
$testapps = "https://businesscentralapps.azureedge.net/githubhelloworld-preview/latest/testapps.zip"
Publish-BcContainerApp `
    -bcAuthContext $bcauthcontext `
    -environment $environment `
    -appFile @($testapps)
```

Which is using the Publish-BcContainerApp with auth context and environment to publish apps to the development scope as described [here](/2021/01/27/publishing-apps-to-a-business-central-online-environment/).

Now we can retrieve the extension id of our test app, by using:

```
$testAppId = (Get-BcInstalledExtensions `
    -bcAuthContext $bcauthContext `
    -environment $environment | Where-Object { 
        $_.displayname -eq "Default Test App Name"
    }).id
```

Now we can run the tests sing the Run-TestsInBcContainer function.

```
Run-TestsInBcContainer `
    -containerName $containerName `
    -bcAuthContext $bcauthContext `
    -environment $environment `
    -extensionId $testAppId `
    -detailed
```

Like when importing the test toolkit, a proxy container is needed to run the tests. A FilesOnly container will do. Output of the function should be something like:

```
  Codeunit 50133 HelloWorld Test Success (0.627 seconds)
    Testfunction TestHelloWorldMessage Success (0.627 seconds)
```

# Test Framework and Test Libraries

The Test Framework apps consists currently of 3 apps:

-   Any
-   Asser
-   Variable Storage

These apps are used frequently when running tests. While writing this blog post, these apps are not available in online Business Central sandbox environments, but within a few releases, they will be.

This means that if your test apps have a dependency on the test framework, you will have to refactor them in order to run the tests now – or wait few months to be able to run these tests in an online Business Central Sandbox environment.

The Test Librarires apps consists of the Test Framework apps + 2 more:

-   System Application Test Library
-   Tests-TestLibraries

These apps are not available in online Business Central sandbox environments and there is currently no plans of adding them.

This means that if your test apps have a dependency on the test libraries, you will not be able to run these tests in an online Business Central Sandbox environment.

# Comes with limitations

As you can read, running tests in online Business Central Sandbox Environments comes with limitations and is not a substitute for running tests in Docker, but there are some cases where it comes in handy, especially if you are running tests on a Per Tenant Extension, which has a dependency on 3rd party apps, which you don’t have available in Docker.

# Why?

Maybe you are thinking: why incorporate the ability to run tests in an online environment in the standard Run-TestsInBcContainer function? why not create a new function, which is targeted online? Why require a container?

The primary reason is really that the only way I can run tests in an online environment today is using the AL Test Runner and the only way I can use that is to use Client Services DLL and connect to the AL Test Runner and execute tests. The client services DLL and other good things, are available in the artifacts (and therefore in a container) and 99% of the code used to run tests in a container is used to run tests in online environments as well.

I might work on removing the need of having a container, but in the interest of time, I decided to ship a version, which runs the code inside a container and starting a FilesOnly container is really fast.

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
