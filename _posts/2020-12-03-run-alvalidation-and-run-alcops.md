---
layout: post
title: "Run-AlValidation and Run-AlCops"
date: 2020-12-03 20:11:38
categories: ["AL Development", "BcContainerHelper", "CI/CD"]
tags: ["AppSource", "AppSourceCop", "CI/CD", "CodeCop", "PerTenantExtensionCop", "UICop", "Validation"]
permalink: /2020/12/03/run-alvalidation-and-run-alcops/
---

On December 1st, I co-hosted a webinar about how people can make sure that their app passes AppSource Validation. Until now, I have always been preaching that people should run CI/CD and include AppSourceCop validation in their CI/CD pipeline, which is supported in the Run-AlPipeline function. But…, as I prepared for the session, it became evident that it would be really helpful to have a function, which basically does exactly the same as our validation pipeline.

Run-AlValidation is the closest you get to that function. Run-AlValidation can perform the same steps as described on [https://aka.ms/CheckBeforeYouSubmit](https://aka.ms/CheckBeforeYouSubmit) given the right parameters.

The first preview version is in [PowerShell Gallery \| BcContainerHelper 1.0.15-preview275](https://www.powershellgallery.com/packages/BcContainerHelper/1.0.15-preview275).

Looking into the code, Run-AlValidation is really just an orchistration engine (like Run-AlPipeline), which invokes a number of the other functions in BcContainerHelper to create containers, extract the source code from apps, compile apps with appSourceCop validation, Publish Apps, Install or Upgrade Apps etc.

In its simplest form, it is executed like this:

```
$validationResults = Run-AlValidation `
    -licenseFile "C:\temp\license.flf" `
    -previousApps @( "path/url to your previous version of the .app files (or blank if this is the first)" ) `
    -apps @( "path/url to the new version of the .app files" ) `
    -validateCurrent `
    -countries "countries you want to validate against (f.ex. us,ca)" `
    -affixes "affixes you own (f.ex. fab,con)" `
    -supportedCountries "supported countries (f.eks. us,ca)"
```

and it will create an output in $validationResult that could look like:

```
2 errors found in nameOfYourApp.app on https://bcartifacts.azureedge.net/sandbox/17.1.18256.19561/us:
src\PageExtension\mypageext.al(12,20): error AS0011: The identifier 'myidentifier' must have at least one of the mandatory affixes 'fab,con'.
src\XMLPorts\Somefile.xmlport.al(1,17): error AS0090: The 'XmlPort' with ID '777777' and name 'MyXmlPrt-Ren-Fab' has been renamed to 'MyXmlPort-Ren-Fab'. Name
 changes are not allowed because it will break dependent extensions.

2 errors found in nameOfYourApp.app on https://bcartifacts.azureedge.net/sandbox/17.1.18256.19561/ca:
src\PageExtension\mypageext.al(12,20): error AS0011: The identifier 'myidentifier' must have at least one of the mandatory affixes 'fab,con'.
src\XMLPorts\Somefile.xmlport.al(1,17): error AS0090: The 'XmlPort' with ID '777777' and name 'MyXmlPrt-Ren-Fab' has been renamed to 'MyXmlPort-Ren-Fab'. Name
 changes are not allowed because it will break dependent extensions.
```

AppSourceCop is run for every country and output is collected. In the first version, errors when publishing and installing/upgrading will surface as exceptions and break the validation. They will not be added to the validation results. I will work on collecting all errors and add everything to validationResults.

If $validationResult is empty, then no validation errors was found.

# Parameters

Note that -previousApps and -apps can be an array of apps or a comma separated string with app filenames. The app filenames can also be .zip files with .app files inside – or they can be a url to an .app file or a .zip file.

There are other parameters. You can override some of the basic functions and you can add -skipverification to skip verification of code signing certificates. Note though that if you do this, the validation doesn’t fail if you forgot to sign your app.

Run-AlValidation also has a -vsixFile parameter to specify which AL Language Extension version to use.

# Run-AlCops

Run-AlCops is used from Run-AlValidation every time AppSourceCop needs to be run, but it can also be used seperately.Run-AlCops needs a running docker container to perform the validation in. The function can be called like:

```
$validationResults = Run-AlCops `
    -containerName my2 `
    -credential $credential `
    -enableAppSourceCop `
    -previousApps @( "https://businesscentralapps.blob.core.windows.net/helloworld-appsource/latest/apps.zip" ) `
    -apps @( "https://businesscentralapps.blob.core.windows.net/helloworld-appsource-preview/latest/apps.zip" ) `
    -affixes @("hw") `
    -supportedCountries @("us")
```

It can also be used for Per Tenant Extensions:

```
$validationResults = Run-AlCops `
    -containerName 'my2' `
    -credential $credential `
    -enablePerTenantExtensionCop `
    -apps @(
        "https://businesscentralapps.blob.core.windows.net/helloworld-preview/latest/apps.zip",
        "https://businesscentralapps.blob.core.windows.net/bingmaps-preview/latest/apps.zip"
        )
```

You can add Code Cop and UI Cop to the call if you like and you can specify a custom ruleset in the ruleSetFile parameter. If you are validating for AppSource, the default AppSource validation ruleset (can be found [here](https://bcartifacts.azureedge.net/rulesets/appsource.default.ruleset.json) is also applied).

# Get-AlLanguageExtensionFromArtifacts

In the latest containerhelper, I added this function to return a path to the .vsixFile from the artifacts specified. This way you can get the latest insider AL Language Extension using:

```
Get-AlLanguageExtensionFromArtifacts -artifactUrl (Get-BCArtifactUrl -select NextMajor -sasToken $insiderSasToken)
```

and you can transfer this to the -vsixFile parameter in Run-AlValidation (and Run-AlPipeline) method.

# Still failing validation?

If you successfully executed Run-AlValidation, with the same countries, the same versions, and with affixes, which are registered at Microsoft and you still fail validation, then you should create an issue here: [Issues · microsoft/navcontainerhelper (github.com)](https://github.com/microsoft/navcontainerhelper/issues) and include the script + the name of the app, the publisher name and the version number you are trying to publish. Then I will try to repro and fix the Run-AlValidation. Goal is, that everything that fails in AppSource validation should also fail in Run-AlValidation (given the right parameters)

I hope this is useful.

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
