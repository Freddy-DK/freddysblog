---
layout: post
title: "Running Tests in Containers"
date: 2019-04-13 10:06:30
categories: ["AL Development", "Docker", "NavContainerHelper"]
tags: ["Docker", "NavContainerHelper", "Tests"]
permalink: /2019/04/13/running-tests-in-containers/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

Since the very start of NAV and Business Central on Containers, it has been possible to run tests through the UI (Windows Client and Web Client), just as you have been able to do when you install directly on a machine. A lot of people have however been looking for a way to run tests automated – and… – since NavContainerHelper 0.5.0.3 (February 25th) a function called **Run-TestsInNavContainer** has been available. A few changes has been implemented since then and I am now ready to describe how to use the function.

# Include the Test Toolkit when creating the container

When creating the Container, you can specify **\-IncludeTestToolkit** in order to include Test Toolkit and all standard tests:

```
$imageName = "mcr.microsoft.com/businesscentral/onprem:w1-ltsc2019"
$credential = New-Object pscredential 'admin', (ConvertTo-SecureString -String 'P@ssword1' -AsPlainText -Force)
$containerName = "mytest"
New-NavContainer -accept_eula `
                 -imageName $imageName `
                 -containerName $containerName `
                 -auth "NavUserPassword" `
                 -Credential $credential `
                 -updateHosts `
                 -includeTestToolkit `
                 -licenseFile "C:\temp\license.flf"
```

This resembles importing the .fob files in the TestToolKit folder on the DVD or in the Docker image.

If you add **\-includeTestLibrariesOnly**, then you will only get the Test Libraries and not the test codeunits.

You should now find a shortcut on the desktop called myTest Test Tool and opening this will reveal the Test Toolkit:![emptytesttool](/assets/images/2019/running-tests-in-containers/emptytesttool.png)

In this, you setup your test by getting the test codeunits and setup the test suite you want to run:![testtool](/assets/images/2019/running-tests-in-containers/testtool.png)

Now, you can press Run and run the test.

But hey… – this is all old news, we have been able to do this for a long time… – why is this blog post describing this???

# Get-TestsFromNavContainer

The simple answer is, that the test suite you setup in the Test Toolkit is reused by the CmdLets. In fact, try to run Get-TestsFromNavContainer:![gettests](/assets/images/2019/running-tests-in-containers/gettests.png)

and you can see the same tests as you have in the Test Tool.

You can specify the **testSuite** as a parameter to **Get-TestsFromNavContainer** and with that filter the tests to only that **testSuite**. Default value for **testSuite** is **DEFAULT**, meaning that if your tests are in a different **testSuite**, you will need to specify that.

You can also specify **testCodeunit**, which will show only the tests in that codeunit. **Note** that you still need to specify the **testSuite**.

# Run-TestsInNavContainer

With that, you can run the same tests by using Run-TestsInNavContainer:![runtests](/assets/images/2019/running-tests-in-containers/runtests.png)

Note that if you will only have colors if you are running PowerShell as administrator.

And **Yes**, **Run-TestsInNavContainer** will run tests using Page testability (unlike invoking codeunits for testing).

**Run-TestsInNavContainer** also has parameters for filtering which tests to run:

-   **testSuite** – you will always filter the tests to a specific test Suite. By default, you filter to the DEFAULT test suite, which means that the remaining parameters will only filter tests in that test suite.
-   **testGroup** – name pattern of test group to run – or \* to run all test groups in the test suite.
-   **testCodeunit** – name or id pattern of test codeunits to run – or \* to run all all test codeunits in the test group / suite.
-   **testFunction** – name pattern of the test functions to run – or \* to run all test functions in the test codeunit / group / suite.

Example:![runtests1](/assets/images/2019/running-tests-in-containers/runtests1.png)

Another example:![runtests2](/assets/images/2019/running-tests-in-containers/runtests2.png)

You get the picture…

# XUnit

Run-TestsInNavContainer can also create an XUnit compatible output from test execution by specifying XUnitResultFileName and the path to a file, which is shared with the container.

Example XUnit result file:

```
<?xml version="1.0" encoding="UTF-8"?>
<assemblies>
  <assembly name="Sales Document Posting Errors" test-framework="PS Test Runner" run-date="2019-04-13" run-time="10:56:31" total="1" passed="1" failed="0" time="0.563">
    <collection name="Sales Document Posting Errors" total="1" passed="1" failed="0" skipped="0" time="0.563">
      <test name="Sales Document Posting Errors:T001_PostingDateIsInNotAllowedPeriodInGLSetup" method="T001_PostingDateIsInNotAllowedPeriodInGLSetup" time="0.563" result="Pass" />
    </collection>
  </assembly>
  <assembly name="Purch. Document Posting Errors" test-framework="PS Test Runner" run-date="2019-04-13" run-time="10:56:32" total="1" passed="1" failed="0" time="0.5">
    <collection name="Purch. Document Posting Errors" total="1" passed="1" failed="0" skipped="0" time="0.5">
      <test name="Purch. Document Posting Errors:T001_PostingDateIsInNotAllowedPeriodInGLSetup" method="T001_PostingDateIsInNotAllowedPeriodInGLSetup" time="0.5" result="Pass" />
    </collection>
  </assembly>
