---
layout: post
title: "Run-AlPipeline"
date: 2020-10-08 22:05:28
categories: ["AL Development", "BcContainerHelper", "CI/CD"]
tags: ["BcContainerHelper", "Continuous Integration", "Pipeline"]
permalink: /2020/10/08/run-alpipeline/
---

WARNING: Very boring long blog post ahead…

Run-AlPipeline is a new function in BcContainerHelper. It has been in preview for a number of releases while being worked on, and a number of partners have already tried to use it. My apologies for changing things under your feet, but to my defense – I did write that the function was in preview.

With version 1.0.8 of BcContainerHelper, Run-AlPipeline is ready for real-life usage… (I think)

# Goal

The goal of this function is that you can use the function as a build mechanism for most PTEs (Per Tenant Extensions) and for some AppSource apps. The function does not make any attempt to allow base app modifications, on-premises apps nor to support older versions of Business Central or NAV.

You might be able to make it work for a lot of different things, but then again, you might find yourself fighting assumption because the function is targeting Business Central online only.

You can run the Run-AlPipeline function on any machine, with Docker installed and BcContainerHelper 1.0.8 or later. The function doesn’t require any metadata, no settings files, no environment variables, all properties are transferred in parameters. This means that this function can be used locally, on Azure DevOps, in Github actions or on Gitlab.

So, even though the function is called Run-AlPipeline, there isn’t a pipeline definition anywhere that gets run, it is an end 2 end thing.

# Sample run

