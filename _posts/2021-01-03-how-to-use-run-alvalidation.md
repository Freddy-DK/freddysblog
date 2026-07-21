---
layout: post
title: "How to use Run-AlValidation"
date: 2021-01-03 13:04:26
categories: ["AL Development", "BcContainerHelper", "CI/CD"]
tags: ["BcContainerHelper", "PowerShell", "Run-AlValidation"]
permalink: /2021/01/03/how-to-use-run-alvalidation/
---

When releasing the first version of Run-AlValidation in BcContainerHelper, i did a quick blog post about the function [here](/2020/12/03/run-alvalidation-and-run-alcops/). This blog post serves to explain some common scenarios of how to run the function. The blog post will also explain the parameters of Run-AlValidation a bit more in depth, for people to have a good chance of using the function before submitting for AppSource Validation.

![](/assets/images/2021/how-to-use-run-alvalidation/image.png)

_**Note**: The parameters and the functionality described in this blog post requires BcContainerHelper 1.0.19 or later (while writing this blog post 1.0.19 preview is available supporting this)_

# Important Parameters

_**Note**: This is the list of parameters you must consider to get a trustworthy result out of Run-AlValidation._

**licensefile** (required) is the path or a secure URL to your developer license file.

**installApps** (optional) is the list of dependencies to 3rd party apps or unchanged apps in your submission. If your submission contains x library apps and some of these library apps are submitted in unchanged versions, then these apps are still needed (as they are a dependency) and must be specified here. If you specify the same app in previousApps and Apps – it will fail when trying to upgrade.

**previousApps** (optional) is the list of the previously successfully validated version of the apps. This is the version of the apps already in AppSource, including your main app and all modified library apps. If the new version of the app is the same as the previous version, the app belongs in installApps.

**Apps** (required) is the list of apps you want to run validation on. Include both the main app and all modified library apps here. If your submission includes both changed and new apps, then all changed and new apps should be specified in Apps, but of course only the previous versions of the changed apps should be specified in previousApps.