</assemblies>
```

XUnit is one of the formats supported by Azure DevOps.

# Azure DevOps

As all of the above wasn’t enough, you can also specify AzureDevOps as a parameter. The AzureDevOps parameter signals how test failures will appear in Azure DevOps pipelines. You can specify **Warn**, **Error** or **No**. Default is No as in test failures will not be displayed. Warn or Error will show test failures accordingly.

# But wait…, how do I create my test suite?

In most professional programming environments, it is enough to attribute your codeunits and functions as tests in order for them to be added to a test subsystem. The fact that Business Central has a table for tests and a UI for running tests inside the client is because we historically did it like this.

In the future, we should assume that this UI will disappear and I am sure that we will get functionality in VS Code to see which tests are available in my app / in the base app for me to run (looking forward to this).

So, with that, I did not want to create functions to populate the database with tests. This would cause partners to place the responsibility of building the test suite in a wrong place. Building the test suite belongs in the test app. In the future by specifying attributes, for now we have to do slightly more.

My recommendation is to include an install codeunit in your test app, which will populate the test toolkit based on attributes. This means that when Microsoft ships a replacement for the test toolkit, you will only need to remove this codeunit. You can find an example of how to do this here: [https://dev.azure.com/businesscentralapps/\_git/HelloWorld?path=%2Ftest](https://dev.azure.com/businesscentralapps/_git/HelloWorld?path=%2Ftest)

![installcu](/assets/images/2019/running-tests-in-containers/installcu.png)

The TestSuite codeunit is also in the same folder. I will probably do some refactoring of this in the near future, but basically this functionality is just temporary.

# The future?

At this time, we do not know how the future is going to look, what functionality we will get from the new Test framework and we also don’t know when we will get this functionality.

It is however the goal to refactor the functionality of **Get-TestsFromNavContainer** and **Run-TestsInNavContainer** to work with the new test framework, when it is delivered in order for CI/CD scripts to continue working with next gen test toolkit. Note, that we might not be able to support all functionality as I do not know whether we will have test suites and test groups in the future.

# Under the hood

This section is for the nerds, who want to know how this works.

You might have noticed, that during the first usage of any of the two functions, it said _Importing Objects from C:\\ProgramData\\NavContainerHelper\\Extensions\\mytest\\PsTestTool\\PSTestToolPage.fob_

Looking in that folder, you will find 3 files:

-   ClientContext.ps1
-   PsTestFuncions.ps1
-   PSTestToolPage.fob

The Test Tool Page is a new Test Toolkit page called 130409 and if you try to launch that, you will see a simplified Test Toolkit page:![130409](/assets/images/2019/running-tests-in-containers/130409.png)

Only one action available: **Run Selected** and no strings for success and failure – instead just simple 0, 1 or 2. Also if ýou scroll all the way to the right, you will see that the list includes the call stack for errors as well.

**ClientContext.ps1** is a PowerShell class which basically works as a Client towards NAV / Business Central. With ClientContext you can open a connection to the Client endpoint and act like you are a client. This is the same functionality as used in the performance test toolkit and you can do stuff like:![clientcontext](/assets/images/2019/running-tests-in-containers/clientcontext.png)

There is no documentation for ClientContext and it will probably only ever be used for running tests while we are waiting for future functionality.

**PsTestFunctions.ps1** is a script, which contains functions for Run-Tests and Get-Tests, which both creates a ClientContext (a connection to the Client) and when open the page 130409 and act like a user running the tests needed.

Any usage of this knowledge is at your own risk… the functionality might be obsolete in the future (I think I have mentioned that enough times by now).

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
