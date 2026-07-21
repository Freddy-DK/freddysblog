---
layout: post
title: "Bugfix: Something went wrong"
date: 2019-04-25 13:22:37
categories: ["Docker", "NavContainerHelper"]
tags: ["Bug", "Generic", "override"]
permalink: /2019/04/25/bugfix-something-went-wrong/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

I have had a few partners reporting a strange bug when using the Web Client in Business Central spring release. Running tests, applying configuration packages or running other long running tasks in the Web Client sometimes would result in an error stating: **Something went wrong**.

![somethingwentwrong](/assets/images/2019/bugfix-something-went-wrong/somethingwentwrong.png)

Looking in the event log you will see an error stating something like:

_Message (NavNSConcurrencyException): RootException: NavNSConcurrencyException_  
_An internal error has occurred. Multiple concurrent calls have been issued to the server from this client._

# The cause

The reason for this bug is, that the Web Client is trying to add something to the event log and the right event log has not been created.

The problem exists in NAV 2018 and Business Central containers. I haven’t heard about it before the spring release containers.

# The fix

This has been fixed in the Generic NAV / Business Central image version 0.0.9.6 and all Docker images created as of today will include this fix.

# The workaround

If you are running into this problem with an existing container image, you have two options:

1\. Update your NavContainerHelper to 0.6.0.12 or later, then NavContainerHelper will automatically inject an additional setup script (if the docker image is 0.0.9.5 or lower), which will create the necessary event logs.

2\. If you are using Docker without NavContainerHelper, you can override the AdditionalSetup.ps1 script with the same script as NavContainerHelper is using:

Write-Host "Registering event sources"
"MicrosoftDynamicsNAVClientWebClient","MicrosoftDynamicsNAVClientClientService" | % {
    if (-not \[System.Diagnostics.EventLog\]::SourceExists($\_)) {
        $frameworkDir =  (Get-Item "HKLM:\\SOFTWARE\\Microsoft\\.NETFramework").GetValue("InstallRoot")
        New-EventLog -LogName Application -Source $\_ -MessageResourceFile (get-item (Join-Path $frameworkDir "\*\\EventLogMessages.dll")).FullName
    }
}

This script will also work with new images, it will check whether the event log has been created before creating it.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