**affixes** (required) is a list of the affixes used in your app, following the guidelines [here](https://docs.microsoft.com/en-us/dynamics365/business-central/dev-itpro/compliance/apptest-prefix-suffix). In order to pass AppSource Validation, you need to make sure that these affixes are registered with Microsoft on your MPN ID and publisher name. This parameter can either be an array of strings or a comma separated string containing the affixes.

**countries** (required) is a list of the countries for which you want to validate your app. This parameter can either be an array of strings or a comma separated string containing the name or the country code of the countries.

# Parameters indicating which version to validate against?

_**Note**: By default, Run-AlValidation will validate against the current version of Business Central AND then version indicated in your app.json files as your minimum common dependency. This is the same mechanism used by Appsource validation and if you want to use this, then you should NOT specify any specific validation parameter._

**ValidateCurrent** (optional) is a switch you can include if you want to validate your app against the current version of Business Central specifically. During AppSource validation we always validate against current version.

**ValidateVersion** (optional) is a parameter which you can set to a specific version if you want to only validate against this version. During AppSource Validation we determine the minimum common dependency from all apps and validate against that.

**validateNextMinor** (optional) is a switch which you can include in order to solely or additionally validate against next minor. In order to validate against next minor, you need to specify a sasToken as well.

**validateNextMajor** (optional) is a switch which you can include in order to solely or additionally validate against next minor. In order to validate against next minor, you need to specify a sasToken as well.

**sasToken** (optional) is a shared access service token, which provides access to insider builds. The token for insider artifacts is published on [Collaborate](https://aka.ms/collaborate) under packages and is called [Working With Business Central Insider Builds](https://partner.microsoft.com/en-us/dashboard/collaborate/packages/9387).

# Additional Parameters

_**Note**: These parameters are all optional and seldomly needed. They are only there for you to test out validation runs in specific ways._

**containerName** (optional) specifies the containerName of the docker container used for every step of the validation. For every country a new container will be created, all using the same containerName and removing the previous container. Default is the defaultContainerName configuration, which is bcserver.

**imageName** (optional) specifies the name in which you want to cache images for faster subsequent turnaround time. Specifying this parameter will almost always make the validation go slower, because validation uses the latest artifacts, and they have probably changed since your last run.

**credential** (optional) specifies the credentials used for creating and using the containers. If you do not specify credentials, a random password will be generated and used with the admin username.

**memoryLimit** (optional) is per default set to 8Gb which is the same as used in AppSource validation. In seldom cases, it can be necessary to increase this value.

**failOnError** (optional) is a switch which can be included in order for validation to fail early. Default behavior is to collect all validation errors and display them all at the end of validation.

**includeWarnings** (optional) is a switch you can include to see AppSourceCop warnings. AppSource submissions do not fail due to warnings, but warnings might become errors in a later version.

**skipVerification** (optional) is a switch you can use to skip verification of whether or not your app files are signed with a code signing certificate. Note, In order to pass AppSource validation, your apps must be signed with a code signing certificate, so skipping verification might not be the best idea.

**skipUpgrade** (optional) is a switch you can use to skip Upgrade test. This switch comes in handy if the previous version of your app doesn’t install on the version of Business Central being validated against. f.ex. if your old version was for 17.5 and doesn’t install on 18.0. This is a valid scenario in AppSource validation as well.

**skipConnectionTest** (optional) is a switch you can use to skip the Connection Test. By default AppSource Validation uses Connection Test to check whether your app causes Business Central to become unresponsive. Connection Test will open a Rolecenter and a few pages. During AppSource validation, we will NOT fail you if connection test fails, we will flag the app for manual test validation (which will cause validation to take more time). If manual tests fail, we will fail you.

**skipAppSourceCop** (optional) is a switch you can use to bypass AppSourceCop if you just want to test the other things. Note, that AppSource validation requires AppSourceCop and you cannot request that we include this parameter when validating your app.

**supportedCountries** (obsolete) this parameter is no longer used. It used to be a list of the countries in which your app will be available. This parameter can either be an array of strings or a comma separated string containing the name or the country code of the countries.

Beside this, there are some parameters to override the container creation and other things. These are rarely used, but can be specified if for some reason your setup requires this and you cannot specify what you need in BcContainerHelper configuration.

# Examples

_The following is a few examples on how to run validation for your app depending on the scenario you have. Remember that multiple apps can be specified as an array of strings (appfiles) or one comma separated string._

## Example #1 – a new submission – a single app

Run-AlValidation \`
    -licenseFile $licenseFile \`
    -apps "C:\\apps\\my.app" \`
    -affixes "fab" \`
    -countries "us"

Will validate **c:\\apps\\my.app** requiring affix **fab** for country **us** on the **current** version of Business Central and the version **indicated by app.json**.

## Example #2 – an update to your app – a single app

Run-AlValidation \`
    -licenseFile $licenseFile \`
    -previousApps "c:\\prevApps\\my.app" \`
    -apps "C:\\apps\\my.app" \`
    -affixes "fab" \`
    -countries "us"

Will validate **c:\\apps\\my.app** as an update to **c:\\prevApps\\my.app** requiring affix **fab** for country **us** on the **current** version of Business Central and the version **indicated by app.json**.

## Example #3 – a new submission – an app with a library app

Run-AlValidation \`
    -licenseFile $licenseFile \`
    -apps @("C:\\apps\\my.app", "c:\\apps\\mylib.app") \`
    -affixes @("fab", "con") \`
    -countries @("us", "ca")

Will validate **c:\\apps\\my.app** with a library app **c:\\apps\\mylib.app** requiring affixes **fab** or **con** for countries **us** and **ca** on the **current** version of Business Central and the version **indicated by app.json**.

## Example #4 – a update to your apps – an app with a library app

Run-AlValidation \`
    -licenseFile $licenseFile \`
    -previousApps @("c:\\prevApps\\my.app", "c:\\prevApps\\mylib.app") \`
    -apps @("C:\\apps\\my.app", "c:\\apps\\mylib.app") \`
    -affixes @("fab", "con") \`
    -countries @("us", "ca")

Will validate **c:\\apps\\my.app** with a library app **c:\\apps\\mylib.app** as an update to **c:\\prevApps\\my.app** and **c:\\prevApps\\mylib.app** requiring affixes **fab** or **con** for countries **us** and **ca** on the **current** version of Business Central and the version **indicated by app.json**.

## Example #5 – a update to your apps – an app with a library app and a third party dependency app

Run-AlValidation \`
    -licenseFile $licenseFile \`
    -installApps @("c:\\thirdparty\\licensing.app") \`
    -previousApps @("c:\\prevApps\\my.app", "c:\\prevApps\\mylib.app") \`
    -apps @("C:\\apps\\my.app", "c:\\apps\\mylib.app") \`
    -affixes @("fab", "con") \`
    -countries @("us", "ca")

Will validate **c:\\apps\\my.app** with a library app **c:\\apps\\mylib.app** and a dependency to **c:\\thirdparty\\licensing.app** as an update to **c:\\prevApps\\my.app** and **c:\\prevApps\\mylib.app** requiring affixes **fab** or **con** for countries **us** and **ca** on the **current** version of Business Central and the version **indicated by app.json**.

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