Before going in detail with all parameters of the Run-AlPipeline function, let me show a sample run. I have cloned this repo [https://dev.azure.com/businesscentralapps/BingMaps](https://dev.azure.com/businesscentralapps/BingMaps) to my local box, after which I run this:

```
Run-AlPipeline `
    -pipelineName "BingMaps" `
    -licenseFile "c:\temp\license.flf" `
    -baseFolder "C:\Users\freddyk\Documents\GitHub\BusinessCentralApps\BingMaps" `
    -appFolders @("app") `
    -testFolders @("test") `
    -installTestFramework `
    -enablePerTenantExtensionCop `
    -enableUICop
```

This function causes an output, which you can watch [here](https://bcdocker.blob.core.windows.net/public/sampleoutput.txt).

Examining the output will give you an idea of what this is. It will create a container, compile the apps specified, publish the apps, run the tests and remove the container.

But this is of course the simplest of runs. You can have multiple app folders, which then would be compiled and published in order of dependencies. You can have dependent apps, that needs to be installed, you can have previous versions of the apps and want to test the upgrade process and so on and so on.

# Real life

The function is already used in all my pipelines, including the Hello World: [https://dev.azure.com/businesscentralapps/HelloWorld](https://dev.azure.com/businesscentralapps/HelloWorld) and the AppSource version of Hello World: [https://dev.azure.com/businesscentralapps/HelloWorld.AppSource](https://dev.azure.com/businesscentralapps/HelloWorld.AppSource).

The AppSource Version is setup for AppSourceCop. breaking change validation etc. and the PTE version is setup for Per Tenant Extension Cop and the stuff needed for PTE.

Please excuse the very long list of boring parameters…

# Parameters (in order of likelihood to be used)

## \-pipelineName

The name of the pipeline or project.

## \-artifact

The description of which artifact to use. This can either be a URL (from Get-BcArtifactUrl) or in the format _storageAccount/type/version/country/select/sastoken_, where these values are transferred as parameters to Get-BcArtifactUrl. Default value is _///us/current_, which will give you the current public US version.

## \-licenseFile

Unless you have a PTE with no tests, no will need a licensefile. Using the test framework will require a license file and so will your appSource app. The license file does not have to reside inside the base folder and can be a secure url to a license file, which then will be downloaded and used. The recommended way is to have this secure url in an Azure KeyVault for use locally and in pipelines.

## \-baseFolder

The baseFolder serves as the base Folder for all other parameters including a path (appFolders, testFolders, testResultFile, outputFolder and packagesFolder). The base folder doesn’t need to be the immediate parent of subfolders, but this folder is shared with the container, responsible for building and testing the app and the container needs access to all these folders. You should not root level folders as folder sharing with containers tend to get slower if the folder shared contains a high number of files and directories.

## \-appFolders

appFolders is an array of folders, containing app sourcecode that needs to be compiled and published. It can also be a comma-seperated string of app folders, the function will auto-convert to an array.

Run-AlPipeline will sort the app Folders after dependencies and will compile, sign and publish the dependent apps first and the depending apps last.

## \-testFolders

testFolders is an array of folders, containing test app sourcecode that needs to be compiled and published. It can also be a comma-seperated string of test app folders, the function will auto-convert to an array.

Run-AlPipeline will sort the test app Folders after dependencies and will compile, publish and test the dependent apps first and the depending apps last.

## \-installApps

_installApps_ is an array of third party .app or .zip files. It can also be a comma-separated string of .app or .zip files, which will be converted into an array. Apps will be installed in the order they appear in the array. A .zip file may contain multiple .app files and they will be sorted and installed dependencies first (for each .zip file). If the apps in the .zip file are runtime packages, they will be installed in alphabetic order.

The apps provided in installApps are apps, on which your apps depend. Apps that needs to be installed before compiling and publishing your apps.

Default is an empty array and as such no third party apps will be installed.

## \-previousApps

_previousApps_ is an array of .app or .zip files. All .zip files are unzipped and all app files are sorted by dependencies. Runtime packages are not allowed. The apps in this list are previous versions of the apps you are compiling and will be used for two things.

If you have enabled AppSourceCop, the previous version of the app you are compiling will be placed in the packages folder and information about this app is placed in appSourceCop.json to signal to the compiler to check for breaking changes, which are not allowed in AppSource apps.

The previous versions of the apps will also be installed before the newly compiled apps will be installed and the newly compiled apps will perform an upgrade instead of an installation.

## \-enableCodeCop

Include this switch to include Code Cop Rules during compilation.

## \-enableAppSourceCop

Only relevant for AppSource apps. Include this switch to include AppSource Cop during compilation.

If previousApps is defined, then the appSourceCop will also check for breaking changes compared to the old version. If AppSourceCopMandatoryAffixes or AppSourceCopSupportedCountries is defined, then that will also be added to the AppSourceCop validation.

Apps submitted for validation will be required to pass AppSourceCop with previousApps and the reserved affixes for the publisher given in app.json.

## \-enablePerTenantExtensionCop

Only relevant for Per Tenant Extensions. Include this switch to include Per Tenant Extension Cop during compilation.

## \-enableUICop

Include this switch to include UI Cop during compilation.

## \-AppSourceCopMandatoryAffixes

Only relevant for AppSource Apps when AppSourceCop is enabled. This needs to be an array (or a string with comma separated list) of affixes used in the app.

## \-AppSourceCopSupportedCountries

Only relevant for AppSource Apps when AppSourceCop is enabled. This needs to be an array (or a string with a comma seperated list) of supported countries for this app.

## \-installTestFramework

Include this switch to include the test framework in the container before compiling apps and test apps. The Test Framework includes the following apps: _Microsoft Any, Microsoft Library Assert, Microsoft Library Variable Storage_ and _Microsoft Test Runner_.

## \-installTestLibraries

Include this switch to include the test libraries in the container before compiling apps and test apps. The Test Libraries includes all the Test Framework apps and the following apps: _Microsoft System Application Test Library_ and _Microsoft Tests-TestLibraries_

## \-installPerformanceToolkit

Include this switch to install test Performance Test Toolkit. This includes the apps from the Test Framework and the _Microsoft Business Central Performance Toolkit app_

Currently there is no good way of running performance tests using PowerShell. We will be working on adding this functionality in order to allow people to run performance regression tests during CI or daily builds.

## \-doNotRunTests

Include this switch to indicate that you do not want to execute tests. Test Apps will still be published and installed, test execution can later be performed from the UI.

## \-testResultsFormat

In what format do you want the testResults file. Possible values are _XUnit_ or _JUnit_. Both formats are XML based test result formats. _JUnit_ is default and contains some properties about memory and installed apps, which due to format limitations cannot be included in XUnit. This parameter is ignored if _doNotRunTests_ is included.

## \-testResultsFile

Filename in which you want the test results to be written. Default is _TestResults.xml_, meaning that test results will be written to this filename in the base folder. This parameter is ignored if _doNotRunTests_ is included.

If you do not include any test folders or if you specify _\-doNotRunTests_, then test execution will be skipped and the file will be deleted.

## \-containerName

This is the containerName going to be used for the build/test container. If not specified, the container name will be the pipeline name followed by _\-bld_. For builds on build agents, there is really no need to specify a containerName. For local builds, where you might want to keep the container after the pipeline has finished, you might want to specify this.

## \-imageName

The value of _imageName_ can have two meanings. If you specify imageName and set the artifact parameter to an empty string, then Run-AlPipeline will use this imageName as a docker image and run that. Since the artifact parameter is defaulted to ///us/current, you will need to set it to “” in order to use the image.

If you specify _imageName AND artifact_, then Run-AlPipeline will (as New-BcImage does) check that the image specified by imageName is made on the artifact specified and if that is the case, it will run the image. If the image doesn’t exist, it will be build and next time around running the same function, it will reuse the image.

The default value of imageName is “my”, which means that in the above example, it will create an image called _my:sandbox-17.0.17126.17434-us-mt_ and run that. The tag is _type-version-country_ followed by _\-mt_ if multitenancy. The default value is fine for self-hosted agents or local computers. If you are using Azure Hosted agents, which doesn’t benefit from cached images, set imageName to an empty string.

Over time a lot of artifacts and images will be downloaded and build and you can use Flush-_ContainerHelperCache -KeepDays 7_ to remove all artifacts and images that hasn’t been used for the last 7 days.

## \-appBuild

Build number for build. Will be stamped into the build part of the app.json version number property.

## \-appRevision

Revision number for build. Will be stamped into the revision part of the app.json version number property.

## \-credential

These are the credentials used for the container. If not provided, the Run-AlPipeline function will generate a random password and use that. The generated password will be printed out during pipeline execution. If you use -keepContainer to keep the container after pipeline execution, it is highly recommended that you specify credentials yourself.

## \-codeSignCertPfxFile

A secure url to a code signing certificate for signing apps. Apps will only be signed if _useDevEndpoint_ is NOT specified.

## \-codeSignCertPfxPassword

Password for the code signing certificate specified by _codeSignCertPfxFile_. Apps will only be signed if _useDevEndpoint_ is NOT specified.

## \-enableTaskScheduler

Include this switch if the Task Scheduler should be running inside the build/test container, as some app features rely on the Task Scheduler.

## \-assignPremiumPlan

Include this switch if the primary user in Business Central should have assign premium plan, as some app features require premium plan.

## \-tenant

Normally all deployments, compilations, deployments and tests are running in the default tenant. If you specify a tenant name, a tenant with this name will be created and used for the entire process.

## \-memoryLimit

MemoryLimit is default set to 8Gb. This is fine for compiling small and medium size apps, but if your have a number of apps or your apps are large and complex, you might need to assign more memory.

## \-createRuntimePackages

Include this switch if you want to create runtime packages of all apps. The runtime packages will also be signed (if certificate is provided) and copied to artifacts folder.

## \-buildArtifactFolder

If this folder is specified, the build artifacts will be copied to this folder. All apps to an apps folder. Runtime packages to a runtimepackages folder and test apps to a testapps folder. Test results will also be copied to the build artifacts folder.

## \-azureDevOps

Include this switch if you want compile errors and test errors to surface directly in Azure Devops pipeline.

## \-useDevEndpoint

Including the _useDevEndpoint_ switch will cause the pipeline to publish apps through the development endpoint (like VS Code). This should ONLY be used when running the pipeline locally and will cause some changes in how things are done.

When using dev Endpoint, apps will not be placed in the appOutputFolder and the appPackagesFolder is also not used for all symbols. Instead, every app will have it’s own .alPackages folder and symbols will be downloaded there (exactly as VS Code does it). The output will also be placed where VS Code places it.

_Normally_ apps are compiled and placed in the output folder. After all apps are compiled, all apps will be signed, the previous versions will be installed (if specified) and then the newly compiled apps will be published and installed/upgraded. With _useDevEndpoint_, every app is compile, published and installed one by one.

The biggest advantage of using -useDevEndpoint is, that you can open the app in VS Code and start debugging right away – everything is set for RAD. Remember to specify _keepContainer_ and _updateLaunchJson_ if this is what you want to do.

## \-updateLaunchJson

Including the _updateLaunchJson_ switch causes the launch.json file in each project to be updated with container information to be able to start debugging right away.

## \-keepContainer

Including the _keepContainer_ switch causes the container to not be deleted after the pipeline finishes.

## \-packagesFolder

This parameter is only relevant when NOT using useDevEndpoint. This is the folder (relative to base folder) where symbols are downloaded and compiled apps are placed. This way each app compilation reuses the symbols downloaded by the prior compilation and the compiled app will also be available. Default is .packages and you will likely not need to change this.

## \-outputFolder

This parameter is only relevant when NOT using useDevEndpoint. This is the folder (relative to base folder) where compiled apps are placed. Default is .output and you will likely not need to change this.

# Advanced – the parameters below are typically not changed

## \-DockerPull

Override function parameter for docker pull. The default scriptblock for this parameter is:

```
{ Param($imageName)
    docker pull $imageName
}
```

## \-NewBcContainer

Override function parameter for New-BcContainer. The default scriptblock for this parameter is:

```
{ Param([Hashtable]$parameters)
    New-BcContainer @parameters
    Invoke-ScriptInBcContainer $parameters.ContainerName -scriptblock {
        $progressPreference = 'SilentlyContinue'
    }
}
```

## \-CompileAppInBcContainer

Override function parameter for Compile-AppInBcContainer. The default scriptblock for this parameter is:

```
{ Param([Hashtable]$parameters)
    Compile-AppInBcContainer @parameters
}
```

## \-PublishBcContainerApp

Override function parameter for Publish-BcContainerApp. The default scriptblock for this parameter is:

```
{ Param([Hashtable]$parameters)
    Publish-BcContainerApp @parameters
}
```

## \-SignBcContainerApp

Override function parameter for Sign-BcContainerApp. The default scriptblock for this parameter is:

```
{ Param([Hashtable]$parameters)
    Sign-BcContainerApp @parameters
}
```

## \-RunTestsInBcContainer

Override function parameter for Run-TestsInBcContainer. The default scriptblock for this parameter is:

```
{ Param([Hashtable]$parameters)
    Run-TestsInBcContainer @parameters
}
```

## \-GetBcContainerAppRuntimePackage

Override function parameter Get-BcContainerAppRuntimePackage. The default scriptblock for this parameter is:

```
{ Param([Hashtable]$parameters)
    Get-BcContainerAppRuntimePackage @parameters
}
```

## \-RemoveBcContainer

Override function parameter for Remove-BcContainer. The default scriptblock for this parameter is:

```
{ Param([Hashtable]$parameters)
    Remove-BcContainer @parameters
}
```

# That’s it

That is the currently supported parameters in the Run-AlPipeline function.

Take it for a spin and file an issue on [https://github.com/microsoft/navcontainerhelper/issues](https://github.com/microsoft/navcontainerhelper/issues) if you encounter things that doesn’t work as expected.

I will update the HOL with a more detailed description on how to use this end 2 end.

Note that if the container doesn’t start correctly it probably isn’t caused by the run-alpipeline function. Only file an issue if you can create containers using New-BcConatiner – and it doesn’t work with Run-AlPipeline.

If you cannot get New-BcContainer working – when Run-AlPipeline also won’t work.

You can always create an Azure VM using [http://aka.ms/getbc](http://aka.ms/getbc) and use that as agent or development machine.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
