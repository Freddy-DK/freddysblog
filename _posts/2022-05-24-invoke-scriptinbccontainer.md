---
layout: post
title: "Invoke-ScriptInBcContainer"
date: 2022-05-24 12:34:06
categories: ["BcContainerHelper"]
tags: ["Docker", "Invoke-ScriptInBcContainer", "PowerShell Session", "Session"]
permalink: /2022/05/24/invoke-scriptinbccontainer/
---

As mentioned in [this blog post](https://freddysblog.com/2022/05/19/major-improvement-when-invoking-scripts-in-containers/), the Invoke-ScriptInBcContainer has undergone some serious changes in BcContainerHelper 3.0.9, which just shipped.

This blog post will describe some details about how this function works.

The function takes a containerName, a scriptblock and an argument list as parameters and will execute this scriptblock inside the container.

The function does however have two ways of performing the exact same function. If the user is using PowerShell as administrator AND the setting called usePsSession is not specifically set to false, then the function will use PowerShell remoting to execute the PowerShell code snippet inside the container.

PowerShell remoting is only available when running with elevated permissions (as administrator). If you are not, or if you have set usePsSession to false – then the function will use docker exec to execute the PowerShell code snippet inside the container.

## Using PowerShell remoting

Invoking a PowerShell snippet using PowerShell remoting is done by establishing a PowerShell Session to the container (caching that session for future use) and invoking the scriptblock using that session.

Establishing a PowerShell session takes approx. 0.9 second (on my machine). Any subsequent usages of that session take approx. 4 milliseconds. The caching of the session is very important for performance on chatty scripts. To illustrate this, you can run this script:

```
$bcContainerHelperConfig.usePsSession = $true
$containerName = "bcserver"
Remove-BcContainerSession -containerName $containerName

Measure-Command {
    $what = Invoke-scriptInBcContainer $containerName -scriptBlock { Param($year)
        "Summer of $year"
    } -argumentList 69
} | ForEach-Object { Write-Host "Time spent: $($_.TotalMilliseconds)" }
Write-Host -ForegroundColor Yellow $what

$cnt = 100
Measure-Command {
    1..$cnt | % {
        $what = Invoke-scriptInBcContainer $containerName -scriptBlock { Param($year)
            "Summer of $year"
        } -argumentList 69
    }
} | ForEach-Object { Write-Host "Time spent: $($_.TotalMilliseconds/$cnt)" }
Write-Host -ForegroundColor Yellow $what
```

The output of this script on my machine is:

![](/assets/images/2022/invoke-scriptinbccontainer/image-48.png)

**NOTE** that the subsequent runs **ONLY** is if the Invoke-ScriptInBcContainer happens in the **SAME** PowerShell session on the **host**.

## Using Docker Exec

If PowerShell remoting is not possible (or you elect not to use it), BcContainerHelper will wrap your scriptblock in some pre- and post- code to best simulate that you are running in a session inside the container and write this PowerShell script to the disk in a folder, which is shared with the container (under the hostHelperFolder on the host).

Now it will serialize the parameters, converting all secureStrings to encrypted strings (else they would not be accessible inside the container) and use docker exec to run the PowerShell script file.

The return value is written to another file, picked up by the host after docker exec.

Sounds cumbersome, but wrapped inside Invoke-ScriptInBcContainer is becomes easy to use. Running the same script as above (just with usePsSession set to false), reveals:

![](/assets/images/2022/invoke-scriptinbccontainer/image-49.png)

In this case, it doesn’t really matter whether the subsequent calls are in seperate PowerShell sessions.

## Two ways of running your pipelines/workflows

When running on Azure DevOps and using one DevOps task to create the container, another one to compile and another one to publish – each of these tasks are seperate PowerShell sessions and the difference between running using PowerShell remoting or Docker exec might not be as outspoken.

When running the Run-AlPipeline function (the do-it-all function used in later versions of the CI/CD HOL and also AL-Go for GitHub), everything happens in one PowerShell session, and it can really take advantage of using PowerShell sessions. Here we might see a 25% performance cut by using Docker exec.

AL-Go for GitHub even adds a RemoveBcContainer override when calling Run-AlPipeline in order to remove the session by killing the underlying process instead of removing the session and potentially hanging.

If you are using individual tasks, you have two options to avoid the famous freezing DevOps tasks:

1.  Set UsePsSession to false
2.  Wrap your code in a try / finally, where you use Remove-BcContainerSession with -killPsSessionProcess in the finally section of your task.

## Error handling in Invoke-ScriptInBcContainer

In the latest version of BcContainerHelper, error handling using Invoke-ScriptInBcContainer has improved significantly.

Try running this script with UsePsSession true or false:

```
$appInfo = Invoke-scriptInBcContainer $containerName -scriptBlock {
    Get-NavAppInfo -ServerInstance X
}
```

As there is no serverInstance X – it will fail, but it does add some additional information:

![](/assets/images/2022/invoke-scriptinbccontainer/image-50.png)

this was certainly not the case before 3.0.9:-)

As a bonus, it will actually dump all event log entries that have been added while the script was running. It will also investigate whether any of the event log entries is an out-of-memory exception and if that is the case, it will re-throw an out-of-memory exception.

You will also see whether your script caused the ServiceTier to crash and end up as not running.

## Returning objects

Now try to run this script:

```
$appInfo = Invoke-scriptInBcContainer -containerName $containerName -scriptBlock {
    Get-NavAppInfo -ServerInstance $serverInstance
}
Write-Host -ForegroundColor Yellow "There are $($appInfo.Count) apps"
$appInfo[0] | Out-Host
```

You will see a slightly different output. When running using a PsSession, the serialized objects have a few extra fields added:

![](/assets/images/2022/invoke-scriptinbccontainer/image-51.png)

The PSComputerName and RunspaceId are added to all objects serialized from Invoke-ScriptInBcontainer to the host. You can just ignore these; I do not use them for anything and I haven’t done any effort to try and remove them.

## You cannot return a SecureString

Only known limitation to using Invoke-ScriptInBcContainer with Docker Exec is, that you cannot return a SecureString from the container. Oher than that, there should be parity between the functionality of using PsSession and Docker Exec.

If not, please file an issue on [https://github.com/microsoft/navcontainerhelper/issues](https://github.com/microsoft/navcontainerhelper/issues).

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
