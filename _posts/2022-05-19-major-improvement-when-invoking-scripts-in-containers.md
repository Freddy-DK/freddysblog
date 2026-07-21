---
layout: post
title: "Major improvement when invoking scripts in Containers…"
date: 2022-05-19 18:36:57
categories: ["BcContainerHelper", "Docker"]
tags: ["Azure DevOps", "Freeze", "GitHub Actions", "Hang", "Pipeline", "Workflow"]
permalink: /2022/05/19/major-improvement-when-invoking-scripts-in-containers/
---

I recently learned that some partners have had had issues when running build pipelines on Azure DevOps with multiple DevOps agents on the same host using Containers with process isolation. 1-2 years ago, we did a number of fixes to support multiple agents on the same host, and I thought that had fixed things, but apparently this was not true.

Hopefully the fixes, which are going into BcContainerHelper today will fix this once and for good.

## Not a BcContainerHelper problem

My immediate thinking was that this is not a BcContainerHelper problem, but that doesn’t mean that it cannot be fixed in BcContainerHelper. There are several workarounds in BcContainerHelper to cope with wrongdoings of other software packages, even just-in-time patching of Business Central some times.

I learned that people were seeing their DevOps tasks complete and then freeze. Everything was A-OK but the pipeline would just hang and eventually timeout and fail. I had even seen the same once on GitHub Actions, but some partners were complaining that they had this very frequently.

There was a bug files on Azure Pipelines [here](https://github.com/microsoft/azure-pipelines-tasks/issues/15819), and then another bug on BcContainerHelper [here](https://github.com/microsoft/navcontainerhelper/issues/2469). Julian from GWS was so kind to setup a repository on Azure DevOps, running 20 simultaneous builds and almost certain, that at least one would fail in every run.

I got access to the repo and started debugging. My first hunch was, that some process, thread or job was running on the agent and the DevOps agent would wait for this to complete before finishing the task.

I was right!

## The PsSession

It turned out to be the PsSession, which is a PowerShell session, which is created by Invoke-ScriptInBcContainer, cached and re-used again and again. First attempt was to just remove the PsSession, but this also caused the task to freeze.

The problem seemed to be that removing the session (using the PowerShell command Remove-PsSession) would hang – and exiting the PowerShell task would implicitly remove the session, meaning that the task would hang – some times.

## 2 solutions possible

There could only be two solutions:

1.  Find a better way to remove the session
2.  Avoid using the session

and I kind of found a solution to both.

## Find a better way to remove the session

I did manage to find a way, where I could remove the session without freezing PowerShell. Instead of using Remove-PsSession, I could locate the process ID of the session (using process isolation, this process ID is the process ID on the host) and simply stop the process.

This works – but unfortunately, I don’t have any way of intercepting the implicit session removal when the PowerShell task is ending, so it would still freeze if I didn’t remove the session manually.

I don’t think the right solution would be to ask everybody to remove all sessions in all tasks in all pipelines – we need a better solution.

## Avoid using the PsSession

This was a fairly easy fix. Add

```
"UsePsSession":  false
```

To c:\\ProgramData\\BcContainerHelper\\BcContainerHelper.config.json and you should be good. This causes Invoke-ScriptInBcContainer to use docker exec instead of a PsSession.

It actually packages your scriptblock in a script .ps1 file, encodes the parameters and uses docker exec to run the script. Return values are written to another file and then returned from the function, but… – to be honest I never did much optimization of running Invoke-ScriptInBcContainer without a PsSession and I was almost certain that there would be bugs.

I was pleased to see that most bugs were in error handling. If something went wrong in the scriptblock, then the error wasn’t returned correctly. Beside this, the old version didn’t support if hostHelperFolder is different from ContainerHelperFolder. This could be a problem for some.

In the next pre-release of BcContainerHelper, these issues are fixed, and you should now get the correct error surfaced from the function. This new functionality could potentially break behavior, so I have added a setting, which you can set to false to get the “old and faulty” behavior back (if this worked for you):

```
$bcContainerHelperConfig.addTryCatchToScriptBlock = $false
```

If you discover an example where the new behavior doesn’t work, please don’t just set this setting to false and live with it. Please open an issue on [https://github.com/microsoft/navcontainerhelper/issues](https://github.com/microsoft/navcontainerhelper/issues) and explain which scriptblock the new mechanism doesn’t work with.

## Out of memory

Additionally, one type of error, which has been hard to detect earlier is Out Of Memory. When I (inside an Invoke-ScriptInBcContainer) use the PowerShell CmdLet to publish an app to the service tier and this causes the servicetier to crash due to lack of memory, I would typically get errors like: “The socket connection was aborted.” as this is what the PowerShell session sees.

In this new version (both using sessions and docker exec), I will (if the scriptblock throws an exception) inspect the eventlog for entries pushed by Business Central or MS SQL – and if any of those have thrown an OutOfMemory exception, I will throw an OutOfMemory error (the root cause) instead of the socket connection error (the symptom).

![](/assets/images/2022/major-improvement-when-invoking-scripts-in-containers/image-46.png)

This should help the troubleshooting in some cases.

## The pre-release

So, if you have had this problem in your pipelines, please install the next pre-release (preview605 or newer) of BcContainerHelper on your agents and add this setting to bcContainerHelper.settings.json on your agents:

```
"UsePsSession":  false
```

Then this should fix the issue. Let me know whether it works. I was considering adding the setting as default for GitHub Actions and Azure Pipelines, but it does have a performance impact, so I decided not to.

BTW. Even though I only saw this once on AL-Go for GitHub, I will (in the next version) use the mechanism to stop the PsSession process by default, meaning that AL-Go for GitHub should not have hanging sessions.

Thanks

**_Freddy Kristiansen_**  
Technical Evangelist
