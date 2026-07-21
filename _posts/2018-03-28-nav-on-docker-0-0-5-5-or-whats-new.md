---
layout: post
title: "NAV on Docker 0.0.5.5 or… – What’s new"
date: 2018-03-28 23:26:10
categories: ["Docker"]
tags: ["Docker", "Generic Image", "NAV on Docker", "Specific image"]
permalink: /2018/03/28/nav-on-docker-0-0-5-5-or-whats-new/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

As some users of NAV on Docker has noticed, the images gets rebuild from time to time. We typically rebuild all images when we have changes to the generic layer, which might be of value to users of NAV on Docker. This blog post describes what’s new since the last blog post on 0.0.4.1 (December 2nd 2017).

A lot of small improvements has happened, but for most users, the changes aren’t really visible, if you “just” use NAV containers for development or test.

## 0.0.5.5 – 2018.03.29

**Remove tenant data from the App database.** For performance reasons, we left the tenant part in the App database when switching to multitenancy. This however made Export-NavContainerDatabasesAsBacpac create wrong bacpac files.

**Test Assemblies**, used for the performance test framework are included in images built on generic 0.0.5.5 or newer. They are located in **C:\\Test Assemblies** inside the container.

**Task Scheduler** is now enabled by default for developer preview and Business Central Sandboxes.

**TestToolkit and Test objects** for localized developer preview and localized business central sandbox containers in **C:\\TestToolKit**.

## 0.0.5.4 – 2018.03.24

In order to stay with the new naming strategy of the microsoft dotnet images, NAV containers are now using **microsoft/dotnet-framework:4.7.1\-windowsservercore****\-ltsc2016** as base image.

**TenantEnvironmentType** is set to Sandbox for Business Central Sandboxes and new tenants are mounted as sandbox tenants.

**Container Age** was checked on restart of the container, meaning that a container, where the age exceeds 90 days will be unable to start once stopped.

## 0.0.5.3 – 2018.02.26

Generate symbols and setup NAV for **dual development** between AL and C/AL.

**Restore bakfile to seperate folder** to avoid conflict with the database in the container.

## 0.0.5.2 – 2018.02.23

**Support for Azure SQL in the ARM template** ([http://aka.ms/getnavext](http://aka.ms/getnavext)), which allows you to deploy bacpacs to Azure SQL as part of your Azure Resource Manager Template.

**Support for TLS 1.2**, which has become a requirement for github and other download places.

**Bugfix**: ClickOnce failure without AcsUri defined.

## 0.0.5.1 – 2018.02.17

**AAD** support for Windows Client using **ClickOnce**

## 0.0.5.0 – 2018.02.15

Support for **Azure Active Directory (AAD) authentication**

## 0.0.4.5 – 2018.02.03

**Fail fast** if the amount of memory assigned to the NAV container isn’t **at least 3Gb**.

**Support for multitenancy.**

## 0.0.4.4 – 2017.12.19

Support for **sharing URLs as folders** to containers to allow for script overriding in Azure Container Services and more.

Bugfix: **Report preview not working** due to missing t2embed.dll

## 0.0.4.3 – 2017.12.09

**Include upgradetoolkit and extensions** from the DVD on the container.

## 0.0.4.2 – 2017.12.04

Support for specifying **custom config settings** for Service Tier, Web Client on Windows Client as a parameter for Docker. Example: –env CustomNavSettings=EnableTaskScheduler=true

Split the install scripts to **version specific folders** to simplify source code.

Alongside all of these changes, the navcontainerhelper PowerShell module has also been updated together with the NAV ARM Templates to give a better experience when deploying test, development and demo environments.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
