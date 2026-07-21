---
layout: post
title: "Multitenant sandbox containers changes behavior…"
date: 2020-10-12 10:57:21
categories: ["BcContainerHelper", "Docker"]
tags: ["Generic Image", "Multitenancy"]
permalink: /2020/10/12/multitenant-sandbox-containers-changes-behavior/
---

With the change to BcContainerHelper, sandbox containers became multitenant by default. This means that you have to remember the ?tenant=default in the WebClient and the “tenant” : “default” in launch.json.

The reason for this change was, that sandbox containers needs to feel and act like online tenants and online tenant are… – tenants. I did however miss out on one thing, which I was made aware of over the weekend.

The default tenant is mounted with AllowAppDatabaseWrite. This was done back in the time to ensure that you had a tenant, which worked more or less like on premises. In a few hours, a new generic image will be published, which reverses this for sandbox containers – the default tenant will NOT have AllowAppDatabaseWrite set to true.

You can still add an additionalparameter if you want the “old” behavior:

```
--env defaultTenantHasAllowAppDatabaseWrite=Y
```

and of course nothing changes if you are using single tenancy.

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
