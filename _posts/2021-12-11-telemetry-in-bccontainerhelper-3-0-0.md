---
layout: post
title: "Telemetry in BcContainerHelper 3.0.0"
date: 2021-12-11 19:12:46
categories: ["BcContainerHelper", "Docker"]
tags: ["BcContainerHelper", "Docker", "Telemetry"]
permalink: /2021/12/11/telemetry-in-bccontainerhelper-3-0-0/
---

BcContainerHelper 3.0.0 just released and as something new, you will see this message when importing the module:

**_BcContainerHelper emits usage statistics telemetry to Microsoft_**

What does that mean? Can you avoid that? Who can access the emitted telemetry data? How? Can you use telemetry for troubleshooting? Can you use telemetry for other things?

This blog post will try to answer these questions.

## First of all, what is usage statistics?

Usage statistics is collects information about which functions are called/used in BcContainerHelper, which artifacts are being used and what host OS and container OS people are running.

This is done to make sure that we prioritize our efforts in the right places. Should we keep supporting Windows Server 2016? which sandbox artifacts are people using? can we set a retention policy on them? etc. etc.

The Information Microsoft gets from usage statistics cannot be used for troubleshooting and BcContainerHelper also doesn’t output a correlation ID which we can query as there is really no information in that, which provides any valuable info for troubleshooting.

## What if you don’t want to participate in emitting usage statistics?

The telemetry connection string used to emit usage statistics to Microsoft is configurable. If you look at the value of this configuration variable:

`$bcContainerHelperConfig.MicrosoftTelemetryConnectionString`

Then this is the application insights connection string used to emit telemetry to Microsoft. Setting this configuration variable to an empty string, or setting this variable in the configuration file (c:\\programdata\\bccontainerhelper\\bccontainerhelper.config.json) will stop emitting telemetry to Microsoft.

You can even set the variable to point to your own application insights account to see what data is emitted to Microsoft if you like.

**Note:** If you obt out of telemetry, we cannot see which functions and features you are using and you risk that we obsolete or deprecate functions which you are using.

## Who can access the telemetry data emitted?

The default applications insights account is located in Azure together with the other application insights accounts used for Business Central production environments. These application insights accounts can only be accessed from people in Microsoft, who has a reason to access the data, from a special PC (Secure Access Workstation), with a special account (Microsoft cannot access data from these things from our normal corporate accounts) – exactly the same security boundaries as already exists for all other Business Central application insights accounts.

## Using telemetry data for troubleshooting

As already stated, Microsoft cannot use usage statistics data for troubleshooting. If you however set the configuration variable **SendExtendedTelemetryToMicrosoft** to **$true**, then you will (for every function you call) get a telemetry correlation Id in the output – and with this, Microsoft can access the output of your function.

Microsoft will never be able to see passwords, secret urls and other secrets as these are not emitted to telemetry – ever.

![](/assets/images/2021/telemetry-in-bccontainerhelper-3-0-0/telemetry.png)

## Can you use telemetry yourself?

By setting the configuration variable:

`$bcContainerHelperConfig.PartnerTelemetryConnectionString`

You will enable extended telemetry to be send to your own application insights account and you can use the output of BcContainerHelper telemetry yourself. The telemetry data you get is exactly the same as the data submitted to Microsoft (if the SendExtendedTelemetryToMicrosoft is true) – even the same correlation Ids.

Telemetry won’t provide other information than what you already see in the output, but you can query it differently. We will work on creating some queries for containerhelper telemetry and share those on [BCTech](https://github.com/microsoft/BCTech/tree/master/samples/AppInsights).

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
