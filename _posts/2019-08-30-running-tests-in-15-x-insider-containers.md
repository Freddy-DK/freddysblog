---
layout: post
title: "Running tests in 15.x insider containers"
date: 2019-08-30 07:19:07
categories: ["AL Development", "Docker", "NavContainerHelper", "PowerShell"]
tags: ["Docker", "NavContainerHelper", "Tests"]
permalink: /2019/08/30/running-tests-in-15-x-insider-containers/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

If you didn’t read this blog post: [https://freddysblog.com/2019/04/13/running-tests-in-containers/](/2019/04/13/running-tests-in-containers/), then please do so before proceeding. This blog post will only describe what’s new when running tests in 15.x containers, which while writing this blog post still are in preview and only available as insider builds through the Ready To Go program. Read more here: [https://freddysblog.com/2019/07/31/preview-of-dynamics-365-business-central-2019-release-wave-2/](/2019/07/31/preview-of-dynamics-365-business-central-2019-release-wave-2/)

# Get the latest…

In order to run tests with 15.x containers you will need the latest NavContainerHelper ([version 0.6.3.2 or later](https://www.powershellgallery.com/packages/navcontainerhelper/0.6.3.2)) and you need Business Central 2019 Wave 2 insider build 15.0.35805.0 or later using

```
docker pull bcinsider.azurecr.io/bcsandbox-master:<country>-<platform>
```

where <country> is the localization and <platform> is ltsc2016 or ltsc2019 based on your version of Windows (see more here: [https://freddysblog.com/2019/07/31/preview-of-dynamics-365-business-central-2019-release-wave-2/](/2019/07/31/preview-of-dynamics-365-business-central-2019-release-wave-2/)).

docker inspect on your image should return the following labels (or newer):

```
"Labels": {
    ...
    "platform": "15.0.35510.0",
    "tag": "0.0.9.92",
    "version": "15.0.35805.0"
},
```

# Include the test toolkit

Exactly like earlier, you need to either add **\-includeTestToolkit** to New-BcContainer in order to include the Test Toolkit – or you need to run the function **Import-TestToolkitToBCContainer** in order to import the test Toolkit.

The Test Toolkit Test Libraries consists of **5 apps**, which are included on the latest Docker images in the C:\\Applications folder:

-   Microsoft\_Any.app
-   Microsoft\_Library Assert.app
-   Microsoft\_System Application Test Library.app
-   Microsoft\_Tests-TestLibraries.app
-   Microsoft\_Test Runner.app

The source for these applications are also included in .zip files.

Microsoft application tests are also available in the **C:\\Applications** folder and unless you specify **\-includeTestLibrariesOnly**, importing the test toolkit will include all tests. This will take some time.

If you run the following script:

```
$auth = "NavUserPassword"
$credential = New-Object pscredential 'admin', (ConvertTo-SecureString -String 'P@ssword1' -AsPlainText -Force)
$containerName = "runtest"
$licenseFile = "<licenseFile>"
$imageName = "bcinsider.azurecr.io/bcsandbox-master:w1-ltsc2019"
New-BcContainer -accept_eula `
                -containerName $containerName `
                -imageName $imageName `
                -auth $auth `
                -credential $credential `
                -updateHosts `
                -alwaysPull `
                -includeTestToolkit -includeTestLibrariesOnly `
                -licenseFile $licenseFile
```

you should see an output ending with something like:

```
...
Ready for connections!
Reading CustomSettings.config from runtest
Creating Desktop Shortcuts for runtest
Publishing C:\Applications\TestFramework\TestLibraries\Any\Microsoft_Any.app
Synchronizing Any on tenant default
Installing Any on tenant default
App successfully published
Publishing C:\Applications\TestFramework\TestLibraries\Assert\Microsoft_Library Assert.app
Synchronizing Library Assert on tenant default
Installing Library Assert on tenant default
App successfully published
Publishing C:\Applications\TestFramework\TestRunner\Microsoft_Test Runner.app
Synchronizing Test Runner on tenant default
Installing Test Runner on tenant default
App successfully published
Publishing C:\Applications\System Application\Test\Microsoft_System Application Test Library.app
Synchronizing System Application Test Library on tenant default
Installing System Application Test Library on tenant default
App successfully published
Publishing C:\Applications\BaseApp\Test\Microsoft_Tests-TestLibraries.app
Synchronizing Tests-TestLibraries on tenant default
Installing Tests-TestLibraries on tenant default
App successfully published
TestToolkit successfully imported
Container runtest successfully created
```

As you see, the 5 apps are published and installed, the same happens if you omit the -includeTestToolkit and instead call the **Import-TestToolkitToBcContainer** function.

After this, you can open the Test Tool shortcut on the desktop and after invoking the Get Test Codeunits action, you can have the single test added to the list.

![altesttool](/assets/images/2019/running-tests-in-15-x-insider-containers/altesttool.png)

# Get and Run the tests

Now you can run the tests from the UI – or you can run them from PowerShell. Try:

```
Get-TestsFromBCContainer -containerName $containerName -credential $credential
```

and you should get:

```
Tests                  Name                  Id 
-----                  ----                  -- 
{WarmupInvoicePosting} Sys. Warmup Scenarios 130411
```

and if you try to run the tests using:

```
Run-TestsInBCContainer -containerName $containerName -credential $credential -detailed
```

you should get:

```
  Codeunit 130411 Sys. Warmup Scenarios Success (0.472 seconds)
    Testfunction WarmupInvoicePosting Success (0.047 seconds)
```

# The Hello World of tests

Lets try to create a new AL test project. In VS Code, use **Ctrl+Shift+P** and select **AL Go!**

Select the latest target platform (4.0 Business Central 2019 release wave 2). Select your own server and change **Server**, **ServerInstance** and **startupObjectId** in **launch.json**:

```
"server": "http://<containername>",
"serverInstance": "BC",
"startupObjectId": 130451,
```

Replace <containerName> with the name of your container.

In app.json, add dependencies on the Test Framework apps:

```
{
  "appId": "dd0be2ea-f733-4d65-bb34-a28f4624fb14",
  "publisher": "Microsoft",
  "name": "Library Assert",
  "version": "15.0.0.0"
},
{
  "appId": "e7320ebb-08b3-4406-b1ec-b4927d3e280b",
  "publisher": "Microsoft",
  "name": "Any",
  "version": "15.0.0.0"
},
{
  "appId": "9856ae4f-d1a7-46ef-89bb-6ef056398228",
  "publisher": "Microsoft",
  "name": "System Application Test Library",
  "version": "15.0.0.0"
},
{
  "appId": "5d86850b-0d76-4eca-bd7b-951ad998e997",
  "publisher": "Microsoft",
   "name": "Tests-TestLibraries",
   "version": "15.0.0.0"
}
```

Remove the Helloworld.al and create a new Codeunit like this:

```
codeunit 50100 "My tests"
{
    Subtype = Test;
    TestPermissions = Disabled;

    var
        Assert: Codeunit "Library Assert";

    [Test]
    [Scope('OnPrem')]
    procedure TestA()
    begin
        Assert.AreEqual(3, 3, '3 and 3 are equal');
    end;

    [Test]
    [Scope('OnPrem')]
    procedure TestB()
    begin
        Assert.AreEqual(3, 4, '3 and 4 are not equal');
    end;
}
```

_**Note**, that the Library Assert codeunit is coming from the Library Assert application and that the TestLibraries contains the “old” Assert codeunit._

**Note also** that TestB is supposed to fail.

Publish the app without debugging and it should start the test tool page.

![altesttool2](/assets/images/2019/running-tests-in-15-x-insider-containers/altesttool2.png)

Invoke the **Get Test Codeunits** action to include the new codeunit. Now you can run the tests directly in the UI (… -> Run Tests) or you can use PowerShell again:

```
Run-TestsInBCContainer -containerName $containerName -credential $credential -detailed
```

and the result should be something like this:

```
  Codeunit 130411 Sys. Warmup Scenarios Success (0.331 seconds)
    Testfunction WarmupInvoicePosting Success (0 seconds)
  Codeunit 50100 My tests Failure (0.331 seconds)
    Testfunction TestA Success (0.017 seconds)
    Testfunction TestB Failure (0.017 seconds)
      Error:
        Assert.AreEqual failed. Expected:<3> (Integer). Actual:<4> (Integer). 3 and 4 are not equal.
      Call Stack:
        "Library Assert"(CodeUnit 130002).AreEqual - Library Assert by Microsoft
        "My tests"(CodeUnit 50100).TestB line 2 - MyTest by Default publisher
        "Test Runner - Mgt"(CodeUnit 130454).RunTests - Test Runner by Microsoft
        "Test Runner - Isol. Codeunit"(CodeUnit 130450).OnRun - Test Runner by Microsoft
        "Test Suite Mgt."(CodeUnit 130456).RunTests - Test Runner by Microsoft
        "Test Suite Mgt."(CodeUnit 130456).RunSelectedTests - Test Runner by Microsoft
        "Command Line Test Tool"(Page 130455).OnAction - Test Runner by Microsoft
```

# Include the Microsoft Tests

If you leave out the **\-includeTestLibrariesOnly**, you will also see that all apps called **Microsoft\_Tests-<something>.app** in the C:\\Applications folder will be published and installed as well. This does take some time as there are quite a few test apps.

You should see your container creation end with something like this:

![adding tests](/assets/images/2019/running-tests-in-15-x-insider-containers/adding-tests.png)

You will still have to add the tests you want to run programatically or manually in the test tool.

**Note:** While writing this blog post (hardcoded in containerhelper 0.6.3.2) the Marketing and the SINGLESERVER test apps are not published as these contains some invalid dotnet references.

# Creating your own test suite

In the [original running tests blog post](/2019/04/13/running-tests-in-containers/), I described how you could populate the Test Suite from code in the install trigger of your app.

If you want to do this in 15.x, you will need to add a dependency to the **Test Runner** app and populate the tables in that. If you populate the C/AL Test Tool, you won’t be able to find and run them in the AL Test Tool.

# C/AL Test Tool vs. AL Test Tool

Note that the “old” C/AL Test Tool objects are still in the base application and if you specify **\-testpage 130409** to **Get-TestsFromBcContainer** and **Run-TestsInBcContainer**, the functions will actually use the “old” test runner. This might be useful if you populate the Test Tool tables in code.

The C/AL test tool will however be removed sometime in the future and we recommend that you start using the AL Test Tool instead.

The AL Test Tool is in the app called **Microsoft\_Test Runner.app** and the source is available in **C:\\Applications** in the container.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
