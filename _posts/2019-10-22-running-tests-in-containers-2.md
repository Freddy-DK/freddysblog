---
layout: post
title: "Running Tests In Containers"
date: 2019-10-22 12:15:35
categories: ["AL Development", "CI/CD", "NavContainerHelper"]
tags: ["CI", "Docker", "Tests"]
permalink: /2019/10/22/running-tests-in-containers-2/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

Running automated tests is an essential part of any CI/CD strategy. For Business Central, we have been able to use containers and the function Run-TestsInNavContainer in the NavContainerHelper PowerShell module as described in [this blog post](/2019/04/13/running-tests-in-containers) (make sure you have read this before continuing) for 6-12 months and a lot of partners are already taking advantage of this. This blog post shows you how to get the max. out of the functions.

# ![this is a test](/assets/images/2019/running-tests-in-containers-2/this-is-a-test.png)

# On the surface, nothing has changed

The functions to get tests and run tests should work in the same way as described in the original blog post and you can now also used the BCContainer alias’s instead of NavContainer – same functionality.

The functions also still uses the **ClientContext.ps1** and **PSTestFunctions.ps1** to connect to the Client Service next to the Web Client in order to run tests and for Business Central containers prior to 15.x, it also still imports the **PSTestToolPage.fob**, which contains the C/AL Test Runner.

From 15.x, the TestToolPage is now part of the TestRunner app, which is installed in 15.x containers when importing the test toolkit.

There are however a number of changes and options you can use when running test. Let me go through the options by showing a few samples and explain what they do.

In order to run these scenarios you need **NavContainerHelper 0.6.4.14** or later.

# Run an entire Test Suite in one go

If you have all your tests in the DEFAULT test suite, you want to run them all and collect the results, you can do this like:

```
$containerName = "test"
$xunitResultsFile = "c:\programdata\navcontainerhelper\results.xml"
Run-TestsInBCContainer `
    -containerName $containerName `
    -credential $credential `
    -testSuite 'DEFAULT' `
    -detailed `
    -XUnitResultFileName $xunitResultsFile
```

The **\-detailed** flag only affects what is written as information on the host and will print information about every test whether they succeeded or failed. Without -detailed, only failing tests will be explicitly printed.

The **\-xunitResultFileName** specifies the filename in which you want to write the output. The output file is written at the end of the test execution and follows [this format](https://xunit.net/docs/format-xml-v2).

# Rerun failing tests

If you want to re-run failing tests, you can do so by reading the xunit results file and re-running the codeunits, which included failing tests:

```
$containerName = "test"
$xunitResultsFile = "c:\programdata\navcontainerhelper\results.xml"
$xunitResults = [xml](Get-Content -Path $xunitResultsFile)
$xunitResults.assemblies.assembly | Where-Object { $_.Failed -ne "0" } | ForEach-Object {
    Write-Host "Rerun $($_.Name)"
    $codeunitId = $_.Name.Split(' ')[0]
    Run-TestsInBCContainer `
        -containerName $containerName `
        -credential $credential `
        -testSuite 'DEFAULT' `
        ​-detailed `
        -xUnitResultFileName $xunitResultsFile `
        -ReRun `
        -testCodeunit $codeunitId
}
```

Specifying **\-rerun** means that the xunitResultsfile will not be deleted and instead the result of this test run will replace the result already written in the results file.

Specifying **\-testCodeunit** means that this test run will only execute codeunits, which matches this value (name or id). **It is significantly faster to specify an id** than a name as that internally turns into a filter. Specifying a name will look through all test codeunits and tests to match the name.

# Running tests one codeunit at a time

If you want to run your tests one codeunit at a time instead of having one long running Run-Tests function, you can use Get-Tests to get the test codeunits and execute the codeunits one by one:

```
$containerName = "test"
$xunitResultsFile = "c:\programdata\navcontainerhelper\results.xml"
$tests = Get-TestsFromBCContainer `
    -containerName $containerName `
    -credential $credential `
    -testSuite 'DEFAULT' `
    -ignoreGroups
$first = $true
$tests | ForEach-Object {
    Run-TestsInBCContainer `
        -containerName $containerName `
        -credential $credential `
        -XUnitResultFileName $xunitResultsFile `
        -AppendToXUnitResultFile:(!$first) `
        -testCodeunit $_.Id
    $first = $false
}
```

Calling **Get-TestsInBCContainer** will enumerate all test codeunits and return them in an array of HashTables.

The **\-ignoreGroups** flag is to ensure that the returned hashtable is the same in all versions, as the **AL Test Runner doesn’t support groups** and as such always returns a result set without groups.

The **\-appendToXUnitResultFile** on Run-Tests will signal to the function that it should append any runs to the existing results file **instead of removing the results file** when starting the test run and creating a new results file when the test run is done.

# Rerunning failed tests without reading the results file

When running one codeunit at a time, you can re-run test codeunits like earlier by reading the results file and re-running failed codeunits. If you don’t want to do this, you can also do this:

```
$containerName = "test"
$xunitResultsFile = "c:\programdata\navcontainerhelper\results.xml"
$tests = Get-TestsFromBCContainer `
    -containerName $containerName `
    -credential $credential `
    -testSuite 'DEFAULT' `
    -ignoreGroups
$rerunTests = @()
$failedTests = @()
$first = $true
$tests | ForEach-Object {
    if (-not (Run-TestsInBcContainer `
        -containerName $containerName `
        -credential $credential `
        -XUnitResultFileName $TempTestResultFile `
        -AppendToXUnitResultFile:(!$first) `
        -testCodeunit $_.Id `
        -returnTrueIfAllPassed)) { $rerunTests += $_ }
    $first = $false
}
if ($rerunTests.Count -gt) {
    Restart-BCContainer -containerName $containerName
    $rerunTests | ForEach-Object {
        if (-not (Run-TestsInBcContainer `
            -containerName $containerName `
            -credential $credential `
            -XUnitResultFileName $TempTestResultFile `
            -AppendToXUnitResultFile:(!$first) `
            -testCodeunit $_.Id `
            -returnTrueIfAllPassed { $failedTests += $_ }
        $first = $false
    }
}
```

Normally Run-Tests doesn’t return any value, but if you specify **\-returnTrueIfAllPassed**, then the function will return **$true if all tests in the codeunit passed** or $false if just one test was failing. We can use this to pickup the codeunits we want to re-run.

# Running tests in an Azure DevOps pipeline

When running tests in a pipeline, we will typically publish the XUnit test results as the test results for the pipeline after the full test run is done. This can be used to troubleshoot and analyze. You can also make Azure DevOps see test failures as warnings or errors, which in the pipeline will look like this:

![pipeline](/assets/images/2019/running-tests-in-containers-2/pipeline.png)

```
$containerName = "test"
$xunitResultsFile = "c:\programdata\navcontainerhelper\results.xml"
Run-TestsInBCContainer `
    -containerName $containerName `
    -credential $credential `
    -testSuite 'DEFAULT' `
    -detailed `
    -XUnitResultFileName $xunitResultsFile `
    -AzureDevOps "warning"
```

You can specify **error**, **warning** or **no** to **\-azureDevOps** which will cause the pipeline to fail, warn or ignore test failures at this point.

Another way of failing a build pipeline is to set failTaskOnFailedTests to true in the CI.yml file on the PublishTestResults task.

# Running all tests in an extension (15+ only)

15+ doesn’t mean that this is only for people age 15 or above – it of course means that this is only supported in Business Central 15 containers or above.

In NAV and Business Central 13.x and 14.x we recommend that you have an install codeunit in the test app, which populates the C/AL Test Runner with the tests in your test app.

If you are running Business Central 15 and beyond we do not recommend that anymore!

Instead you can split your tests in separate test apps and run them individually by specifying the app id to Run-Tests:

```
$containerName = "test"
$xunitResultsFile = "c:\programdata\navcontainerhelper\Tests-ERM-results.xml"
$app = Get-NavContainerAppInfo $containerName | Where-Object { $_.Name -eq "Tests-ERM" }
Run-TestsInBCContainer `
    -containerName $containerName `
    -credential $credential `
    -detailed `
    -XUnitResultFileName $xunitResultsFile `
    -extensionId $app.AppId
```

The **\-extensionId** parameter should be an appId from the extension in which you want to run all tests. This way you don’t have to populate the AL Test Runner tables and it is likely that this is the way we will be running tests even when the AL Test Runner is retired. Note, that the [HelloWorld template for CI/CD](https://dev.azure.com/businesscentralapps/HelloWorld/_git/HelloWorld?path=%2Ftest&version=GBmaster) has adopted this and the test app does no longer have an install codeunit.

Remember to specify **\-AppendToXUnitResultFile** if you want to collect the results from multiple apps in one XUnit Results file – or use a results file per app (like above).

If your scope of test runs is an app and you use seperate XUnit Result files, there is no need for specifying -rerun, as you always will run the entire app.

# Running all tests in an extension one codeunit at a time (15+ only)

Like earlier, where you could run all tests in a test suite one codeunit at a time, you can do the same with test codeunits in an extension:

```
$containerName = "test"
$xunitResultsFile = "c:\programdata\navcontainerhelper\Tests-ERM-results.xml"
$app = Get-NavContainerAppInfo $containerName | Where-Object { $_.Name -eq "Tests-ERM" }
$tests = Get-TestsFromBCContainer `
    -containerName $containerName `
    -credential $credential `
    -extensionId $app.AppId `
    -ignoreGroups
$first = $true
$tests | ForEach-Object {
    Run-TestsInBCContainer `
        -containerName $containerName `
        -credential $credential `
        -detailed `
        -XUnitResultFileName $xunitResultsFile `
        -AppendToXUnitResultFile:(!$first) `
        -testCodeunit $_.Id
    $first = $false
}
```

Specifying **\-extensionId** on Get-Tests will return the tests in that extension and then you can enumerate the test codeunits and run them one by one.

Remember – Business Central 15.x containers and beyond.

# Skipping disabled tests (15+ only)

Another functionality, which is new in Business Central 15.x containers is that you can specify an array of hashtables, which describes tests that should be disabled.

```
$containerName = "test"
$xunitResultsFile = "c:\programdata\navcontainerhelper\Tests-ERM-results.xml"
$app = Get-NavContainerAppInfo $containerName | Where-Object { $_.Name -eq "Tests-ERM" }
Run-TestsInBCContainer `
    -containerName $containerName `
    -credential $credential `
    -detailed `
    -XUnitResultFileName $xunitResultsFile `
    -extensionId $app.AppId `
    -disabledTests @(
        @{"CodeunitName" = "Sales Document Posting Errors"; "Method" = "*" },
        @{"CodeunitName" = "Purch. Document Posting Errors"; "Method" = "T002_PostingDateIsInNotAllowedPeriodInUserSetup" }
    )
```

The **\-disabledTests** can be an array of HashTables, which contains a **codeunitName** property and a **Method** property and in the output, you will see that disabled tests are skipped:

![skippedtests](/assets/images/2019/running-tests-in-containers-2/skippedtests.png)

On the DVD (and in Docker) in the \\Applications\\TestFramework\\TestRunner, there is a file called DisabledTests.json. This is a list of Microsoft Tests, which are currently disabled. When you have tests, which you temporarily want to disable, you should create a file like this yourself. To Use this file as input to Run-Tests, you need to use:

```
$disabledTests = Get-Content $disabledTestsFile | ConvertFrom-Json
```

and then use -disabledTests $disabledTests.

# What if Run-Test is failing

Admitted, there have been some instabilities when running tests through the Run-Tests function (maybe there still are?), but we have done our utmost to make it more stable. These instabilities has mostly been due to various timeout settings, which wasn’t setup correctly and this should be solved now, but you might still run into issues.

If the **Container runs out of memory**, if the **Service Tier crashes,** if there is a **Communication Failure** or if the **Session in which you are running tests crashes**, you will get an error and test execution will be cancelled.

The error is likely to be **ClientSession state is InError or Uninitialized**. If you experience problems, where Run-Tests crashes, here is what you can try:

**#1** – add **\-restartContainerAndRetry** to Run-Tests. If an exception occurs and test execution cannot continue, then Run-Tests will restart the container and retry the last codeunit automatically. This could take care of situations like memory issues. It is OK to use -restartContainerAndRetry in your CI/CD pipeline in order to avoid false failures.

**#2** – monitor memory usage in the container (using **docker stats**) and recreate the container with more memory (or process isolation) if exhausted.

**#3** – get the **Eventlog** from the container using **Get-BCContainerEventLog** and see whether this is a Service Tier or Session crash and whether the reason behind this is solvable for you.

**#4** – try to add **\-debugMode** to Run-Tests. This will increase the output with some additional debug information that you can include when creating an issue. You should only run with -debugMode in your CI/CD pipeline while troubleshooting issues.

**#5** – try to add **\-usePublicWebBaseUrl** and run the tests. This causes the PowerShell script to connect to the **public web base url** instead of keeping the connection inside the container (localhost). You can use the **\-usePublicWebBaseUrl** in CI/CD pipelines if you need.

**#6** – try to run the tests with **\-connectFromHost**. This will run the PowerShell script, which connects to the Client Service on the host instead of inside the container. This in itself shouldn’t change anything (if it does, please create an issue and let me know), but it allows you to start a web debugger like [Fiddler](https://www.telerik.com/fiddler) and trace the connection and capture the last communication between Run-Tests and the Client Service. The communication can look like:

![fiddler](/assets/images/2019/running-tests-in-containers-2/fiddler.png)

In this image, we can see that InvokeAction was called on ControlId 11 and we can inspect response, names, events etc.

I do **not recommend using -connectFromHost in CI/CD** pipelines as this might cause some files to be locked and subsequently, some containers cannot be deleted.

**#7** – **Create an issue on GitHub [here](https://github.com/microsoft/navcontainerhelper/issues).** When creating an issue, include the **script used to create the container**, include the **full output of the container creation**, the **command used to run the tests**, the **eventlog** and/or the **fiddler trace** – as much information as possible to help resolve the issue and make everybody’s life better.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
